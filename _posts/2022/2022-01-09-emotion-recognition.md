---
title: Emotion Recognition
image: /assets/img/emotion-recognition.png
author: jack
date: 2022-01-09 20:55:00 +0800
categories: [python, machine learning]
tags: [python, machine learning]
math: false
pin: false
---

The project is a fork of [usef-kh/fer](https://github.com/usef-kh/fer). We made an application for detecting user-provided human face image. And we use the code from [atulapra/Emotion-detection](https://github.com/atulapra/Emotion-detection) to implement a real-time camera emotion detector.

Github: [https://github.com/blueskyson/emotion-recognition](https://github.com/blueskyson/emotion-recognition)

It is the final project of CSIE7606 - Computer Vision and Deep Learning by [Jenn-Jier (James) Lien](https://www.csie.ncku.edu.tw/en/members/25) at National Cheng Kung University. This course not only allows students to gain an in-depth understanding of the theoretical knowledge of computer vision, machine learning, and artificial intelligence-deep learning, and to analyze how the principles of deep learning are combined with the development of artificial intelligence and computer vision, but also focuses on the technical functionalities. These are explained through the instructor's industry-academic collaboration experiences, including facial expression analysis, cloud-based intelligent monitoring services, automated optical inspection, intelligent robotic arm control, and autonomous vehicles.

## 1. Introduction

Facial emotion recognition plays an important role in **human-computer interactions** and can be applied to **digital advertisement**, **gaming**, and **customer feedback assessment**.

*One specific emotion recognition dataset that encompasses the difficult naturalistic conditions and challenges is FER2013.* Human performance on this dataset is estimated to be 65±5% [2], which means <span style="color:orange;">it is hard even for humans to recognize the emotions.</span> This work adopts <span style="color:orange;">VGG network</span> and shows various experiments to explore different <span style="color:orange;">optimization algorithms</span> and <span style="color:orange;">learning rate schedulers.</span> The authors thoroughly tune the model and hyperparameters to achieve <span style="color:red;">state-of-the-art results at a testing accuracy of 73.28 %.</span>

<span style="color:blue;">We have reproduced this paper’s work. Furthermore, we apply Haar filter to make an application that detects emotions from user-provided images and integrate our model in real-time camera emotion detector.</span>

## 2. System Architecture

<span style="color:orange;">VGGNet</span> is a classical convolutional neural network architecture used in pattern recognition. The network consists of <span style="color:red;">4 convolutional stages</span> and <span style="color:red;">3 fully connected layers</span>.

Each convolutional stage contains two <span style="color:red;">convolution blocks</span> and a max-pooling layer. Each <span style="color:orange;">convolution block</span> contains a <span style="color:red;">convolutional layer</span>, a <span style="color:red;">ReLU activation</span>, and a <span style="color:red;">batch normalization</span> layer. Batch normalization is used to speed up learning process, reduce the internal covariance shift, and prevent gradient vanishing or explosion.

The first two fully connected layers are followed by a ReLU activation. The third fully connected layer is for classification.

Overall, convolutional stages are responsible for feature extraction, and fully connected layers are trained to classify the images.

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition1.png)

- **Conv1** to **Conv4** are convolution stages, which are used for feature extraction.
- A blue rectangle represents a convolution block.
- A green rectangle represents a max-pooling layer.
- **FC1** to **FC3** are fully connected layers, which are used to classify emotions from features extracted by former stages.

### Optimizers

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition2.png)

The authors test 6 different optimizers, including SGD,
SGD with Nesterov Momentum, Average SGD, Adam,
Adam with AMSGrad, Adadelta, and Adagrad. This
experiment is done under two configurations:

<span style="color: blue;">Constant LR</span>: Fixed learning rate of 0.001.
<span style="color: blue;">Decaying LR</span>: Decaying the initial learning rate of
0.01 by a factor of 0.75 if the validation accuracy
plateaus for 5 epochs (RLRP scheduling).

<span style="color: red;">They find that the SGD with Nesterov Momentum
is the best.</span>

### Learning Rate (LR) Schedule

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition3.png)

The authors test 5 different LR schedulers, including
Reduce Learning Rate on Plateau (RLRP), Cosine
Annealing (Cosine), Cosine Annealing with Warm
Restarts (CosineWR), One Cycle Learning Rate
(OneCycleLR), and Step Learning Rate (StepLR).

Using SGD with Nesterov momentum optimizer.

<span style="color: red;">They find that the RLRP is the best since it
monitors the current performance before deciding
when to drop the learning rate.</span>

## 3. Data Collection

<span style="color: orange;">FER2013</span> is a dataset composed of 35587 grey-scale 48x48 images of faces classified in <span style="color: orange;">7 categories: anger, disgust, fear, happiness, sadness, surprise, neutral.</span>

The dataset is divided in a training set (28709 images), a public
test set (3589 images), usually considered the test set for final
evaluations.

FER2013 is known as a challenging dataset because of its <span style="color: orange;">noisy
data with a relatively large number of non-face images and
misclassifications</span>. It is also <span style="color: orange;">strongly unbalanced</span>, with only 436
samples in the less populated category, "Disgust", and 7215
samples in the more populated happiness.

<span style="color: red;">This dataset analogous to real world challenges in the field.</span>

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition4.png)

