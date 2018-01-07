---
title: spark工作流程简析
date: 2018-01-07 18:44:37
tags: spark
categories: spark
---
Hello，有一个月没写blog了感觉很自责，必须整起来！最近由于工作上遇到的一些调优困难，让我对Spark有些敬畏，所以集中的研究了下鬼魅玄学Spark，和大家分享一下。首先先来看看spark的基本工作流程。
<!--more-->
## Work Flow
和hadoop一样，spark也是master-slave机制，Spark通过driver进程，将task分发到多个executors上并发进行计算。整个driver和所有的executors组成了一个spark application，每一个application是运行在cluster manager上的，Spark本身集成了standalone cluster，当然，Spark还可以运行在赫赫有名的YARN和Mesos上。我平时使用的公司集群都是基于YARN cluster manager的，因此本文重点探讨基于YARN的spark。

下图就是spark在cluster manager下的整体工作流程。
![](http://otmy7guvn.bkt.clouddn.com/blog/15/15-1.png) 

## The Driver
Driver是整个application最核心的部分，他运行的是application的main方法，它伴随这整个application的生命周期，driver进程的结束就会带来整个application的结束。

对于所有的Spark任务，他们其实都是实现RDD的transformation和action操作，而这些操作，最后是需要driver将他们转化和分发成tasks，然后才可以去执行。所有的user program都会被driver通过DAG(directed acyclic graph)转化成实际的tasks执行计划，除此之外，driver还会在tasks执行的期间，监控executor上的tasks，并且保证他们拥有健康而合理的资源。

## Executors
Executors是Spark application的执行者，他们也是伴随着application的生命周期而存在的，值得注意的是，**Spark job在executors执行失败的情况下依然可以继续进行**。Executors会对具体的tasks的执行结果返回给driver，同时给缓存的RDD提供存储空间。

## Detail
下面，我们一起看看整个Spark application中，driver和executors的都会起到什么作用。Spark通过spark-submit指令提交application，并且spark-submit为application加载driver并且加载用户指定的main方法。与此同时，driver通过cluster manager申请供executors工作的资源，随后cluster manager根据driver的请求加载executors。然后整个application开始执行，在这个过程中，根据RDD的transformation或者action，driver把这些任务以tasks的形式，源源不断的传送给executors，于是executors不停地进行计算和存储的任务。当driver的main方法执行结束的时候，他会结束掉executors并且释放掉资源。这就是driver-executors的整体工作流程。

## Reference
* [Karau, Holden, et al. Learning spark: lightning-fast big data analysis. " O'Reilly Media, Inc.", 2015.](http://shop.oreilly.com/product/0636920028512.do)