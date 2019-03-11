---
layout: post
title: DeconvNet 理论
category: 语义分割
tags: deconvnet
description: deconvnet
---

**关键点**

1. encoder-decoder 结构
2. 可学习的深度反卷积网络：反卷积层由 deconvolution、relu 和 upooling组成。
3. 结合目标检测：把每张图片的不同的 proposal 送入网络，通过 combin 得到最后的语义分割结果。
4. 引入 CRF（条件随机场）。

## 一、FCN 突出的缺点

1. 网络的感受野是固定的

那些大于或者小于感受野的目标，就可能被分裂或者错误标记。具体点来说，对于大目标，进行预测时只使用了local information所以可能会导致属于同一个目标的像素被误判为不连续的标签(即不同目标)，如下图，左侧为input，中间为ground truth，右侧为result，可见这个大巴由于too large所以被分割成了很多不连续的部分。

<center>

<img src="https://raw.githubusercontent.com/chiemon/chiemon.github.io/master/img/DeconvNet/1.png">

</center>

而对于小目标来说，经常会被忽略掉，被当作了背景。如下图，左侧为input，中间为ground truth，右侧为result。由于人很远所以在图中面积too small，结果被当作背景了：

<center>

<img src="https://raw.githubusercontent.com/chiemon/chiemon.github.io/master/img/DeconvNet/2.png">

</center>

2. 目标的细节结构常常丢失或者被平滑处理掉

输入 deconvolution-layer的label map太粗糙了（只有16x16），而且 deconvolution 这个步骤在 FCN 这篇文章中做的过于简单了。缺少一个在大量数据上得到训练的deconvolution network 使得准确地重构物体边界的高维非线性结构变得困难。

## 二、网络结构

<center>

<img src="https://raw.githubusercontent.com/chiemon/chiemon.github.io/master/img/DeconvNet/3.png">

</center>

DeconvNet 和 SegNet 的结构非常类似，只不过 DeconvNet 在 encoder 和 decoder 之间使用了 FC 层作为中介。

- 卷积层：使用VGG-16（去除分类层），把最后分类的全连接层去掉，在适当的层间应用Relu和Maxpooling。增加两个全连接层（1x1卷积）来强化特定类别的投影。
- 反卷积层：卷积层的镜像，包括一系列的 unpooling，deconvolution，Relu 层
- 网络输出：概率图，和输入图像大小相同，表明每个像素点属于预定义类别的概率

## 三、Unpooling

与 SegNet Upsample 的方法一样。正向 pooling 的时候用 switch variables 记录 Maxpooling 操作得到的activation的位置，在 Unpooling 层利用 switch variables 把它放回原位置，从而恢复成 pooling 前同样的大小。

switch variables 记录的只是 Pooling 的时候被选中的那些值的位置，所以 Unpooling之后得到的 map 虽然尺寸变回来了，但是只是对应的位置有值，其它地方是没有值的。

## 四、Deconvolution

- 通过deconvolution使稀疏响应图变得稠密。反卷积操作可以实现1个输入,多个输出， 经过反卷积之后会得到扩大且密集的响应图。
- 经过反卷积之后，裁剪(crop)响应图的边界，使其等于unpooling层的输出大小(也是deconvolution层输入的大小)。
- 得到由一系列deconvolution,unpooling layer组成的层级反卷积网络。

## 五、网络总结

- 该网络实质上将语义分割视作了实例分割问题。
- 将潜在包含对象的子图像作为输入，并以此生成像素级预测作为输出。
- 通过将网络应用于从图像中提取的每个proposal候选区域，并将所有proposal的输出集合到原始图像空间，得到整个图像的语义分割。

较低层网络更多捕捉物体的粗略的外形，像位置，形状，区域等，在高层网络中捕捉更加复杂的模式类别。反池化与反卷积在重构feature map时发挥着不同的作用，反池化通过原feature map中较强像素的位置信息来捕捉 example-specific 结构，进而以一定的像素来构建目标的结构细节，反卷积中的卷积核更倾向于捕捉 class-specific 形状，经过反卷积，虽然会有噪声的影响，但激活值仍会与目标类别进行相关联。该网络将反卷积和反池化结合，获得较好的分割效果。

## 六、训练方法
