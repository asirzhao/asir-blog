---
title: 再深入聊聊梯度下降和牛顿法
date: 2017-08-11 17:26:56
tags: 
	- unconstrained optimization
	- gradient descent
	- newton's method
categories: machine learning
---
上次我们一起聊到了gradient descent和newton's method，而且我们已经知道了gradient descent和newton's method都是convex optimization的好方法，这次我们就跳出convex optimization，从更大的unconstrained optimization角度来探讨下这两种方法之间的关联和区别。

假设我们现有一个的optimization task，要求objective function \\(f(x)\\)的最小值，我们一般有两种方案：
* 考虑到\\(f(x)\\)的最小值很有可能是全局最小值，那么我们可以通过寻找\\( \nabla f(x)=0\\)的点来确定最小值，这就是**newton's method**的思想
* 既然我们要寻找最小值，那我们可以顺着一条\\(f(x)\\)逐渐减小的路径，顺着这条路径一直走下去，直到不再变小，这就是**gradient descent**的思想

OK，简单的叙述之后，我们开始正题！
<!--more-->

## 泰勒级数(Taylor series)
首先我们需要回忆一下高等数学中重要的Taylor series，如果\\( f(x)\\)在点\\( x_0\\)的领域内具有\\(n+1\\)阶导数，那么，在该领域内，\\( f(x)\\)可展开成\\(n\\)阶Taylor series，忽略无限大次项的形式就是
$$f(x)=f(x_0)+ \nabla f(x_0)(x-x_0) + \frac{ \nabla ^2 f(x_0)}{2!}(x-x_0)^2 +...+\frac{ \nabla ^n f(x_0)}{n!}(x-x_0)^n $$
其实在高等数学中学到Taylor series的时候，我本人是十分无感的，我并不知道这个东西到底有什么用处，相信很多人和我有相似的经历。

> In mathematics, a Taylor series is a representation of a function as an infinite sum of terms that are calculated from the values of the function's derivatives at a single point.

事实上，Taylor series所表现的是，对于\\( f(x)\\)在点\\( x_0\\)附近的一个估计，也可以理解为，根据\\( x_0\\)点处的各阶derivatives之和构成一个新的function，这个function就是对\\(f(x)\\)的逼近和拟合，而且这种逼近和拟合，随着Taylor series阶数增加而更接近于真实的]\\(f(x)\\)。如果我们使用0阶Taylor series来逼近的话，那我们就粗暴的认为，\\( f(x)\\)在点\\( x_0\\)附近的值就都是\\(x_0\\)，这当然太粗暴直接了，哈哈。

既然这太粗暴了，那么我们就用1st order Taylor series来做一个逼近和估计，这就是gradient descent的思想；如果我们用2nd order Taylor series来估计呢，那就成了newton's method了

OK，我们继续娓娓道来。

## 1st order Taylor series & gradient descent
假设\\(x_k\\)是第k次gradient descent迭代后的\\(x\\)取值，那我们在此处的1st order Taylor series 就是
$$f(x)=f(x_k)+ \nabla f(x_k)(x-x_k)$$
其中\\(x\\)是迭代的下一个方向，gradient descent的目标就是让\\(f(x)\\)达到局部甚至全局最小值，那么每一次迭代，也需要尽可能的减小更多以达到这个目的，那么
$$f(x_k)-f(x)=- \nabla f(x_k)(x-x_k)$$
显然，上式应该尽可能的大，即**\\(- \nabla f(x_k)(x-x_k)\\)越大越好**，我们现在把\\((x-x_k)\\)做一个替换，用单位向量\\(\vec g\\)和标量\\( \alpha\\)分别代表方向和大小，现在的任务就变成了
$$ \min \nabla f(x_k)(x-x_k) = \min( \alpha \vec{\nabla f(x_k)}⋅ \vec g)$$
我们都知道，**对于两个向量来说，当他们方向相反时，他们的内积是最小的**，因此当\\(\vec g\\)的方向是\\( \vec{\nabla f(x_k)}\\)的反方向时，上式可以取到最小值，于是就有
$$x-x_k=- \alpha \nabla f(x_k)$$
$$x:=x_k- \alpha \nabla f(x_k)$$
到这一步，是不是看到了熟悉的gradient descent呢，yeah mate！We make it!
## 2nd order Taylor series & newton's method
和上面的gradient descent相似，假设\\(x_k\\)是第\\(k\\)次newton's method迭代后的\\(x\\)取值，那我们在此处的2nd order Taylor series 是
$$f(x)=f(x_k)+ \nabla f(x_k)(x-x_k) + \frac{1}{2}(x-x_k)^T \nabla^2 f(x_k)(x-x_k) $$
我们对等号两边同时对\\(x\\)求导，并令其为零
$$ \nabla f(x) = \nabla f(x_k) + (x-x_k) \nabla^2 f(x_k)=0$$
由于newton's method的原理就是通过\\(\nabla f(x)=0\\)来寻找最小值，**故上式为零的解\\(x\\)其实就是newton's method在\\(k+1\\)次迭代后的新的\\(x\\)值**。其中\\(\nabla f(x_k)\\)是\\(x_k\\)处的一阶导数，\\( \nabla^2 f(x_k)\\)是\\(x_k\\)处的二阶导数Hessian矩阵元素

