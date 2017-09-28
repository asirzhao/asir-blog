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
### Exponentially weighted averages
Exponentially weighted averages，也被称为moving averages，是一种综合历史数据的加权平均方法，课程中Ng用了伦敦一年的气温变化曲线作为例子，对于固有变量\\(\theta\\)来说，我们要求的平均值\\(v\\)应该是
$$v_0 = 0$$ 
$$v_1 = \beta v_0 + (1- \beta) \theta_1 $$
$$v_2 = \beta v_1 + (1- \beta) \theta_2$$
$$v_3= \beta v_2 + (1- \beta) \theta_3$$
$$\cdots$$
$$v_t= \beta v_{t-1} + (1- \beta) \theta_t$$
其中\\(\beta\\)是一个因子，它决定了moving averages大约向前平均了\\(\frac{1}{1- \beta}\\)个\\(\theta\\)值，例如\\(\beta = 0.9\\)，那么大约向前平均了10个值，并且是向前按指数衰减加权获得的平均值。
### Bias correction
在exponentially weighted averages中，有一个问题很尖锐，那就是在最初的求解过程中，由于\\(v_0=0\\)，导致前面的数字结果距离正确结果较小，如下图所示，紫色曲线是获得的结果，而在起始位置的值明显是偏小的。
![](http://otmy7guvn.bkt.clouddn.com/blog/7/7-1.png) 
此时，我们引入bias correction，原理也很简单，就是此处我们不使用\\(v_t\\)作为最终的结果，而是使用\\( \frac{v_t}{1- \beta^{t}}\\)作为最后的结果，通过bias correction，我们会获得绿色的曲线。

我们可以看到，起始绿色钱和紫色曲线在最后基本没有差别，几乎重合，但是在曲线开始的时候，绿色曲线比紫色曲线更加逼近真实情况，因此，Ng给我们以下建议：
* 当我们不关注moving averages initial value大小的时候，我们可以不使用bias correction
* Bias correction对于initial value效果更好
## Gradient descent optimization
### momentum
在gradient descent中，我们经常会遇到一种情况，如图所示：
![](http://otmy7guvn.bkt.clouddn.com/blog/7/7-2.png) 
在水平方向上，我们希望更快的下降，而在垂直方向上我们希望更小的下降速率，以避免过多的iteration，针对这种情况，我们讲moving averages的思想带入进来，这就是momentum方法。

在momentum中，我们的每次迭代中：
$$v_{dW}= \beta v_{dW}+(1- \beta)dW$$
$$v_{db}= \beta v_{db}+(1- \beta)db$$
$$W:=W- \alpha v_{dW}$$
$$b:=b - \alpha v_{db}$$
在很多的文献中，上面的\\(1- \beta\\)项被省略掉了，这样做只是将等式等量做了缩放，并不影响实际的效果，Ng表示，两种方式的momentum几乎没有差别，大家可以放心使用。

实际上，momentum可以理解为将数次之前迭代过程中的gradient变化也带入到了这次迭代过程中，我个人认为就是带入了一种gradient变化的趋势，这样可以更好的控制gradient descent的方向和大小。例如上图中的纵向方向中，加入momentum后可以轻松的将纵向梯度正负抵消到近似0，这样就可以减少在gradient descent在纵向的反复迭代，因为，那既是无用功，也是我们不愿意看到的情况。

另外，Ng给我们了一个\\(\beta\\)的理想取值，既0.9，这个值大约取了前10次迭代结果的moving averages。另外，Ng表示，在momen中很少使用bias correction，因为10次迭代之后，这种问题很快就会消除，而一般的gradient descent，iteration次数远远大于10次。
### RMSprop
除了momentum，还有一些类似的方法，例如大名鼎鼎的RMSprop(root means square prop)，在这个方法中，主要体现了square的应用，我们来看看，在每次的迭代中：
$$S_{dW}= \beta S_{dW} + (1- \beta)(dW)^2$$
$$S_{db}= \beta S_{db}+(1- \beta)(db)^2$$
$$W:=W- \alpha \frac{dW}{\sqrt {S_{dW}}+ \epsilon}$$
$$b:=b- \alpha \frac{db}{\sqrt {S_{db}} + \epsilon}$$
\\(\epsilon\\)是为了防止分母为0的一个item，取值建议为\\(10^{-8}\\)

假设在gradient descent中，\\(W\\)下降速率太低，也就是\\(dW\\)太小，那么在RMSprop的迭代中，\\(dW\\)将会除以一个很小的值\\( \sqrt{S_{dW}}\\)，也就是\\(W\\)将会减去一个较大的值，反之亦然。

通过这种方式，我们缓解了上图所示的情况，改善了gradient descent的合理性，同时可以使用更大的\\(\alpha\\)去实现更快的gradient descent.
### Adam
我们看到了momentum和RMSprop优化方法的厉害之处，现在Adam方法横空出世，他融合了momentum和RMSprop，他是如何融合的呢，我们把momentum中的\\( \beta\\)命名为\\( \beta\_1\\)，把RMSprop中的\\( \beta\\)命名为\\( \beta\_2\\)，在第\\(t\\)次迭代中：
$$v_{dW}= \beta_1 v_{dW}+(1- \beta_1)dW, \qquad v_{db}= \beta_1 v_{db}+(1- \beta_1)db$$
$$s_{dW}= \beta_2 s_{dW}+(1- \beta_2)(dW)^2, \qquad s_{db}= \beta_2 s_{db}+(1- \beta_2)(db)^2$$
加上bias correction后
$$v^{corrected}_{dW}= \frac{v_{dW}}{1- \beta^t_1}, \qquad v^{corrected}_{db}= \frac{v_{db}}{1- \beta^t_1}$$
$$s^{corrected}_{dW}= \frac{s_{dW}}{1- \beta^t_2}, \qquad s^{corrected}_{db}= \frac{s_{db}}{1- \beta^t_2}$$
$$W:=W- \alpha \frac{v^{corrected}_{dW}}{ \sqrt{s^{corrected}_{dW}}+ \epsilon}$$
$$W:=W- \alpha \frac{v^{corrected}_{db}}{ \sqrt{s^{corrected}_{db}}+ \epsilon}$$
其中，\\(\epsilon\\)是为了防止分母为0的一个item，取值建议为\\(10^{-8}\\)，\\(\beta\_1\\)建议取值0.9，\\(beta\\_2\\)建议取值0.999。