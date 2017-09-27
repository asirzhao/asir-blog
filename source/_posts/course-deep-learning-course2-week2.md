---
title: Learning notes-Deep Learning, course2, week2
date: 2017-09-27 21:38:35
tags:
	- gradient descent
	- moving averages
categories: learning notes
---
大家好，课程来到了第二周，这周主要是一些优化方法，使得整个neural networks可以更快更好的工作，我们一起来recap一下。
<!--more-->
## Mini-batch gradient descent
这一节我就不打算写了，比较基础，其实mini-batch gradient descent是batch gradient和stochastic gradient descent综合后的结果，一方面解决了batch gradient计算量大，容易converge到local minimum的问题，也解决了stochastic gradient descent epoch太多的弊端，是现在gradient descent使用最广泛的方法。

对于mini-batch gradient descent，Ng建议batch大小取决于数据量多少，在小数据量上完全没有必要做mini-batch，直接使用batch gradient descent就可以，对于大数据量的情况，最好选择2的乘方，如64,128,256,512来作为batch size. 当然，batch size也要满足CPU和GPU的内存大小。
## Exponentially weighted averages
Exponentially weighted averages，也被称为moving averages，是一种综合历史数据的加权平均方法，课程中Ng用了伦敦一年的气温变化曲线作为例子，对于固有变量\\(\theta\\)来说，我们要求的平均值\\(v\\)应该是
$$v_0 = 0$$ 
$$v_1 = \beta v_0 + (1- \beta) \theta_1 $$
$$v_2 = \beta v_1 + (1- \beta) \theta_2$$
$$v_3= \beta v_2 + (1- \beta) \theta_3$$
$$\cdots$$
$$v_t= \beta v_{t-1} + (1- \beta) \theta_t$$
其中\\(\beta\\)是一个因子，它决定了moving averages大约向前平均了\\(\frac{1}{1- \beta}\\)个\\(\theta\\)值，例如\\(\beta = 0.9\\)，那么大约向前平均了10个值，并且是向前按指数衰减加权获得的平均值。