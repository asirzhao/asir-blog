---
title: Learning notes-Deep Learning, course2, week1
date: 2017-09-24 14:06:08
tags: 
	- regularization
	- gradient descent
categories: learning notes
---
大家好，最近在学习Andrew Ng的Deep learning课程，于是决定写一些learning notes来recap和mark一下学到的知识，避免遗忘。由于该课程的course1比较基础，我个人认为没有mark的必要，所以从course2开始，按照week来mark.
<!--more-->
## Data set
在machine learning中，data set可以说是最重要的部分，区别于传统machine learning，deep learning中的data set分布更侧重于training，Ng建议我们讲data set分为三部分：
* training set——训练数据集
* dev/validation set——模型选择和参数调整，泛化能力测试
* testing set——模型效果测试
一定有很多人对于dev和testing set有一些疑问，最开始我也是懵逼的，来看看下面这段话
> Dev/Validation Set: this data set is used to minimize overfitting. You're not adjusting the weights of the network with this data set, you're just verifying that any increase in accuracy over the training data set actually yields an increase in accuracy over a data set that has not been shown to the network before, or at least the network hasn't trained on it (i.e. validation data set). If the accuracy over the training data set increases, but the accuracy over then validation data set stays the same or decreases, then you're overfitting your neural network and you should stop training.
Testing Set: this data set is used only for testing the final solution in order to confirm the actual predictive power of the network.

