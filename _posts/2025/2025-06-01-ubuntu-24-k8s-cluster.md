---
title: Build a Kubernetes Cluster on Ubuntu 24
image: /assets/img/default-banner.jpg
author: jack
date: 2025-06-01 20:55:00 +0800
categories: [blog, linux]
tags: [linux]
math: false
pin: false
---

In this tutorial, we’ll set up a two-node Kubernetes cluster on Ubuntu 24.04 Server, consisting of one control plane node and one worker nodes.

A Kubernetes cluster consists of a control plane and worker nodes. The control plane manages and orchestrates the cluster, while the worker nodes actually run the application containers (Pods). This guide will walk you through every step, from OS installation and node configuration to deploying basic workloads.

We’ll start by preparing two Ubuntu 24.04 virtual machines as the Master and Worker Nodes. Each VM should have at least 2GB of RAM and 2 CPU cores.


| Node Type | CPU | RAM | Disk | OS | NAT DHCP IP |
| --------- | --- | --- | ---- | -- | ------------ |
| Master    | 2   | 2GB | 15GB | Ubuntu 24.04 | 192.168.122.11 |
| Worker    | 2   | 2GB | 15GB | Ubuntu 24.04 | 192.168.122.12 |

## Master Node Setup

### System Update and Basic Settings

First, log into the Master Node and update the system:

```sh
$ sudo apt update
```

Disable swap:

```sh
$ sudo swapoff -a
```

Enable Kernel IP Forwarding:

```sh
$ echo "net.ipv4.ip_forward=1" | sudo tee -a  /etc/sysctl.conf
$ sudo sysctl -p
```

Enable the `overlay` and `br_netfilter` kernel modules and verify they are loaded:

```sh
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

$ sudo modprobe overlay
$ sudo modprobe br_netfilter
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf 
net.bridge.bridge-nf-call-iptables = 1 
net.bridge.bridge-nf-call-ip6tables = 1 
net.ipv4.ip_forward = 1 
EOF

$ sudo sysctl -p
$ lsmod | grep br_netfilter 
$ lsmod | grep overlay
```

## Install kubelet, kubeadm, kubectl

Follow the [official Kubernetes guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) to install tools using APT. (The version used here is v1.33—this may change, so check the official guide for the latest `curl -fsSL` URL.)

```sh
$ sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$ sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list
```

Then update APT and install the tools:

```sh
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl
```

## Install containerd

We'll use containerd as the container runtime. Follow the [official Docker documentation](https://docs.docker.com/engine/install/ubuntu/) to add the Docker APT repository:

```sh
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Then update and install containerd:

```sh
$ sudo apt-get update
$ sudo apt-get install -y containerd.io
```

Edit `/etc/containerd/config.toml` to enable SystemdCgroup and comment `out disabled_plugins = ["cri"]`:

```sh
$ sudo vim /etc/containerd/config.toml
```

```toml
# disabled_plugins = ["cri"]
SystemdCgroup = true
```

Restart containerd:

```sh
$ sudo systemctl restart containerd
```

## Initialize Kubernetes with kubeadm

Use `kubeadm` to initialize the Kubernetes cluster. The `--pod-network-cidr` flag specifies the IP range for the pod network. If `10.100.0.0/16` conflicts with your network setup, choose another CIDR block.

```sh
$ sudo kubeadm init --pod-network-cidr=10.100.0.0/16
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

> If you run `sudo crictl ps` and see etcd or kube-apiserver constantly restarting, try switching from cgroup v2 to v1. This resolved issues in my QEMU/KVM VMs.
> ```sh
> $ sudo vim /etc/default/grub
> ```
> Modify the `GRUB_CMDLINE_LINUX_DEFAULT`:
> ```ini
> GRUB_CMDLINE_LINUX_DEFAULT="systemd.unified_cgroup_hierarchy=0"
> ```
> Then update GRUB and reboot:
> ```sh
> $ sudo update-grub
> ```

Install Flannel as the pod network plugin. Flannel is a lightweight CNI plugin suitable for Kubernetes.

```sh
$ kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

## Worker Node Setup

Follow the same steps as the Master Node to update the system, disable swap, enable IP forwarding, install kubelet, kubeadm, kubectl, and containerd. **Do not run** `kubeadm init`.

On the Master Node, run the following command to get the join command:

```sh
$ kubeadm token create --print-join-command
```

Run the printed command on the Worker Node to join the cluster.

## Verify Cluster Status

Run the following on the Master Node to verify cluster health:

```sh
$ kubectl get nodes
```