---
title: Reading Notes-Practical lessons from predicting clicks on ads at facebook
date: 2017-08-23 11:30:43
tags: 
	- gbt
	- logistic regression
	- gradient descent
categories: reading notes
---
OK，今天我们来review一篇经典的paper，这篇paper是3年前facebook的研究成果，关于gbt和lr的结合，这个搭配对于近几年的CTR预测以及推荐系统的发展都产生了深远的影响。虽然已经很难被称为一篇新paper了，但是还是值得我们去看看。

我们一起简单看看这篇paper的核心point.
<!--more-->
## Notes
传统CTR预测中，logistic regression一直有着很好的效果，lr不仅可以线性分类，同时也可以给出样本属于该类别的posterior probability

但是传统的lr也有着本身的缺憾，lr本身就是liner分类器，对于线性不可分的features效果不是很理想。同时在对于连续feature离散化的时候，效果很大程度依赖于离散分桶的人为经验。

该paper提出了一种依靠gbt进行feature transform的方法，不多说废话，我们直接上图
![](https://github.com/JoeAsir/blog-image/raw/master/blog/3/3-1.png)
这就是这篇paper最最最核心的部分了。

> Input features are transformed by means of boosted decision trees.The output of each individual tree is treated as a categorical input feature to a sparse linear classifier. Boosted decision trees prove to be very powerful feature transforms.

从图中我们可以看到，原始feature被gbt进行了transform，样本落入到哪个tree node，则该位置1，其他位置0，随后再进入线性分类器lr中进行最后的分类。

假设有一个sample，在图中所示的模型中，gbt有两棵树，从左到右是tree1和tree2，tree1中sample被分到了第一个tree node，tree2中sample被分到了第二个tree node，那么最终transform得到的new sample就变成了(1,0,0,0,1)

通过gbt的transform后，feature不仅从非线性转换成了线性（类似于SVM的kernel效果），而且feature被完全的离散成了0-1稀疏feature，无论从线性可分还是特征稀疏的角度上，都变得比原始feature更加理想！

因为是一篇相对老一些的paper，所以我叙述的比较简单，大家可以get到gbt+lr这个模型的基本原理就可以了。我自己在私下也用python写了一个简单的demo，感兴趣的朋友[可以看看](https://github.com/JoeAsir/Machine-learning-demo/blob/master/algorithm/gbtWithLogisticRegression/gradient_logistic.py)，欢迎提出意见，欢迎folk！

## Reference
* [He, Xinran, et al. "Practical lessons from predicting clicks on ads at facebook." Proceedings of the Eighth International Workshop on Data Mining for Online Advertising. ACM, 2014.](http://quinonero.net/Publications/predicting-clicks-facebook.pdf)
