---
title: 深入聊聊正则化
date: 2017-08-27 01:55:07
tags: 
	- regularization
	- MAP
	- ridge regression
	- lasso regression
categories: machine learning
---
最近和优男一起聊到了L1和L2 regularization，期间遇到了很多没有想明白的问题，加上最近工作有些忙，空余时间用来倒腾新到货的小米路由器，只能趁周末自己研究研究，下面和大家分享一下regularization中一些深入的问题。不讨论基础知识，直接上干货。
<!--more-->
## MAP and regularization
我们都知道，当cost function在没有加regularization的时候，我们对参数使用的是**MLE**(Maximum likelihood estimation)，对应频率学派所认为的参数本无分布规律的观点；在Andrew Ng经典的CS229中，这位AI大师曾经提到，regularization其实是对参数的**MAP**(Maximum a posteriori estimation)，是基于贝叶斯学派认为的参数本有**priori distribution**，同时吸纳了MLE的一种中间观点。

这里的priori distribution，就是根据经验，认为参数应该大致符合某个distribution，这样，最终获得的参数估计结果也会和这个被认为的distribution有一些相近

而我们所熟知的L1 regularization，其实就是认为参数的priori distribution是**Laplacian distribution**，而L2 regularization，则认为参数的priorit distribution是**Gaussian distribution**，相信大家对Gaussian distribution是很熟悉的，而对于Laplacian distribution，它的分布是
$$p(x;a)= \frac{a}{2} e^{-a|x|}$$
下图就是两者的一个比较：
![](http://otmy7guvn.bkt.clouddn.com/blog/4/4-1.png) 
在后文中，我们会看到，两种distribution的特点决定了两种regularization的性质。
### Lasso regression
在liner regression中，我们假设参数\\( \theta\\)服从Laplacian distribution，cost function就成了
$$J( \theta) = \frac{1}{2} \sum^{m}_{i=1} (y^{(i)}- \theta^T x^{(i)})+ \lambda \sum^{d}_{j=1}| \theta^{(i)}|$$
上式就是Lasso regression
### Ridge regression
在liner regression中，我们假设参数\\( \theta\\)服从Gaussian distribution，cost function就成了
$$J( \theta) = \frac{1}{2} \sum^{m}_{i=1} (y^{(i)}- \theta^T x^{(i)})+ \lambda \sum^{d}_{j=1} ( \theta^{(i)})^2$$
上式就是Ridge regression或shrinkage
## geometry of error surfaces
在不考虑参数priori distribution的时候，cost function的形式是
$$J( \theta) = \frac{1}{2} \sum^{m}_{i=1} (y^{(i)}- \theta^T x^{(i)})^2$$
用二维截面图展示就是
![](http://otmy7guvn.bkt.clouddn.com/blog/4/4-2.png) 
图中只有objective function，横纵轴是参数\\( \theta\\)，截取过来的图，所以上面的参数是\\(w\\)，\\(l\\)是loss值，箭头指向的点就是cost function的极小点。在不考虑参数priori distribution的时候，这个点就是我们的optimization target.

下面大家来我一起做一个头脑风暴，所谓参数的priori distribution，其实就是用来限制最后optimization结果的一个限定，那么我们其实就是在做一个受限制的的convex optimization，即：
$$ \theta=argmin \frac{1}{2} \sum^{m}_{i=1} (y^{(i)}- \theta^T x^{(i)})^2$$
$$ s.t. \sum^{d}_{j=1}| \theta^{(i)}|^p \geq \beta$$
那么此时的图就变成了：
![](http://otmy7guvn.bkt.clouddn.com/blog/4/4-3.png) 
我们从图中可以看到，在加入了限制后，最终的optimization不是落在极小值点，而是落在图中所示的位置。从另一个角度来想，regularization item的加入，使得整个cost function在寻找最小值的时候，要均衡的考虑objective function和regularization item的大小.

在这个地方，我和优男讨论的时候有一个地方没有想通，例如使用gradient descent进行optimzation的时候，怎么保证优化可以落到图中的点呢，我是这么考虑的：当加入regularization后，cost function本身就有了变化，随之而来的是gradient也发生了变化，在gradient descent迭代过程中就已经把regularization的影响带了进去，因此在每一次迭代的时候，实际上应该都是按照上式的限制进行优化的。

当然，上图也可以用来就是为什么lasso可以获得稀疏特征，那就是因为lasso更可能在坐标轴上和objective function产生交点，进而使得一些特征变成0.

## Reference
* [CS 195-5: Machine Learning](https://pdfs.semanticscholar.org/91a9/5626d24c8393e3b784e44f62de201d20dede.pdf)
* [STAT 897D](https://onlinecourses.science.psu.edu/stat857/node/155)