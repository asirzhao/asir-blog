---
title: 从凸函数到梯度下降和牛顿法
date: 2017-07-28 15:53:55
tags: [convex optimization,gradient descent,newton's method]
categories: machine learning
---
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
记得我在和优男一起研究Logistic Regression的时候，他问了我几个非常尖锐的问题，让我顿时哑口无言
* 为什么Logistic Regression可以使用通过gradient optimization找到局部最优
* 为什么Logistic Regression还可以用Newton's method呢？
* Newton's method中Hessian matrix必须positive definite有什么意义呢，Logistic cost function能保证吗？

这些细节问题，说实话我也没有认真的想过。在夸奖他之余，我们也一起开始了研究，希望从中学习到一些更深层的东西，趁着现在有个blog分享给大家
<!--more-->

## 凸函数(Convex function)
在开始之前，我有一个关于术语的建议。中文中的凸函数，实际上是看上去是凹下去的，对应的，中文中的凹函数确实看上去突起来的，amazing吧？这是有一定历史原因的，感兴趣的朋友可以去查阅下资料，这里我们不再复述。所以为了避免让大家产生误解，我鼓励大家使用英文，convex function和convcave function.这样会避免很多不必要的麻烦。

OK，我们来一起看看，convex optimization	

对于一维函数 \\(f(x)\\)来说，在定义域内的任意变量\\(a\\)和\\(b\\)，对于任意的 \\( 0 \leq \theta \leq 1\\)，如果满足以下条件，则称为convex function
$$f(\theta a+(1-\theta b)) \leq \theta f(a) + (1- \theta)f(b)$$
我们再用图片直观的感受一下

![](http://otmy7guvn.bkt.clouddn.com/blog/1/1-1.png) 

显而易见的是，当公式中等号去掉的时候，函数就是strictly convex function，等号存在的时候，函数是convex function，或者no-concave function.

Convex function具有一定的性质，我们简单的描述一下，不做详细的推导

### First order condition
对于convex function \\(f\\)，在定义域内一阶可导，且导数为
$$ 	\nabla f=( \frac{\partial f(x)}{x_1}, \frac{\partial f(x)}{x_2},...,  \frac{\partial f(x)}{x_n})$$
那么 \\(f\\) 的first order condition就是对于定义域内任意 \\(x\\) 和 \\(y\\)
$$ f(y) \geq f(x) + \nabla f(x)^T (y - x)$$
OK，再来张图片直观感受一下

![](http://otmy7guvn.bkt.clouddn.com/blog/1/1-2.png) 

其实简单的来讲，就是对于convex function \\(f\\)，他的函数值永远大于等于切线上的值

### Second order condition
对于function \\(f\\)，在定义域内二阶可导，且 \\(n\\) 维方阵Hessian matrix的元素为
$$ \nabla ^2 f(x) = \frac{ \partial ^2 f(x)}{ \partial x_i \partial x_j}, \quad i,j = 1,2,...,n$$
当且仅当Hessian matrix semi-positive defnite的时候，\\(f\\) 是convex function。这里的证明我不想展开讲，在后面我会给出reference链接。

下面给出一些 \\(\Bbb R\\) 空间下常见的convex function
* 线性函数：\\(f(x) = ax+b\\)
* 指数函数：\\(f(x)=e^ {ax}\\)
* 负熵函数： \\(f(x)=xlogx \quad on \quad \Bbb R_{++}\\)

对应的，一些常见的concave function
* 线性函数：\\(f(x) = ax+b\\)
* 对数函数： \\(f(x)=logx  \quad on \quad \Bbb R_{++}\\)

其中大家可以看到，线性函数既是convex也是concave函数，比较特殊，这和它本身的first order condition为常数有关。

以上就是convex function的一个简单介绍，你也许会问，为什么花这么多力气来介绍convex function. 其实，在machine learning中，convex function的优化是一个非常重要的核心，很多算法最后的最后，都是要optimize这么一个convex function，我们会用liner regression和logistic regression为例子，进一步从convex function的简介过渡到gradient descent和newton's method.

## 梯度下降法(Gradient descent)
关于gradient descent，我们使用liner regression作为例子来讨论，如果对算法本身还有不熟悉，可以在后面的reference中查阅相关资料。

Liner regression算法的实质是least square method，他的cost function是
$$J( \theta)= \frac{1}{2} \sum_{i=1} ^m (h_ \theta (x ^{(i)})-y^{(i)})^2, \quad h_ \theta (x)= \theta ^Tx $$
对于liner regression来说，算法的实质就是获得\\( J( \theta) \\)以\\( \theta\\)为参数的minimum，gradient descent就是实现了这么一个过程，类似的，我在这里不详细的讲解gradient descent，作为基础内容，详见reference. 

那么针对cost function，gradient descent是如何保证收敛的呢，我们一起来看看

对于\\(J( \theta)\\)，我们将其带入convex function的定义公式中，注意这里我们的自变量是\\( \theta\\)，我们可以通过推导证明该式成立，也就是说，least square cost function是convex function.

既然有这个结论了，那么我们可以想象一下，least square cost function作为convex function，是有全局最小值的，并且gradient descent不会出现陷入局部最优无法自拔的现象，只要gradient descent保证参数足够好的情况下，理论上，是完全可以获得一个全局最优的解的。

> Gradien descent算法本身并不能保证可以获得全局最小值，只有在objective function是convex function的时候才可以保证

由此，我们可以得出，gradient descent是liner regression的一个很好的optimization 方法

## 牛顿法(Newton's method)
Newton's method 这块内容，我们将会用logistic regression作为例子，同样，关于logistic regression相关的基础知识详见reference

同样，我们先来关注下log cost function
$$J( \theta)=- \frac{1}{m} \sum_{i=1} ^{m} (y^{(i)} logh_{ \theta}(x^{(i)}) + (1-y^{(i)})log(1-h_{ \theta}(x^{(i)}))) \quad h_{ \theta}(x)= \frac{1}{1+e^{- \theta^Tx}}$$
