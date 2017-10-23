---
title: Reading Notes-Swish：A Self-gated Activation Function
date: 2017-10-22 16:13:30
tags: 
	- activtion function
categories: reading notes
---
Hi all，今天和大家分享一篇比较新的paper，是关于一种新的activation function，关于我们知道的activation function，有sigmoid，tanh，ReLU以及ReLU的一些变种，那我们今天来看看这种新提出的activation function到底有什么特色。
<!--more-->
## Notes
首先定义，swish activation function \\(f(x)=x \cdot \sigma (x)\\)，其中\\(\sigma(x)\\)是sigmoid function，也就是\\( \sigma(x)=1/(1+ e^{-x})\\).Swish functin的图像如图所示：
![](http://otmy7guvn.bkt.clouddn.com/blog/11/11-1.png) 
我们再来看下swish function的1st and 2nd derivatives，
![](http://otmy7guvn.bkt.clouddn.com/blog/11/11-2.png) 
下面我们一起来集中看看swish function的优点都有什么，作者给出了以下几点：函数值没有上限，函数值有下限，函数不单调，函数光滑连续，我们一起看看：
### Unbounded above
Unbounded above的实质，是防止activation function在bounded value处发生saturation. bounded above 带来的问题，就是越接近bounded value的时候，function gradient就会越小，逐渐接近0，这就导致gradient descent异常缓慢甚至无法converge。例如sigmoid 和tanh function，他们都是bounded below and above，当我们采用这两种activation function的时候，我们必须谨慎小心的让初始值尽量在function的接近liner的部分来避免上面问题的产生，因此，unbounded above是一个很好的优点，例如ReLU及其变种都采用了这一原则。

### Bounded below & non-monotonicity
Bounded below其实也是一种很好的方法，并且也有activation function已经采用了，采用该方法后，所有负数input都会得到相差无几的activation value，也就是说，-1000和-1的值几乎没有区别，按照author的话来讲，就是我们将
> make large negative input "fogotten"

这其实也是regularzation的一种思想，这种方法在ReLU等方法中也有体现，但是，swish可以通过自身的非单调性质，将比较小的negative input仍然以negative value输出，non-monotonicity提供了更好的gradient flow.

### Smothness
关于smoothness的优点，我们来看一张图：
 ![](http://otmy7guvn.bkt.clouddn.com/blog/11/11-3.png) 
 
 总而言之，个人感觉swish应该算是一个不错的activation，本人由于时间原因，还没有来得及自己测试它，但是据我所看到的讨论，swish的实际效果貌似不是十分稳定，所以我们可以持保留意见，进一步观察它的表现。
 
## Reference
* [Ramachandran P, Zoph B, Le Q V. Swish：a Self-Gated Activation Function[J]. arXiv preprint arXiv:1710.05941, 2017.](https://arxiv.org/pdf/1710.05941.pdf)