这三者的比例则是/99.5%/2.5%/2.5%/，这样的原因是因为deep learning中，数据量足够大而且deep learning的学习能力很强，大家一定注意这一点。当然，如果实在没有test set，但是有dev set也是可以接受的。
## Bias and variance
### 什么是bias和variance
bias & variance是machine learning 领域一个经典的辩证问题，在Ng经典的CS229中就重点的讲述过，具体的定义我不太想给出了，后续有时间可以专门写一篇，后面会给出一些资料链接。我们简单的看一幅图
![](http://otmy7guvn.bkt.clouddn.com/blog/6/6-5.png) 
左图就是一个典型的high bias situation，模型没有办法很好的拟合数据，这也就是我们常说的under fitting，右图则是典型的high variance situation，模型过分的拟合了training set，这就是我们最需要防范的over fitting.当然，中间的则是比较理想的状况。
### Solution
在实际的工作中，我们应该怎么分析自己模型的bias和variance情况呢，Ng给了我们一个流程图，如下：
![](http://otmy7guvn.bkt.clouddn.com/blog/6/6-2.png) 
首先检验是否存在high bias 情况，具体方法是在training set 和 dev set上计算error，对比training error和dev error，如果两者都很高，那么就是high bias，如果training error很小而dev error很高，那么一定是high variance，如果两者都很大，那么就是最差的情况了既high bias又high variance

对于high bias，我们可以通过更复杂的神经网络、更长的训练时间，更强的网络结构来解决这个问题；
对于high variance，我们可以通过更多的数据，regularization的方法来解决。
## Regularization
### L1&L2 regularization
这部分内容我就不多说了，我之前专门详细深入的讲述过L1和L2 regularization，大家可以去看一看。

唯一需要明确的一点是，在加入L1或者L2 regularization之后，在观测cost function convergence 的时候，**一定要带上regularization item**，否则结果是很难看到convergence的，这和regularization性质有很大的关系。 
### Dropout
Dropout是neural network中一种经典的regularization方法，经典到什么程度呢，我当年毕设课题中都用到了这个方法，而且效果超赞

Dropout方法的实质是**按比例随机隐藏**掉neural network中layer里的某些units，也就是说，再一次epoch中，只有一部分的units对应的weights和bias会得到更新，而下一次epoch中，则是另一部分units对应的weights和bias得到更新，如下图
![](http://otmy7guvn.bkt.clouddn.com/blog/6/6-3.png) 
那么为什么Dropout可以实现regularization效果呢，Ng告诉我们：
> Intuition:Can't rely on any one feature, so have to spread out weights

如何理解呢？加入dropout后，每个unit对应的weights和bias不能完全依赖上层units，因为他并不是每一次epoch都可以work on，因此在学习的过程中，见笑了over fitting的风险。实际上，dropout可以产生shrink weights的效果，和L2 regularization相似，因此也是一种regularization方法。

但是，dropout和L2 regularization唯一的区别在于，他很难给出一个regularization item，所以你没有办法画出cost function convergence的轨迹。
### Other methods
除了经典的L1、L2 regularization和dropout方法，还有一些防止over fitting的方法，例如图像处理中，我们可以用data augmentation，旋转，翻转，加噪声等方法。

还有一个early stopping方法，我们都知道，随着training 的epoch增多，模型对training set拟合会越来越好，随之带来的问题就是可能over fitting，我们可以通过early stopping，让模型在没有产生over fitting的时候停下来，效果可能会更好。
## Exploding/vannishing gradient
### 什么是exploding/vanishing gradient
对于deep learning，曾经最为棘手的问题就是exploding/vanishing gradient，甚至是限制deep learning发展的瓶颈，我们来一起看看。
 假设我们有一个**比较深**的neural network，假设一共有\\(l\\)层，对应的weights是\\(W^{[1]}\\)到\\(W^{[l]}\\)，bias是\\(b^[1]\\)到\\(b^{[l]}\\)，我们为了计算方便，假设bias均为0，active function为\\(g(z)=z\\)，那么，\\(y\\)就等于
$$\hat{y}=W^{[l]}W^{[l-1]}W^{l-2} \cdots W^{[3]}W^{[2]}W^{[1]}X$$
大家感兴趣的话可以验证一下，很简单的。

那么现在问题来了，当\\(l\\)很大的情况下，如果\\(W\\)元素都大于1，那么最后的结果就会非常非常大，甚至到无限大，这种情况叫exploding gradient；相应的，如果\\(W\\)元素都小于1，那么最后的结果就会特别小，甚至为零，这就是vanishing gradient.
### Solution
对于上面的问题，我们一般在weights初始化的时候做一些工作来解决可能出现的exploding or vanishing gradient。我们可以直观的理解一下，对于active function\\(g(z)\\)，假设bias为零，
$$z=w_{1}x_1 +w_{2}x_2+ \cdots +w_{n}x_n$$
我们要我们可以看到，\\(n\\)的增大，\\(w\\)会变小，我们让\\(w\\)始终保持以0为mean，1为varance的Gaussian distribution下，就可以很好的控制\\(w\\)的大小，那么我们可以看到，\\(var(w)= \frac{1}{n^{l-1}}\\)

在Ng的建议中，如果active function是sigmoid，我们一般取\\(var(w)= \frac{1}{n^{l-1}}\\)，如果是reLu，我们取\\(var(w)= \frac{2}{n^{l-1}}\\)，对于tanh，\\(var(w)= \frac{1}{n^{l-1}}\\)或者\\(var(w)= \frac{2}{n^{l-1}+n^{l}}\\)，这样可以很好的避免exploding or vanishing gradient.
## Gradient checking
### Gradient approximation
在调试neural network的时候，我们会经常做gradient check的工作，以确定整个network正常的运行，Ng在这里建议我们使用双边逼近的方法去做gradient check，这里我不做太多描述，主要上一张图：
![](http://otmy7guvn.bkt.clouddn.com/blog/6/6-6.png) 
通常来说，双边逼近的方法获得结果更加准确。
### Gradient checking notes
> 1. Don't use in training-only to debug(too slow)
2. If algorithm fails grad check, look at components to try to identify bug
3. Remeber regularization
4. Dosen't wrok with dropout
5. Run at random initialzation; perhaps again after some training.

## Reference
* [Deep learning-Coursera Andrew Ng](https://www.coursera.org/specializations/deep-learning)
* [Deep learning-网易云课堂 Andrew Ng](https://mooc.study.163.com/course/deeplearning_ai-2001281003#/info)
* [Bias and variance](https://en.wikipedia.org/wiki/Bias%E2%80%93variance_tradeoff)
