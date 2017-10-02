---
title: Learning notes-Deep Learning, course2, week3
date: 2017-09-30 15:44:25
tags: 
	- hyperparameter
	- batch norm
	- covariate shift
categories: learning notes
---
不知不觉来到第三周的课程了，大家加油！这周的主要内容是hyperparameter selection和batch normal的问题，我们一起来看看这一周的内容！
<!--more-->
## Hyperparameter selection
Hyperparameter selection在machine learning中是一个非常重要的优化过程，例如gradient descent中的learning rate \\(\alpha\\)就是关乎算法结果的重要hyperparameter，那么我们应该怎么去选择呢？Ng给出了两个建议：
* 构建多个hyperparameter交叉，随机选择大小，选择效果较好的范围，继续随机选择hyperparameter大小，观察结果。
* 选择参数的时候分段选择，并且使用log分段，例如在0.0001到1之间选择，讲数轴分成0.0001，0.001，0.01，0.1和1，这样选择出的结果更好

对于整体模型的hyperparameter selection，Ng也出了建议，那就是babysitting和parallel方法，一种是对一个模型多次调整，一种是同事启动多个不同hyperparameter的模型，最后取效果最好的。

两种方法殊途同归，可以根据自己的具体情况做出选择。
## Batch norm
### Normalization
相信大家都听说过大名鼎鼎的normalization吧，这是对于数据预处理的一种很棒的方法，它可以很好的提升数据处理，例如gradient descent的处理速度和效果，在引入batch norm之前，我也稍微提一下normalization，下面上公式：

对于输入数据来说，我们可以按一下方法来normalize
$$ \mu = \frac{1}{m} \sum_i x^{(i)}$$
$$X = X- \mu$$
$$ \sigma^2 = \frac{1}{m} \sum_i (x^{(i)})^2 $$
$$ X = X/ \sigma ^2$$
这样，我们就把输入数据转化成了符合期望为0，方差为1的Gaussian distribution的数据。

当然，这只是normaliztion中的一种方法，也是最常用的z-score方法。
### Batch norm
上面说的normalization方法可以推广到neural networks中，对于nerual networks中的某一个layer来说，可以看做是以此计算过程，在这个过程中，我们可以引入normalization，对于\\(z^{(i)}\\)来说：
$$ \mu = \frac{1}{m} \sum_i z^{(i)}$$
$$ \sigma ^2= \frac{1}{m} \sum_i (z^{(i)}- \mu)^2 $$
$$z^{(i)}_{norm}= \frac{z^{(i)}- \mu}{ \sqrt{ \sigma^2 + \epsilon}}$$
$$z^{N(i)}= \gamma z^{(i)}_{norm} + \beta$$
然后我们用最终的\\(z^{N\[l](i)}\\)来替换\\(z^{\[l](i)}\\) 就可以，其中\\( \gamma\\)和\\(\beta\\)是两个parameter，可以通过gradient descent来更新，这两个parameter存在的意义，就是可以调整normalization映射的Gaussian distribution，而不是统一映射到Normal distribution值得注意的是，\\(\epsilon\\)是一个很小的数，用来避免分母分0的情况。

如果\\(\gamma = \sqrt{ \sigma^2 + \epsilon}\\)且\\( \beta = \mu\\)的话，那么其实\\(z^{N(i)}=z^(i)\\)的，大家可以算算，这种情况下，就是相当于没做normalization.
### Batch norm on neural networks
对于neural networks，输入\\(X\\)通过parameter\\(w^{[1]}\\)和\\(b^{[1]}\\)得到\\(z^{[1]}\\)，通过\\(\beta\\)和\\(\gamma\\)获得\\(z^{N[1]}\\)，经过active function后获得\\(a^{[1]}\\)，通过\\(w^{[2]}\\)和\\(b^{[2]}\\)获得\\(z^{[2]}\\)，一直到最后的输出层，完成forward propagation.

在整个过程中，一共有四个parameters，分别是\\(w^{[l]}\\)，\\(b^{[l]}\\)，\\( \beta^{[l]}\\)，\\( \gamma^{[l]}\\)，我们都知道：
$$z^{[l]}=w^{[l]}a^{[l-1]}+b^{[l]}$$
但是，我们在做batch normal的时候，首先先把\\(z^{[l]}\\)映射到了期望为1方差为0的Gaussian distribution上，这就意味着\\(b^{[l]}\\)是可以忽略掉的，因为在batch normal的时候也会被减去，因此，我们的parameter只有三个，即：\\(w^{[l]}\\)，\\( \beta^{[l]}\\)，\\( \gamma^{[l]}\\)

在backforward的时候，我们和普通的neural networks一样，只是可以不用再去计算\\(db\\)
### Solve covariate shift
什么是covariate shift？简单的理解，就是模型需要随着样本的变化而变化，Ng举得例子就很直观，你的training set里都是黑猫，这样获得的模型，对于花猫识别就是不适用的，这就叫covariate shift. 其实，batch norm改善neural networks效果的原因，就可以理解为solve covariate shift的过程。

OK，我们来详细看看原因，我们有一个如图的neural networks：
![](http://otmy7guvn.bkt.clouddn.com/blog/8/8-1.png) 
在标示出的位置，有parameter\\(w^{[3]}\\)和\\(b^{[3]}\\)，如果我们盖住前面的部分，那么我们将获得如图的neural networks
![](http://otmy7guvn.bkt.clouddn.com/blog/8/8-2.png) 
对于neural networks来说，相当于黑箱输出了\\(a^{[2]}\\)，而\\(a^{[2]}\\)的值其实是不固定的，每一次iteration后都不是一样的值，这就产生了covariate shift问题。

但是，batch norm可以将\\(a^{[2]}\\)的期望和方差限制到\\(\beta\\)和\\(gamma\\)限定的范围内，以此**极大限度**的缓解了covariate shift现象。

另外，batch norm还可以有一些regularization的作用，由于每次mini-batch gradient descent中batch norm作用的sample均不一样，类似于dropout的效果，但是，我们一般不会把norm batch列入regularization范畴内。

## Reference
* [Deep learning-Coursera Andrew Ng](https://www.coursera.org/specializations/deep-learning)
* [Deep learning-网易云课堂 Andrew Ng](https://mooc.study.163.com/course/deeplearning_ai-2001281003#/info)