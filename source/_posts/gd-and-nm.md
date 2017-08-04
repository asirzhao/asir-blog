---
title: 再深入聊聊梯度下降和牛顿法
date: 2017-08-04 17:26:56
tags: [convex optimization,gradient descent,newton's method]
categories: machine learning
---
上次我们一起从convex function引出了gradient descent和newton's method，而且我们已经知道了gradient descent和newton's method都是convex optimization的好方法,这次我们就专门来讲一讲这两个方法。

用过这两种方法的朋友们都知道，newton's method只需要很少的迭代次数，就可以获得gradient descent上百次乃至上千次迭代的效果。那么，造成这一点的原因又是什么呢?

