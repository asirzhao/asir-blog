---
title: Learning Notes-Deep Learning, course4, week1
date: 2017-11-26 20:30:47
tags: CNN
categories: learning notes
---
Hi, all. 最近开始休假了，可以有空继续自己的学习，一方面补一补前面的作业，一方面继续自己的学习，今天我们来到了course4，也就是convolutional neural networks 的内容。我们一起来看看！
<!--more-->
## Convolution
在课程中，Ng从edge detection的角度来给大家讲了讲convolution，因为本人是image processing出身，所以认为Ng在这里讲的还是很浅显易懂的，我就不再专门的markdown。主要来看看convolution中的一些技巧。
### Padding
我们都知道，在最纯粹的convolution中，我们假设原image尺寸是\\(n \*n\\)，convolution filter尺寸是\\(f \*f\\)，那么最终的结果image尺寸应该是\\( (n-f+1) \*(n-f+1)\\)，也就是说，结果的尺寸变小了。如果想让输出image的尺寸不发生改变，那么我们就要使用大名鼎鼎的padding了。

Padding其实就是表示，在原始image中，向外扩大多少尺寸，一般我们会使用简单复制相邻元素值的方法进行扩充。假设对于一个\\(6 \*6\\)的原始image，采用\\(3 \*3\\)的filter，加上\\\(p=1\\)的padding，那么原始图像尺寸变成了\\(8 \* 8\\)，结果变成了\\(6 \*6\\)，原始image和结果image一模一样！于是加入了padding的convolution公式就成了\\( (n+2p-f+1) \*(n+2p-f+1)\\).

在这里Ng引入了两个概念，valid和same convolutions，所谓valid convolution，就是没有padding 的convolution；所谓same convolution，就是输入输出的尺寸完全一样。

### Stride
继padding之后，还有一个很重要的参数，就是步长stride，步长stride决定了filter做convolution时候的步长，如果stride=1，那么filter就会挨着计算，如果stride=2，那么就会跳跃这进行计算。

总结一下，假设原image尺寸是\\(n \*n\\)，convolution filter尺寸是\\(f \*f\\)，padding值是\\(p\\)，stride值是\\(s\\)那么最终的结果image尺寸应该是\\( ( \frac {n+2p-f}{s}+1) \*( \frac {n+2p-f}{s}+1)\\)，如果除不尽的话，我们选择向下取整，也就是不足以做convolution的区域，我们选择放弃。

##Convolution over Volume
对于一般的图像处理，我们使用的都是RGB图像，我们都知道，RGB图像有三个channel，这种情况下，convolution应该如何做，我们来看下面的图：
![](http://otmy7guvn.bkt.clouddn.com/blog/13/13-1.png) 
假设我们的图像是6×6×3，也就是hight×width×channel(depth)，因此对应的filter也要有3的channel(depth)，最后可以得到一个4×4的结果。

当然，我们可以采用不止一个filter，如图
![](http://otmy7guvn.bkt.clouddn.com/blog/13/13-2.png) 
我们加入了两个不同的filter，他们的大小都是3×3×3，于是最终的结果就是4×4×2，请注意：结果的channel数目取决于filter的个数，而和输入的channel没有任何关系。

## CNN
### Convolution Layer
下面我们来看看CNN网络中的一个layer的工作原理是什么，首先来看截图：
![](http://otmy7guvn.bkt.clouddn.com/blog/13/13-3.png) 
这张图十分复杂，我们一起仔细看看这张图，对于一个6×6×3的RGB图像，我们用了两个3×3×3的filter，我们可以把输入image看做\\(x\\)，也就是\\(a ^{[0]}\\)，filter看做\\(w ^{[1]}\\)，得到的结果就是\\(w ^{[1]}a ^{[0]}\\)，我们再加上一个bias项\\(b^{[1]}\\)，那么就获得了一个liner output\\(w ^{[1]}a ^{[0]}+b^{[1]}\\)，我们再使用一个non-liner function例如ReLU，如此获得一个4×4×2的output。如此就是CNN的一个layer.

如此我们可以看到，CNN和之前的DNN实质上都存在一种liner function到non-liner function的转化，通过non-liner function去classify线性不可分的data，另外，在CNN中，每一个filter就可以获得一个不同的feature，多个filter可以让我们从多个角度去classify data.

另外，相比较于fully connected 的DNN，CNN所需要的parameters也少了很多，这一点值得我们注意。

### Pooling
Pooling原理还是很简单的，我们来看一张图：
![](http://otmy7guvn.bkt.clouddn.com/blog/13/13-4.png) 
首先我们来看看max pooling，如图，我们取filter尺寸\\(f=2\\)，stride大小\\(s=2\\)，对于一个filter中的元素，我们取max作为输出；相对应的，如果我们取average，那么就成了average pooling，pooling中的hyperparameter只有filter尺寸\\(f\\)和stride大小\\(s\\)，值得注意的是，pooling过程中不存在学习过程，no parameters to learn!

### Fully Connected layer
Fully connected layer在CNN其实很简单，我们只需要将input展开，按照DNN的方法进行fully connected就可以了。

一般情况下，我们认为有prameter变化的才算一层，因此我们不认为pooling是一个layer，我们举个一个最简单的CNN做例子：CONV-POOL-CONV-POOL-FC-FC-Softmax，我们这就是一个简单的5层的CNN，在下周的课程中我们可以看到一些经典的CNN框架，这里就不再复述。

## Why Convolutions
关于这个问题，Ng给出了两个意见，我们一起看看：
> **Parameter sharing**: A feature detector (such as a vertical edge detetor) that's useful in one part of image is probably useful in another part of the image.
**Sparsity of connections**: In  each layer, each output value depends only on a small number of inputs.

## Reference
* [Deep learning-Coursera Andrew Ng](hhttps://www.coursera.org/learn/convolutional-neural-networks)
* [Deep learning-网易云课堂 Andrew Ng](https://mooc.study.163.com/course/2001281004#/info)