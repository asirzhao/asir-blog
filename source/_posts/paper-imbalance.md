---
title: Reading Notes-Class Imbalance, Redux
date: 2017-09-10 13:21:56
tags: 
	- imbalanced data
	- undersampling
	- bagging
categories: reading notes
---
再次感谢优男，向我提出了又一个尖锐的问题，使得我有机会思考和研究，并且最终可以看到这篇paper，并且最后可以分享给大家。

我个人在工作之中遇到过imbalanced data的问题，我只是直观的感受到，imbalanced data的最后效果往往不是很棒，网上也只是给出了oversampling和undersampling的建议，并没有提及这其中的一些缘故，今天我们一起通过这篇paper来学习学习。
<!--more-->
## Notes
我们假设有positive和negative两类sample，其中positive samples符合\\(P(x)\\)的Guassian分布，negative samples符合\\(G(x)\\)的Guassian分布，分类平面将空间划分成positive region\\(\cal R^{+} \_{w}\\)和negative region\\(\cal R^{-} \_{w}\\)，如下图所示：
![](http://otmy7guvn.bkt.clouddn.com/blog/5/5-1.png) 
图中\\(w^{ \*}\\)是理想的分割平面，\\(w^{ \*}\\) 应该是使loss最小的取值，即
$$w^{*}= \arg\underset{w}{\min} \cal L^{*}(w)$$
对于loss值，其实就是分类中被错分的fn(false negative)和fp(false positive)的期望值，显然，通过minimun该loss得到的会是图中的\\(w^{*}\\)，因为这个分类平面所带来的error明显是最少的。
$$\cal L^{*}(w) = \cal C_{fn} \int _{\cal R^{w} _{-}} \it P(x)dx + \cal C_{fp} \int _{\cal R^{w} _{+}} \it G(x)dx$$
对于整个数据集\\(\cal D \\)来说，我们假设数据量较少的一类(paper中设定positive类较少)所占比例为\\(\pi\\)(小于0.5)，那么对于带有比例\\(\pi\\)的数据集\\(\cal D_{\pi}\\)，全局期望是
$$\bf E_{\cal D_{ \pi}} [\cal L(w)]=\pi \cal C_{fn} \int _{\cal R^{w} _{-}} \it P(x)dx + (1- \pi) \cal C_{fp} \int _{\cal R^{w} _{+}} \it G(x)dx$$
此处，我个人的理解是，在两类数据均衡的情况下，全局情况下的期望其实是和上面的loss等价的，但是imbalanced data带来了不均衡的因子\\(\pi\\)，因此，两个公式不再等价。

OK，既然不等价，那么问题就来了，paper上说，通过最小化全局期望获得的\\(\hat w\\)，是向着较少数量类别的样本倾斜，也就是第一幅图中，向较少的postive那边skewed，原因是因为\\( \cal R \_{+} ^{ \hat w} < \cal R \_{+} ^{w^{\*}}\\), 也就是说，\\(\hat w\\)分割的positive region面积小于\\(w^{\*}\\)分割出的面积，面积的减小势必导致分割平面向positive类别方向偏移。

遗憾的是，关于面积的证明我实在看不明白，也email了一些人，也没有得到一个满意的答案，如果有朋友看明白了的话，**记得留言或者email我！**

到了这里，paper大概介绍了undersampling的裨益，undersampling的核心其实就是消除前面提到的比例\\(\pi\\)，让它趋近于0.5后，分类平面\\(\hat w\\)就会趋近于理想分类平面\\(w^{*}\\)。

这里，作者提出了一个bagging方法，就是多次做undersampling，最后最结果做bagging可以获得更好的效果，如下图
![](http://otmy7guvn.bkt.clouddn.com/blog/5/5-2.png) 
paper还对比了其他的方法，比如Weighted Empirical Cost Minimization(如weighted SVM)和SMOTE方法效果不如bagging undersampling，我上一幅图说明下SMOTE的缺点，更多细节，大家可以详细看看paper，如图：
![](http://otmy7guvn.bkt.clouddn.com/blog/5/5-3.png) 
SMOTE方法是随机选择方向生成新的sample，但是如果新的sample产生了图中位置，则效果不会很好。

OK，今天就这么多，记得看明白了中间的推导一起分享啊！
## Reference
* [Wallace, Byron C., et al. "Class imbalance, redux." Data Mining (ICDM), 2011 IEEE 11th International Conference on. IEEE, 2011.](https://pdfs.semanticscholar.org/a8ef/5a810099178b70d1490a4e6fc4426b642cde.pdf)
* [PPT-Class Imbalance, Redux](https://www.google.co.jp/url?sa=t&rct=j&q=&esrc=s&source=web&cd=3&cad=rja&uact=8&ved=0ahUKEwiVmqXlkprWAhVEsJQKHYJiCf4QFgg8MAI&url=https%3A%2F%2Fcourse.ccs.neu.edu%2Fcs6140sp15%2F4_boosting%2Fslides%2Fwallace_imbalance_icdm_11_for_class_2012_final.pptx&usg=AFQjCNG6GpjKeinzCsXrZWWY1edtbBMgog)