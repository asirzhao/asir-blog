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

OK，