## 4. Our Experimental Results

### Accuracy

In our experiment the model had <strong>great gains from epoch [0 to 30]</strong>, and <strong>continues to advance [epoch 31~100]</strong> thanks to the <span style="color: orange;">RLRP learning rate scheduler</span>.

<p style="color: red;">
<strong>Terminating at the 100th epoch, the resulting testing set accuracy is 71.52415%</strong>, which is on par with that of the original paper (73.06%) we based our experiment on.
</p>

<p style="color: blue;">
<strong>This is state-of-the-art on FER2013 without extra training data.</strong>
</p>

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition5.png)

### Loss

During the training phase, the training loss never equaled to validation loss, indicating that <strong>there is no underfitting</strong>.

Nor was the training loss much lower than the validation loss, indicating that <strong>no overfitting occured</strong> during the process.

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition6.png)

### The Paper's Original Fine Tuning Result

The authors tune their best model (73.06% accuracy) for <strong>50</strong> extra epochs using both cosine annealing schedulers with <strong>lr=0.0001</strong>. <span style="color: orange;">Cosine Annealing performs best here and improves the model by 0.05 %</span>. Cosine Annealing with Warm Restarts negatively impacts performance in this tuning stage reducing the accuracy by 0.42 %. Both models <span style="color: blue;">perform better after training on the combined dataset</span> resulting from the combined training and validation data.

<p style="color: red;">
<strong>Paper’s final best model achieves an accuracy of 73.28%.</strong>
</p>

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition7.png)

### Our Fine Tuning Result

Tune our best model (71.6912% accuracy) for <strong>20</strong> extra epochs using both cosine annealing schedulers with <strong>lr=0.0001</strong>.

Both models <span style="color: blue;">perform better after training on the combined dataset</span> resulting from the training and validation data.

<p style="color: red;">
<strong>Our final best model achieves an accuracy of 72.3878%.</strong>
</p>

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition8.png)

## 5. Demo

Correct Recognitions:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition9.png)

Incorrect Recognitions:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition10.png)

Real-time Camera Detector:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition11.gif)

## 6. Conclusion and Future Works

This paper achieves <span style="color:red;">single-network state-of-the-art classification accuracy on FER2013</span> using a <span style="color:orange;">VGGNet</span>. We thoroughly tune all hyperparameters towards an optimized model for facial emotion recognition. <span style="color:orange;">Different optimizers and learning rate schedulers are explored</span> and the best initial testing classification accuracy achieved is <span style="color:red;">73.06%</span>, surpassing all single-network accuracies previously reported. We also carry out extra tuning on our model using <span style="color:orange;">Cosine Annealing and combine the training and validation datasets</span> to further improve the classification accuracy to <span style="color:red;">73.28%</span>.

For future work, we plan to explore different <span style="color:orange;">image preprocessing</span> techniques on FER2013 and investigate <span style="color:orange;">ensembles of different deep learning architectures</span> to further improve our performance in facial emotion recognition.

<p style="color:blue;">
We got our best model by training 100 epochs and reproduced the fine tuning experiments with 20 extra epochs. The result shows that combine training and validation with Cosine Annealing performs best, which is consistent with the paper.
</p>

## 7. Additional Work

We use <span style="color:orange;">ResNet34V2</span>, published by Microsoft Reasearch. In contrast to the original ResNet34, ResNet34V2 puts batch normalization layer and ReLU activation before the convolution layer.

In our experiment the model had <strong>great gains from epoch[0~20], and continues to advance [epoch 31~100]</strong> thanks to the RLRP learning rate scheduler.

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/emotion-recognition12.png)

<span style="color:red;">Terminating at the 140th epoch , the resulting testing set accuracy is 66%.</span>  The accuracy is lower than VGGNet, but it is a little bit better than human performance.

### Comparison

|                 | Our VGGNet | Paper's VGGNet | ResNet34V2 | SVM  |
|-----------------|------------|----------------|------------|------|
| Testing Accuracy| 72.387852% | 73.28%         | 66%        | 31%  |
| epoch           | 120        | 350            | 140        | X    |
| fine-tuning     | final 20 epochs with combined training and validation dataset | final 50 epochs with combined training and validation dataset | X          | X    |

## 8. Reference Sources

- [1] Yousif Khaireddin, Zhuofa Chen, “Facial Emotion Recognition: State of the Art Performance on FER2013”, arXiv:2105.03588
- [2] Roberto Pecoraro, Valerio Basile, Viviana Bono, Sara Gallo, “Local Multi-Head Channel Self-Attention for Facial Expression Recognition”, arXiv:2111.07224
- [3] Shivam Gupta, “Facial emotion recognition in real-time and static images”, DOI: 10.1109/ICISC.2018.8398861
- [4] Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun, “Identity Mappings in Deep Residual Networks”, arXiv: 1603.05027

Referenced Repositories
1. Code for the paper "Facial Emotion Recognition: State of the Art Performance on FER2013", https://github.com/usef-kh/fer
2. Real-time Facial Emotion Detection using deep learning, https://github.com/atulapra/Emotion-detection
3. ResNet for pytorch, https://github.com/pytorch/vision/blob/main/torchvision/models/resnet.py

