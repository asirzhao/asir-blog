---
title: Learning notes-Deep Learning, course3, week2
date: 2017-10-18 21:28:12
tags: 
	- learning strategy
	- transfer learning
	- multi-task learning	
categories: learning notes
---
Hi all，course3来到了week2，本周的课程依然主要是关于一些learning strategy，这些方法相当实用。虽然不是什么具体的算法，但都都是Ng在科研和工作中积累下来的宝贵经验，对于实际问题十分有效。

我们一起来看看。
<!--more-->
## Error analysis
### Carry out error analysis
按照通常的流程，在进行training过程后，我们在dev set会进行模型的测试，如果dev error比training error大很多的话，我们应该去排查问题的症结所在呢？Ng给出了solution

例如在cat recognition中，我们发现错分的sample有很多dog图像，还有很多猫科动物的图像，还有一些是模糊的cat图像。于是我们自然而然的想到三种解决方案：
* 解决狗错分为猫的问题
* 解决猫科动物被错分成猫的问题
* 提升模糊图像被误分的问题

可是由于我们精力和时间都有限，需要找出误分最主要的问题，因此我们要做的，是把所有错分的图像罗列出来，或者随机抽样一定的图像，分析每种错误它有多少，占错分图像多少比例。我们来看截图
![](http://otmy7guvn.bkt.clouddn.com/blog/10/10-1.png) 
每一个错分的图像都会进行标签化的统计，最后通过统计每一个标签，找出影响错分最严重的因素，作为我们的改进方向。
### Clean up incorrectly labeled data
在常见的错误中，错误的label是一种很常见的问题，这种问题往往来自于标注时候，错误的label会对training造成误导。

首先，对于training set，来说，incorrectly labeled data应该怎么处理？首先，Ng告诉了我们一个性质：

> DL algorithms are quite robust to random errors in the training set

DL因为其自身的robust性质，当training set中有少许的，随机产生的incorrectly labeled data时，效果并不会有多差，我们完全不需要去管他。但是，当这incorrectly labeled data很多时就不行了，因为它们带来的是systematic errors，极端的想，如果把所有的白狗都错误的标注成了猫，那么这个cat recognition系统一定不会好，因为它一定会把白色的狗判断成为猫。

再来看看dev/test set中的incorrectly labeled data，对于这个问题，我们要做的是，评估incorrectly labeled data对dev error带来了多少贡献，解决的过程也是类似的，来看截图：
![](http://otmy7guvn.bkt.clouddn.com/blog/10/10-2.png) 
我们把incorrectly labeled也作为一个要素或标签，放在错分图像分析的过程中，
看看最终的统计结果，再决定incorrectly labeled data是不是影响dev error的主要原因，是否值得我们去fix it up.

最后，关于correcting incorrect dev/test set example，Ng给出了一些建议：
> Apply same proces to your dev and test sets to make sure thry continue to come from the same distribution.

在修正的过程中，一定要保证dev和test set同时被修正，如果他们不再符合同一distribution，那么会对于后续的评价带来一些问题。

> Consider examining examples your algorithm got right as well as ones it got wrong.

我们在更正的时候，不能只是看被错分的图像，对于被正确分类的，也有可能存在incorrect labeled 的情况。

> Tran and dev/test data may now come from slightly different distribution

正如刚才讲的，DL对于training有一定程度的robust性，incorrect labeled data可能不会对training set带来这些问题，在这种情况下，我们可以不用去更正training set，这种情况，我们是可以接受的。

### Build up quickly and iterate
最后Ng用一个speech recognition作为例子，我们首先要分析出可能影响效果的一些因素：
* Noisy background
* Accented speech
* Far from microphone
* Young children's speech
...
针对这些问题，我们该如何构造我们的模型呢，Ng给出了建议
* Set up dev/test set and metric
* Build initial system quickly
* Use bias/variance analysis & error analysis to prioritize next steps.

总而言之，guideline是
> Build your first system quickly, then iterate.

## Mismatched training and dev/test data
### Training and testing on different distributions
之前我们再三强调过一个尖锐的问题，那就是training/dev/test set一定要在同一个distribution下，但是实际上，愿望总是美好的而现实很残酷，我们总是会面对一些training and testing on different distribution问题。

例如在猫识别的任务中，我们需要将这个模型部署到手机app上，我们手上的数据只有10k是从手机拍摄获得的，而有200k的数据是从网络上获得的，这两种图像显然不属于同一distribution，我们应该怎么办？

首先来看option1，我们将所有的210k数据充分混合在一起，其中205k作为training set，2.5k作为dev，2.5作为test set。这样看起来是一个很不错的方法，但是，确实很不好的一个方法，为什么这么说呢？

在整个过程中，dev/test set其实扮演了一个非常重要的角色，它决定了我们的target，也就是整体的优化方向。在这个例子中，我们要优化的方向是app上的图像，而这种data set分割方法和我们的task target并不符合，因此并不优秀。

我们再来看option2，我们将200k的来自网络的图片全部放入training set，然后将10k的app数据，5k放入training set，2.5k作为dev，2.5作为test，这样做的话，dev/test决定的target 和我们的task target是一致的，所以长远来看，虽然option2的training/dev set并不是同一distribution，但是从长远看它的效果还是很不错的。
### Bias & variance with mismatched data distribution
在training/dev/test set符合同一distribution的时候，我们通过比较training error和dev error就可以定性是否存在high variance的问题。但是，当training set和dev set不符合同一distribution的时候，这个判断就显得有些困难了。我们应该怎么处理呢？

这时候，我们可以从training set中取出一小部分数据，命名为training-dev set，这部分数据将不再进行training，而是作为评判training效果的一个set，此时我们就有了training error，training-dev error和dev error三个error，再结合human error，training error和training-dev error之间的差值可以反映出模型是否有high bias或者variance，这样可以更科学的来评判模型效果。相应的，training-dev error和dev error相差越多，data mismatch的程度越大。
### Addressing data mismatch
我们如何addressing data mismatch呢，首先我们来看看Ng的两条guideline：
* Carry out manual error analysis to try to understand difference between training and dev/test sets
* Make training data more similar;  or collect more data similar to dev/test sets

理解一下，首先我们要通过人工的analysis去分析出造成training set和dev set之间distribution不同的原因，比如语音识别中的有无汽车噪声等等；然后我们需要根据这些差别，让training set和dev set更加的相似，甚至相通。

但是要注意的是，我们在这个过程中，要避免出现overfitting的情况出现，例如Ng举出的例子，在识别车内的人声过程中，我们可以通过人工的合成汽车声音与人的声音让training set和dev set更加的相似，但是如果我们的只用一段汽车噪音循环往复的去做合成，例如吧1min的汽车噪声循环的合成到1h的人声中，那结果一定是不尽如人意的，因为出现了overfitting.
