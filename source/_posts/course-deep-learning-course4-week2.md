---
title: Learning Notes-Deep Learning, course4, week2
date: 2017-11-29 16:26:12
tags: CNN
categories: learning notes
---
我们继续来看看course4的week2，CNN的知识还是蛮丰富的，本周主要讲了一些景点的CNN结构以及一些computer vision的技巧和知识，一起recap一下。
<!--more-->
## Classic Networks
Ng一共给我们带来了3个最为经典的CNN网络，这里我会给出网络的截图和paper原文，抽空我也会看看原文，希望大家和我一起来看看。
### LeNet-5
![](http://otmy7guvn.bkt.clouddn.com/blog/14/14-1.png) 
[Lécun Y, Bottou L, Bengio Y, et al. Gradient-based learning applied to document recognition[J]. Proceedings of the IEEE, 1998, 86(11):2278-2324.](http://yann.lecun.com/exdb/publis/pdf/lecun-01a.pdf)
### AlexNet
![](http://otmy7guvn.bkt.clouddn.com/blog/14/14-2.png) 
[Krizhevsky A, Sutskever I, Hinton G E. ImageNet classification with deep convolutional neural networks[J]. Communications of the Acm, 2012, 60(2):2012.](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf)
### VGG-16
![](http://otmy7guvn.bkt.clouddn.com/blog/14/14-3.png) 
[Simonyan K, Zisserman A. Very Deep Convolutional Networks for Large-Scale Image Recognition[J]. Computer Science, 2014.](https://arxiv.org/pdf/1409.1556.pdf)

以上可以说是最为经典的三个cnn网络了，大家可以通过阅读paper获得一些详细的知识，都是经典之作，推荐阅读。

## Residual Networks(ResNets)
对于residual networks，我们在这里具体看一下，它的具体原理可以通过下图的residual block来看看：
![](http://otmy7guvn.bkt.clouddn.com/blog/14/14-4.png) 
其实，residual block是把\\(a^{[l]}\\)直接作为\\(a^{[l+2]}\\)输入，也就是说：
$$a^{[l+2]}=g(z^{[l+2]}+a^{[l]})$$
其中\\(g\\)是activation function，如ReLU等。这种思想也被称为short circuit或者skip connection。把上面的residual block串联起来，就变成了我们的residual networks，如下图：
![](http://otmy7guvn.bkt.clouddn.com/blog/14/14-5.png) 
Residual networks最大的特点就是，普通networks随着layer增大，training error理论上是会变小，但是实际上会在某个最小点后增大，但是residual networks则会严格的随着layer增多而减小training error，下面是原文：
[He K, Zhang X, Ren S, et al. Deep Residual Learning for Image Recognition[J]. 2015:770-778.](https://arxiv.org/pdf/1512.03385.pdf)

## Network in Network and 1×1 Convolutions
通常我们使用的filter，都是奇数的kernel matrix，在某些情况下，1×1的filter也会被我们使用，它到底有什么作用呢？我们来看一张图：
![](http://otmy7guvn.bkt.clouddn.com/blog/14/14-6.png) 
从这张图中可以看出，1×1的filter可以压缩input的channel(depth)，因此1×1filter还是有一些意思的。下面是原文：
[Lin M, Chen Q, Yan S. Network In Network[J]. Computer Science, 2013.](https://arxiv.org/pdf/1312.4400.pdf)

## Inception Network
关于inception network，我们先来看一张图：
![](http://otmy7guvn.bkt.clouddn.com/blog/14/14-7.png) 
对于同一个input，我们分别采用不同的filter，甚至max pooling，在保证输出的hight和width一样的前提下，将结果堆叠起来，作为我们的输出，这样做的好处是，我们不需要自己挑选filter，我们将所有的可能都交给network，让它来决定去选择什么样子的结构。原文是：
[Szegedy C, Liu W, Jia Y, et al. Going deeper with convolutions[C].  Computer Vision and Pattern Recognition. IEEE, 2015:1-9.](https://arxiv.org/pdf/1409.4842.pdf)
同时，Ng在课程上说明，inception network 中大量使用了1×1filter来降低计算量，这一点值得我们注意。
我们来看看Inception 单元的图解：
![](http://otmy7guvn.bkt.clouddn.com/blog/14/14-8.png) 
这一周的课程感觉量很大，介绍了很多的网络，我准备下面慢慢的看看这些paper，站在巨人的肩上去看世界，一定会有别样的风景！

## Reference
* [Deep learning-Coursera Andrew Ng](hhttps://www.coursera.org/learn/convolutional-neural-networks)
* [Deep learning-网易云课堂 Andrew Ng](https://mooc.study.163.com/course/2001281004#/info)
