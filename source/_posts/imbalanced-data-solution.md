---
title: Imbalanced data 问题总结方法汇总
date: 2017-10-25 23:01:09
tags:
	- imbalanced data
categories: machine learning
---
Hello，大家好，最近忙里偷闲看了[A systematic study of the class imbalance  problem in convolutional neural networks](https://arxiv.org/pdf/1710.05381.pdf)，感觉paper呈现的研究内容感觉很一般，但是，paper中关于imbalanced data的solution方法倒是写的很不错，也勾起了我对于这一块总结的欲望。之前也写过一篇关于imbalanced data的paper notes，但是对于这一块的具体方法总结还不是很足够，于是用这篇paper为主线好好sum up一计。

我们来一起看看。
<!--more-->
## Data level methods
首先我们来看一看data level methods，这类方法有一个共性，那就是通过改变data的数量来完成对imbalanced data problem的解决。
### Oversampling
