---
title: Imbalanced data 问题总结方法汇总
date: 2017-11-11 23:01:09
tags:
	- imbalanced data
categories: machine learning
---
Hello，大家好，双十一真的很累，一直在加班，忙里偷闲看了[A systematic study of the class imbalance  problem in convolutional neural networks](https://arxiv.org/pdf/1710.05381.pdf)，感觉paper呈现的研究内容感觉很一般，但是，paper中关于imbalanced data的solution方法倒是写的很不错，也勾起了我对于这一块总结的欲望。之前也写过一篇关于imbalanced data的paper notes，但是对于这一块的具体方法总结还不是很足够，于是用这篇paper为主线好好sum up一计。

我们来一起看看。
<!--more-->
## Data level methods
首先我们来看一看data level methods，这类方法有一个共性，那就是通过改变data的数量来完成对imbalanced data problem的解决。
### Oversampling
Oversampling可以说是最直观的solution之一，它的核心思想是，对于较少一类别的samples，过此重复采样，以此让两种类别的样本接近平衡。但是，对于一个sample多次重复训练，很有可能带来overfitting，因此，简单粗暴的重复采样并不可取。因此，很多改进的版本应运而生：
#### SMOTE
SMOTE算法是一种经典的oversampling方法，它的主要思想是对较少数目类别的样本，随机抽取\\(m\\)个样本，对于随机抽取出的样本，每个样本选取距离最近的\\(n\\)个样本，在他们的连线上随机选取一个点，作为较少类别的补充样本。

通过这种方式，SMOTE可以对较少类别样本进行扩充，进而实现oversampling，平衡数据分布。
#### Cluster-base oversampling
Cluster-based方法的最大特点莫过于最开始对数据进行一个聚类分析，数据会变成数个cluster，然后对于每一个cluster在进行数据的oversampling，这样可以保证oversampling后的数据最大程度上的保持原有的分布。

### Undersampling
与oversampling相对应的则是undersampling，undersampling的核心思想是对于较多类别的samples抽样，使得两个类别数据趋于相近。但是，随机抽样获得会使得类别丧失很多的信息，甚至导致数据分布发生改变。
#### One-sided selection
one-sided selection的主要思想是，为了保证数据整体的分布，我们优先去除靠近边界的样本，这样可以保证较多分类的数据分布。
#### Data decontamination

## Classifier level methods
下面我们来看看通过改变classifier level来解决imbalanced data的方法，这类方法侧重于分类器本身的一些性质而并非两类数据的个数。
### Thresholding
我在之前的博客中聊到过，imbalanced data的分类平面会倾向于较少数据的分类一侧，所以我们可以通过改变类别预测的probabilty的threshold来修正分类平面。常用的方法就是加入关于类别数目的prior probability：
$$y_i(x)=p(i|x)= \frac{p(i) \cdot p(x|i)}{p(x)}$$
### Cost sensitive learning
Thresholding方法其实对已经train好的模型的采取的一种方式。相应的，我们在模型训练的时候就来消除imbalanced data的一些影响，如何做到呢？答案就是cost function.

我们可以通过调整learning rate，加强对cost比较大的samples，并且最终的优化目标从标准的cost function变成misclassification cost，如此就可以解决imbalanced data的问题了
### One-class classification
该方法可以说是换了一种思维看问题，我们不再将classification作为我们的task，而是变成了对于一种异常检测的问题。我们只是着眼于较多samples的类别，认为另一类别的samples是一种异常值。

当然，这种方法适合那种极端的imbalanced data，对于一般的情况并不一定很适用。

## Reference
* [Buda, Mateusz, Atsuto Maki, and Maciej A. Mazurowski. "A systematic study of the class imbalance problem in convolutional neural networks." arXiv preprint arXiv:1710.05381 (2017).](https://arxiv.org/pdf/1710.05381.pdf)