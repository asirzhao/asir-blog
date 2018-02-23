---
title: Spark Tuning
date: 2018-02-23 13:10:32
tags: spark
categories: spark
---
Hi, all, 最近一直在研究spark tuning方面的问题，深感这是一个经验活，也是一个技术活，查阅和很多资料，在这里mark一下。
<!--more-->
上次我们review了一下spark的work-flow，主要是基于spark on yarn的，同样的，我们在这里探讨的也主要是基于spark on yarn。
## Resource Allocation
### Some Configuration
Resource allocation是spark中一个非常重要的环节，给予一个application过少的resource会带了执行效率的低下和执行速度的缓慢；相反，过多的resource则会带来资源浪费，影响cluster上其他appllication的运行，因此，一个合适的resource allocation是非常非常重要的，我们来看看几个比较重要的parameter：
* num-executors: 表明spark申请executors的数目，我们可以通过设置spark.dynamicAllocation.enabled来让spark根据数据动态的分配executors，这样可以有效的提高资源利用率；
* executor-cores: 指定每一个executor的core数目，core数目决定了每个executor的最大并行task数目
* executor-memory: 指定分配给每一个executor的内存大小。

### Some Tips
* 对于executor来说，在过于大的memory上运行可能会带来比较高的GC(gabage collection) time，对于一个executor来说，建议给出的上限memory是64G；
* 由于HDFS在并行读写的时候存在一些瓶颈，因此每一个executor中最好不要超过5个并行任务，即cores数不要超过5个，有实验可以证明，spark在多executor少core的配置下执行效率更高；
* 相反的，对于executor来说，过分少的core，例如1个，将会使得executors数目变多，例如某个broadcast过程，需要传播到所有的executors上，那么过分多的executors会降低执行的效率。

## Memory Mangement
关于spark中的memory management，我们先来看一张图：
![](http://otmy7guvn.bkt.clouddn.com/blog/16/16-1.png) 
在图中我们可以看到，spark把memory分成了三部分，即spark memory、user memory和reserved memory，我们顺次来看看：
### Reserved Memory
所谓reserved memory，它就是系统预留下的一部分memory，用于存储spark的内部对象，默认大小为300m，绝大部分情况下，我们都不会修改这些参数。值得注意的是，当executor被分配的memory小于1.5倍的reserved memory时，将会抛出“please use larger heap size”的错误。
### User Memory
User memory用于储存spark的transfermation的一些信息，比如RDD之间的依赖信息等等，这部分内存默认大小为(Java Heap - 300M)*0.25，其中的300M其实就是上面提到的reserved memory.具体的大小要依赖于spark.memory.fraction参数，这个参数决定了user 和 下面要讲到的spark memory的分配比例。
### Spark Memory
上文已经到了，spark memory主要是spark自己使用的memory部分，这部分的大小依赖于spark.memory.fraction参数，即(Java Heap - 300M)*spark.memory.fraction，其中fraction的default为0.75。

Spark memory主要有两个用途，一是用于spark的shuffle等操作，而是用来cache spark中的RDD，因此spark memory也自然而然的分成了两部分，即负责shuffle操作的execution memory和负责cache的storage memory，两者的大小通过spark.memory.storageFraction参数来分割，默认值是0.5。

在spark memory中，还有一个重要的性质，那就是storage 和 execution memory的共享机制，说的简单一些就是，当一边内存空闲而另一方内存紧张的时候，可以借用对方的内存，我们下面看看在内存出现冲突的时候，spark怎么协调：
* 当storage占用execution memory的时候，发生execution memory使用紧张的情况时，强制将storage占有的内存释放并归还execution，丢失的数据将会后续重新计算；
* 当execution占用storage memory的时候，发生storage memory紧张的情况，被占用的内存不会被强制释放，因为这会带来任务丢失，storage会耐心等待知道execution执行完释放出内存。

## Data Serialization
在整个spark任务中，数据传输都是经过序列化后(serialization)之后传输的，因此数据的序列化是很重要的，冗余的序列化过程会让整个spark任务变慢，spark提供两种序列化方式：
* Java serialization：这是spark默认的序列化方式，java序列化是一种很经典和稳定的序列化方法，但是最大的缺点就是——慢！
* Kryo serialization：Kryo 序列化可以让spark任务更加快速，甚至10倍于java序列化；但是它不支持所有的Serializable类型，同时需要为用户自己开发的class进行注册后，才可以使用Kyo.

关于Kryo的详细信息，可以查看[spark documentation](https://spark.apache.org/docs/latest/tuning.html#data-serialization)，或者[Kryo documentation](https://github.com/EsotericSoftware/kryo)

## Summary
关于spark调优的问题，有很多因素，我也是简单的做了一些了解并分享给大家，除了我提到的，还有诸如GC等等因素，大家可以根据我给出的references做进一步的了解。

## References
* [How-to: Tune Your Apache Spark Jobs (Part 2)](http://blog.cloudera.com/blog/2015/03/how-to-tune-your-apache-spark-jobs-part-2/)
* [Spark Documentation-Tuning](https://spark.apache.org/docs/latest/tuning.html)
* [Karau, Holden, et al. Learning spark: lightning-fast big data analysis. " O'Reilly Media, Inc.", 2015.](http://shop.oreilly.com/product/0636920028512.do)