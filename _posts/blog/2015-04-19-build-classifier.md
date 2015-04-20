---
layout: post
title: How-to build a cascade of boosted classifiers based on Haar-like features
description: 因为最近想做一个级联分类来识别手势，所以就顺手翻译一下。
category: blog
---

> [原文地址](http://wiki.opencv.org.cn/images/2/21/OpenCV_ObjectDetection_HowTo.pdf) 前面的部分因为都比较好处理，所以就不翻译了 0w0

## Step 1 - 准备工作

### 工具

配置好opencv，需要用opencv中自带的 *createsamples.exe* 、 *"haartraining.exe* 和 *performance.exe* 。在这个例子中我们使用“C:\Temp\”作为工作空间。

### 负样本

你需要准备很多图片（大概5,000 到 10,000）作为负样本。当然也可以从网上找。

负样本最好选用BMP图像，因为JPG图像的压缩率可能有所不同。这里使用的负样本的图片尺寸和之后用于获取图像的摄像机是一样的（384x256 像素）。但是并不是必要的。

“C:\Temp\negatives\”下包含有一个文件夹，和描述文件，文件夹用于放置负样本图像，描述文件为“train.txt”和“test.txt”。

首先需要一个这些图像的列表，可以用DOS命令来实现。在描述文件同级的目录下，使用命令“dir /b /s > infofile.txt”，然后打开infofile.txt，把所有的路径都改为相对路径。（可以使用替换）

### 正样本

从现实场景中获取图像的优点在于，图像有不同的反光、光照和背景，使用“createsamples”命令可以模仿这些条件，但是总体来说效果并不好。

你可以从公有资源中获取数据（譬如人脸识别），也可以自己创建数据。

未加工的碗（这个例子中训练碗的级联分类器）的图像来自于视频文件。使用了几个有不同地板和光照的场景，来达到不同条件的目的。对于每个场景，相机被放置在不同的角度和距离。

你可能想用“objectmarker.exe”这个工具来从原图像中提取感兴趣的物体。可以为每个文件中的感兴趣物体标定一个轮廓，并且保存在“info.txt”文件中，这个文件可以用于训练或者测试。

