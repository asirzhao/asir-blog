---
title: Learning notes-Deep Learning, course2, week3
date: 2017-09-30 15:44:25
tags: 
	- batch norm
	- hyperparameter
categories: learning notes
---
不知不觉来到第三周的课程了，大家加油啊，我们一起来看看这一周的内容！
<!--more-->
## Hyperparameter selection
Hyperparameter selection在machine learning中是一个非常重要的优化过程，例如gradient descent中的learning rate \\(\alpha\\)就是关乎算法结果的重要hyperparameter，那么我们应该怎么去选择呢？Ng给出了两个建议：
* 构建多个hyperparameter交叉，随机选择大小，选择效果较好的范围，继续随机选择hyperparameter大小，观察结果。
* 选择参数的时候分段选择，并且使用log分段，例如在0.0001到1之间选择，讲数轴分成0.0001，0.001，0.01，0.1和1，这样选择出的结果更好

对于整体模型的hyperparameter selection，Ng也出了建议，那就是babysitting和parallel方法，一种是对一个模型多次调整，一种是同事启动多个不同hyperparameter的模型，最后取效果最好的。

两种方法殊途同归，可以根据自己的具体情况做出选择。
