---
title: 再深入聊聊梯度下降和牛顿法
date: 2017-08-04 17:26:56
tags: [gradient descent,newton's method]
categories: machine learning
---
上次我们一起从convex function引出了gradient descent和newton's method，而且我们已经知道了gradient descent和newton's method都是convex optimization的好方法,这次我们就专门来讲一讲这两个方法。
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

用过这两种方法的朋友们都知道，newton's method只需要很少的迭代次数，就可以获得gradient descent上百次乃至上千次迭代的效果。那么，造成这一点的原因又是什么呢?

假设我们现有一个的optimization task，要求objective function \\(f(x)\\)的最小值，我们一般有两种方案：
* 考虑到\\(f(x)\\)的最小值很有可能是全局最小值，那么我们可以通过寻找\\( \nabla f(x)=0\\)的点来确定最小值，这就是**newton's method**的思想
* 既然我们要寻找最小值，那我们可以顺着一条\\(f(x)\\)逐渐减小的路径，顺着这条路径一直走下去，直到不再变小，这就是**gradient descent**的思想

OK，简单的叙述之后，我们开始正题！

## 泰勒级数(Taylor series)
首先我们需要回忆一下高等数学中重要的Taylor series，如果\\( f(x)\\)在点\\( x_0\\)的领域内具有\\(n+1\\)阶导数，那么，在该领域内，\\( f(x)\\)可展开成\\(n\\)阶梯Taylor series
$$f(x)=f(x_0)+ \nabla f(x_0)(x-x_0) + \frac{ \nabla ^2 f(x_0)}{2!}(x-x_0)^2 +...+\frac{ \nabla ^n f(x_0)}{n!}(x-x_0)^n $$
其实在高等数学中学到Taylor series的时候，我本人是十分无感的，我并不知道这个东西到底有什么用处，相信很多人和我有相似的经历。

事实上，Taylor series所表现的是，对于\\( f(x)\\)在点\\( x_0\\)附近的一个估计，而且，这个附近十分小，无限接近于点\\(x_0\\)。如果我们使用0阶Taylor series来估计的话，那我们就粗暴的认为，\\( f(x)\\)在点\\( x_0\\)附近的值就是\\(x_0\\)，这当然太粗暴直接了，哈哈。

既然这太粗暴了，那么我们就用1阶Taylor series来做一个逼近和估计，这就是gradient descent的思想；如果我们用2阶来估计呢，那就成了newton's method了

OK，我们继续娓娓道来。

## 1st order Taylor series & gradient descent
对于objective function \\(f(x)\\)，假设\\(x_k\\)是第k次gradient descent迭代后的\\(x\\)取值，那我们在此处的1st order Taylor series 就是
$$f(x)=f(x_k)+ \nabla f(x_k)(x-x_k)$$
其中\\(x\\)是迭代的下一个方向，gradient descent的目标就是让\\(f(x)\\)达到局部甚至全局最小值，那么每一次迭代，也需要尽可能的减小更多以达到这个目的，那么
$$f(x_k)-f(x)=- \nabla f(x_k)(x-x_k)$$
显然，上式应该尽可能的大，即]\\(- \nabla f(x_k)(x-x_k)\\)越大越好，我们现在把\\((x-x_k)\\)做一个替换，用单位向量\\(g\\)和标量\\( \alpha\\)分别代表方向和大小，现在的任务就变成了
$$min \nabla f(x_k)(x-x_k) = min \alpha \nabla f(x_k)⋅g$$
我们都知道，对于两个向量来说，当他们方向相反时，他们的内积是最小的，因此当\\(g\\)的方向是\\( \nabla f(x_k)\\)的反方向时，上式可以取到最小值，于是就有
$$x-x_k=- \alpha \nabla f(x_k)$$
$$x:=x_k- \alpha \nabla f(x_k)$$
到这一步，是不是看到了熟悉的gradient descent呢，yeah mate！We make it!
## 2nd order Taylor series & newton's method