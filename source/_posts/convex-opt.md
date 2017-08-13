---
title: 从凸函数到梯度下降和牛顿法
date: 2017-08-02 15:53:55
tags: 
    - convex optimization
    - gradient descent
    - newton's method
categories: machine learning
---
记得我在和优男一起研究logistic regression的时候，他问了我几个非常尖锐的问题，让我顿时哑口无言
* 怎么保证logistic regression通过gradient descent找到的是最优解；
* 为什么logistic regression可以用newton's method呢？
* Newton's method中Hessian matrix必须positive definite有什么意义呢，log cost function能保证吗？

这些细节问题，说实话我也没有认真的想过。在夸奖他之余，我们也一起开始了研究，希望从中学习到一些更深层的东西，趁着现在有个blog分享给大家
<!--more-->

## 凸函数(Convex function)
在开始之前，我有一个关于术语的倡议。中文里的“凸函数”，看上去是凹下去的，对应的，中文里的“凹函数”看上去凸起来的，amazing吧？这是有一定历史原因的，感兴趣的朋友可以去查阅下资料，这里我们不再复述。所以为了避免让大家产生误解，我鼓励大家使用英文，**convex function**和**concave function**.这样会避免很多不必要的麻烦。

OK，我们来一起看看，convex function

对于一维函数 \\(f(x)\\)来说，在定义域内的任意值 \\(a\\)和\\(b\\)，对于任意的 \\( 0 \leq \theta \leq 1\\)，如果满足以下条件，则称为convex function
$$f(\theta a+(1-\theta b)) \leq \theta f(a) + (1- \theta)f(b)$$
我们再用图片直观的感受一下

