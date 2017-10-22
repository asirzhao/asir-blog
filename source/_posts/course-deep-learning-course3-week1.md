---
title: Learning Notes-Deep Learning, course3, week1
date: 2017-10-12 12:34:05
tags: 
	- learning strategy
	- orthogonalization
categories: learning notes
---
课程3主要讲的是deep learning中的一些strategy，这些strategy可以帮助我们快速的分析模型所存在的问题，避免我们的优化方向有偏差而导致的人力以及时间的浪费，这一点对于团队尤其重要。

我们一起来recap一下week1的课程
<!--more-->
## Orthogonalization
对于ML task来说，有众多因素影响最终的效果，这些因素相互犬牙交错，因此我们在提升模型效果的时候，一定要把所有的因素orthogonalization一下，Ng举的例子就很形象，就像显示器的控制按钮一样，每个按钮各司其职，一个控制高度，一个控制宽度，一个控制大小，一个控制梯度，通过各自调整每一个按钮，我们可以很好的完成画面调整。

对于orthogonalization优化模型，Ng给出了4方面的建议：
* Fit training set well in cost function
* Fit development set well on cost function
* Fit test set well on cost function
* Performs well in real world

我们详细来看看这四条：

对于第一条，首先模型必须要对于training set有良好的拟合效果，如果这点达不到的话，模型一定是**high bias**，也就是**under fitting**了，那么我们必须尝试通过more complex的模型，bigger neural networks或者是longer training time去更充分的拟合training set.

对于第二条，在很好的拟合training set的前提下，我们就要看看development set的效果了，如果对于development set fit效果不好的话，那基本上就是**high varience**，也就是**over fitting**的问题了，这时候，regularization或者more training data可以解决解决这个问题。

对于第三条，在符合上两条的前提下，如果模型在test set上表现不佳，我们就需要更大的development set去涵盖更多的情况，并通过扩充后的development set重复第二条的检验

对于最后一条，在符合上三条的前提下，如果模型在real world中表现不佳，那么很大程度上是因为我们的development 和test set与real world相差比较多，比如猫脸检测中，我们的data set都是高清的图像，但是real world 中，都是像素很低的图像。因此，我们需要让development 和test set更接近与real world，然后重复上面的步骤。

通过这四个步骤，我们就可以通过orthogonalization来完成模型的调整

## Metric
### Single number evaluation
Metric无疑是ML task中很重要的环节，通过metric，我们可以评估不同模型之间的优良差异，并且可以选择出最理想的模型。

但是metric指标琳郎满目，例如对于两个模型，模型A的precision高于B的，但是A的recall又低于B，这时候就不太好评价两个模型，在这种情况下，我们需要采用单一的数字评价指标，例如我们可以用F1-score来进行评估，single number evaluation metric是我们做metrics时一定要注意的
### Satisficing   and optimizing
在某些情况下，例如我们不仅仅要求模型的指标，还对其他的，例如模型时间会有要求，如果一个模型有很高的模型accuracy，但是却很耗费时间，那是我们不能接受的，如下图例子：
![](http://otmy7guvn.bkt.clouddn.com/blog/9/9-1.png) 
图中的accuracy是optimizing metric，通常更高的accuracy就代表这classifier更加的优秀；但是，这里还有一个必须低于100ms 的running time作为satisficing metric，通常来说，如果我们有\\(N\\)个metrics，那么我们的optimizing metric必须只有一个，剩下的\\(N-1\\)metrics 都是satisficing metrics，只要以threshold形式进行限定就可以了。
## Data set
### Distributions
对于training set，dev set和test set来说，所有data一定要保证服从同一个data distribution，例如猫脸实验，如果training set是高清大图而dev set是模糊图像，那么最终一定很难获得理想的metric.

最重要的是，你所构建模型的data，一定和模型应用场景的data在同样的distribution下，Ng给出的guideline是
> Choose a development set and test set to reflect data you expect to get in  the future and consider important to do well.

### Size of data set
传统的ML task中，dataset的分布一般如下图所示：
![](http://otmy7guvn.bkt.clouddn.com/blog/9/9-2.png) 
但是在big data时代，一般采用下图：
![](http://otmy7guvn.bkt.clouddn.com/blog/9/9-3.png) 
Ng同样给出了guideline：
* Set uop the size of test set to give a high confidence in the overall performance of the system.
* Test set helps evaluate the proformance of the final classifier which could be less than 30% of the whole data set
* The development set has to be big enough to evaluate different ideas

## Change data set and metric
### Change data set 
这个问题的原因，其实就是data set不在同一distribution下的问题，如果模型在dev和test set 上都有很好的表现和metric，但是在real world中效果并不好，那么我们要做的一定就是改变data set，让dev/test set与real world在同一distribution下
### Change metric
我们使用ML来解决现实中的问题时，metric也不是一成不变的，需要根据具体的情况做出一些改变，例如，色情图像识别中，我们误讲非色情识别维色情图像，是可以接受的，但是将色情图像识别成非色情图像则是不可接受的，相似的，风控系统中，将风险用户分类为正常用户的错误，比把正常用户分类为风险用户的错误要严重很多，因此，在类似的场景下，我们的metric需要随着业务场景做出一些改变。

正常情况下，我们计算的模型error是：
$$Error = \frac{1}{m_{dev}} \sum^{m_{dev}}_{i=1} \mathcal \{ \hat{y}^{(i)} \neq y^{(i)} \}$$
但是在上述场景中，我们需要加入一个权重，来对两种不同错误加以区分
$$ w^{(i)}=\left\{
\begin{aligned}
1 \quad x^{(i)}notpron \\
10 \quad x^{(i)} pron \\
\end{aligned}
\right.
$$
$$Error = \frac{1}{ \sum w^{(i)}} \sum^{m_{dev}}_{i=1} w^{(i)} \mathcal \{ \hat{y}^{(i)} \neq y^{(i)} \}$$
这样就把两种不同的问题区分开了。
## Improve model performance
模型performance的提升是模型的核心问题，我们如何确定模型调整的大体方向，Ng给出了如下的图
![](http://otmy7guvn.bkt.clouddn.com/blog/9/9-4.png) 
我们默认human-level是很接近理论误差，也就是Bayes error，我们需要比较human-level，training error和dev error这三者之间的关系，human-level和training error之间的差值更大的话，我们就需要去减小bias，反之，我们需要去减少variance，具体的方法，还是我们之前的老套路。

## Reference
* [Deep learning-Coursera Andrew Ng](https://www.coursera.org/specializations/deep-learning)
* [Deep learning-网易云课堂 Andrew Ng](https://mooc.study.163.com/course/deeplearning_ai-2001281003#/info)
