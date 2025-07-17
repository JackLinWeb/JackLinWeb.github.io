---
title: Booting Linux Kernel with a custom init program on QEMU Ubuntu 24
image: /assets/img/default-banner.jpg
author: jack
date: 2025-07-13 20:55:00 +0800
categories: [blog, linux]
tags: [linux]
math: false
pin: false
---

This post documents how I wrote and compiled a minimal `init` program in C, built a simple `initramfs`, and successfully booted it using a custom Linux kernel v6.12 on Ubuntu 24.04 via QEMU. Before you begin, make sure you’ve already built your own Linux kernel as shown in [this post](/posts/boot-tiiny-linux).

## Create a Minimal C Project

```bash
mkdir -p my-init/bin
cd my-init
touch init.c
```

Edit `init.c`:

```c
// init.c
#include <stdio.h>
#include <unistd.h>

int main() {
    while (1) {
        printf("$ ");
        fflush(stdout);

        char input[256];
        if (fgets(input, sizeof(input), stdin) == NULL) break;

        printf("Sorry, I don't know how to do that.\n");
    }
    return 0;
}
```

Now compile this as a statically linked binary. This ensures the kernel doesn’t rely on any external shared libraries or runtime:

```bash
gcc init.c -static -o bin/init
```

## Build an initramfs Image

Before the kernel can execute the `init` program, we must place it inside an initial RAM-based file system (initramfs). The kernel will extract this archive into memory at boot and use it as the root filesystem.

```bash
cd bin
chmod +x init
cd ..
find . -print0 | cpio --null --create --verbose --format=newc | gzip -9 > ../initrd
```

This generates an `initrd` file — a gzipped `cpio` archive — which contains your custom `init` binary.

## Enable Kernel Features

Make sure your kernel supports `initramfs`, ELF binaries, and 64-bit execution. Run:

```bash
make menuconfig
```

Then enable these options:

1. General setup ---> Initial RAM filesystem and RAM disk (initramfs/initrd) support
2. Executable file formats ---> Kernel support for ELF binaries
3. Processor type and features ---> 64-bit kernel

Recompile the kernel afterward if needed.

## Boot with QEMU

Now boot your kernel and initramfs using QEMU:

```bash
qemu-system-x86_64 -kernel arch/x86/boot/bzImage -initrd initrd -nographic -append "console=ttyS0"
```

Expected output:

```text
$ Sorry, I don't know how to do that.
$ Sorry, I don't know how to do that.
...
```

Each time you type a line and press Enter, it responds with a fixed message — just like the minimal shell we designed!

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2025/tiny-linux-init-1.png)