我们令\\(\nabla f(x_k)=g\\)，\\(\nabla^2 f(x_k)=H\\)，则上式变成
$$g+H(x-x_k)=0$$
进一步的
$$x=x_k-H^{-1}g$$
由于\\(-g H^{-1} \\) 是优化的前进方向，在寻找最小值的过程中，这个方向一定是和梯度方向\\(g\\)相反才可以更快的下降，那么就有\\( g^T H^{-1} g > 0\\)，这不就是positive definite的定义吗？也就是说，**Hessian矩阵是positive definite的**。

想象一下，如果Hessian是negative definite的话，参数更新的方向就成了和\\(g\\)相同的方向，newton's method将会发散，这一点，也是newton's method的缺点。在objective function是non-convex function的情况下，如果第\\(k\\)次迭代获得的\\(x_k\\)处的Hessian matrix negative definite，那么newton's method将会发散，从而导致不收敛。当然，为了解决这种问题，后续有改进的BFGS等方法，我们在这里暂时不详细讨论。
## Sum up
下面我们再来总结性质的对比一下两种方法，来看一张图
![](http://otmy7guvn.bkt.clouddn.com/blog/2/2-1.png) 
事实上，这两种方法都采用了一种逼近和拟合的思想。假设现在处于迭代\\(k\\)次之后的\\(x_k\\)点，对于objective function，我们用\\(x_k\\)点的Taylor series \\(f(x)\\)来逼近和拟合，当然了，上图我们看到，gradient descent是用一次function而newton's method采用的是二次function，这是二者之间最显著的区别。

对于new's method，在拟合之后，我们通过\\( \nabla f(x)=0\\)求得的\\(x_k\\)点作为此次迭代的结果，下次迭代时候，又在\\(x_k\\)处次进行二次function的拟合，并如此迭代下去。

Newton's method采用二次function来拟合，我们可以感性的理解为，newton's method在寻找下降的方向时候，关注的不仅仅是此处objective function value是不是减小，还关注此处value下降的趋势如何，而gradient descent只关心此处function value是不是减小，因此newton's method可以迭代更少次数获得最优解。对于标准二次型的objective function，newton's method甚至可以一次迭代就找到全局最小值。

但是值得注意的是，上面所说的标准二次型function，实质上是convex function，在一般的unconstrained optimization中，更多的情况则是non-convex optimization，对于一般的non-convex optimization，newton's method是相对不稳定的，因为我们很难保证Hessian matrix的positive definite。鉴于此，我们会加入步长\\(\lambda\\)限制，防止其一次迭代过大而带来迭代后Hessian matrix negative definite的情况，即
$$x:=x- \lambda H^{-1} g$$
对于这种思想，我个人认为，是在整体non-convex function中寻找一个局部的convex function，通过步长将newton's method限制在这个局部中，最后收敛到局部最优中。由此可见，newton's mtehod在non-convex中受限制比较大。

相比之下，由于gradient descent采用的一次function做拟合，只需要考虑沿着梯度反方向寻找最小值，因此gradient descent适用于各种场景，甚至是non-convex optimization，虽然不能保证是全局最优，但至少gradient descent是可以值得一试的方法。

下面来总结一下：
* Gradient descent 和 newton's method都是利用Taylor series对objective function进行拟合来实现迭代的；
* Gradient descent 采用一次型function拟合而 newton's method采用的是二次型function，因此newton's method迭代更迅速；
* Newton's method每次迭代都会计算Hessian matrix的逆，在高维feature情况下，这使得每次迭代会比较慢；
* Newton's method在non-convex optimization中很受限制，而gradient descent则不受影响。

好了，先写这么多，这其中的知识量还是很深奥的，也不知道自己有没有叙述明白，欢迎大家一起来讨论！

**最后感谢优男的宝贵意见！**
## Reference
* [UCLA courseware](http://www.math.ucla.edu/~biskup/164.2.14f/PDFs/recursions.pdf)
* [CCU courseware](https://www.cs.ccu.edu.tw/~wtchu/courses/2014s_OPT/Lectures/Chapter%209%20Newton%27s%20Method.pdf)
* [Taylor series](https://en.wikipedia.org/wiki/Taylor_series)