![](http://otmy7guvn.bkt.clouddn.com/blog/1/1-1.png) 

显而易见的是，当公式中等号去掉的时候，函数就是**strictly convex function**.

Convex function具有一定的性质，我们简单的描述一下。

### First order condition
对于 function \\(f\\)，在定义域内**一阶可导**，且导数为
$$ 	\nabla f=( \frac{\partial f(x)}{x_1}, \frac{\partial f(x)}{x_2},...,  \frac{\partial f(x)}{x_n})$$
那么 \\(f\\)是convex function的**充要条件**是：对于定义域内任意 \\(x\\) 和 \\(y\\)
$$ f(y) \geq f(x) + \nabla f(x)^T (y - x)$$
OK，再来张图片直观感受一下：

![](http://otmy7guvn.bkt.clouddn.com/blog/1/1-2.png) 

其实简单的来讲，就是对于convex function \\(f\\)，它的函数值永远大于等于切线上的值！

### Second order condition
对于 function \\(f\\)，在定义域内**二阶可导**，且 \\(n\\) 维方阵Hessian matrix的元素为
$$ \nabla ^2 f(x) = \frac{ \partial ^2 f(x)}{ \partial x_i \partial x_j}, \quad i,j = 1,2,...,n$$
当且仅当Hessian matrix positive semi-defnite的时候，\\(f\\) 是convex function。以上互为**充要条件**。这里的证明我不想展开讲，在后面我会给出reference链接。

---
下面给出一些 \\(\Bbb R\\) 空间下常见的convex function：
* 线性函数：\\(f(x) = ax+b\\)
* 指数函数：\\(f(x)=e^ {ax}\\)
* 负熵函数： \\(f(x)=xlogx \quad on \quad \Bbb R_{++}\\)

对应的，一些常见的concave function：
* 线性函数：\\(f(x) = ax+b\\)
* 对数函数： \\(f(x)=logx  \quad on \quad \Bbb R_{++}\\)

其中大家可以看到，线性函数既是convex也是concave函数，比较特殊，这和它本身的first order condition为常数有关。

以上就是convex function的一个简单介绍，你也许会问，为什么花这么多力气来介绍convex function. 其实，在machine learning中，convex function的优化是非常重要的，很多算法说到底，都是要optimize一个convex function，我们会用liner regression和logistic regression为例子，进一步从convex function的简介过渡到gradient descent和newton's method.

## 梯度下降法(Gradient descent)
关于gradient descent，我们使用liner regression作为例子来讨论。Liner regression算法的实质是least square method，他的cost function是
$$J( \theta)= \frac{1}{2} \sum_{i=1} ^m (h_ \theta (x ^{(i)})-y^{(i)})^2, \quad h_ \theta (x)= \theta ^Tx $$
对于liner regression来说，算法的实质就是去求出\\( J( \theta) \\) **以\\( \theta\\)为参数**的minimum，gradient descent算法的作用就是去实现了这个过程，gradient descent的基础知识详见reference. 

那么针对cost function，gradient descent是如何保证收敛的呢，我们一起来看看

对于\\(J( \theta)\\)，我们将其带入convex function的定义公式中，注意这里我们的自变量是\\( \theta\\)，我们可以通过推导证明该式成立，也就是说，least square cost function是convex function.

既然有这个结论了，那么我们可以想象一下，least square cost function作为convex function，是存在全局最小值的，也就是说，gradient descent不会出现陷入局部最优无法自拔的现象，只要gradient descent保证参数足够好的情况下，理论上，是完全可以很好的逼近全局最优的解的。

> Gradien descent算法本身并不能保证获得全局最小值，只有在objective function是convex function的时候才可以保证

下图可以看出，右边的object function是non-convex function，因而很容易陷入到局部最小值无法自拔，而左边的objective function是一个标准的convex function，在gradient descent参数合理的前提下，可以逼近全局最优。

![](http://otmy7guvn.bkt.clouddn.com/blog/1/1-3.png) 

当然，gradien descent的一些改进方法，例如stochastic gradient descent在解决non-convex optimization上有一些帮助，但是我们在这里不做讨论，后面有时间我会专门再写。

由此，我们可以得出，gradient descent不仅仅是minimize liner regression的一个很好的方法，也是convex optimization的一种理想方法

## 牛顿法(Newton's method)
Newton's method 这块内容，我们将会用logistic regression作为例子。同样，我们先来关注下log cost function，这里，**我们取label为-1和+1**，因为这样得到的cost function比0,1下的计算更加简单
$$J( \omega)= - \frac{1}{m} \sum_{i=1} ^{m} log(1+e^{-y^{(i)} \omega^T x^{(i)}})$$
这里我们采用了-1和+1作为标签值，和大多数教材中不一样，大家可以下来自己推导一下\\(J( \omega)\\)，这种写法广泛的应用在了比较logistic regression和SVM两大分类器的文献中，希望大家熟知。

此处我们对原始的likehood function加上了 \\(- \frac{1}{m}\\)的系数，同样，当我们把 \\(J( \omega)\\)带入到convex function的定义中，可以验证上式为convex function，值得注意的是，\\(J( \omega)\\)是\\( \omega\\)的函数。

其实，我们也可以将log cost function展开后，利用最基本的函数convex和concave性质来获得上式是convex function的结论，碍于公式实在太难打，就留给大家去证明吧。

OK，既然log cost function是convex function，我们一定是可以用gradient descent去求解的。问题是，如果我们用newton's method呢？

Newton's method的基本原理详见reference，这里我们可以发现，既然log cost function是convex function，那么根据second order condition可以知道，它的Hessian matrix一定是positive semi-definite的。如果我们加上了L2 regularizer，**由于L2 regularizer本身就是一个strict convex function**，那么log cost function就一定是strict convex function了，也就是：
$$J( \omega)= - \frac{1}{m} \sum_{i=1} ^{m} log(1+e^{-y^{(i)} \omega^T x^{(i)}})+ \frac{1}{2}|| \omega||^2$$
因此，在log cost function中，**Hessian matrix是positive definite的**，完全满足newton's method 的要求。同样，类似于上一部分，newton's method也可以找到log cost function的全局最优。
## Sum up
OK，我们说到这里也确实讲了不少，这篇blog有些冗长，希望朋友们不要焦虑。总体来说，我想表达的是以下几个观点：
* Machine learning中我们寻求的其实就是objective function一个全局最优值，这些问题是通过gradient descent等方法解决的；
* Gradient descent和newton's method都是convex optimization的好方法，他们都可以对于convex function获得全局最优；
* 对于non-convex optimization问题，stochastic gradient descent也很有效果，我们后续再慢慢学习。

好了，核心思想就这三点，今天先说这么多！

## Reference
* [EE364, Convex Optimization Stanford University](https://see.stanford.edu/materials/lsocoee364a/03ConvexFunctions.pdf)
* [Regularized Logistic Regression is Strictly Convex](http://qwone.com/~jason/writing/convexLR.pdf)
* [XinyiLI大神的blog](https://www.yangzhou301.com/2016/03/14/826442654/)
* [Liner regression](https://en.wikipedia.org/wiki/Linear_regression)
* [Logsitc regression](https://en.wikipedia.org/wiki/Logistic_regression)
* [Gradient descent](https://en.wikipedia.org/wiki/Gradient_descent)
* [Newton's method](https://en.wikipedia.org/wiki/Newton%27s_method)

