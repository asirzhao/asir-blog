---
title: Spark Tuning
date: 2018-01-26 17:10:32
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

### On Yarn
假设我们现在设置executor-cores为5，那么在spark on yarn中，spark就会向yarn申请5个虚拟的core；但是，对于executor-memory来说要稍稍复杂一些，我们一起来看看：


## Memory Mangement

## Data Serialization

## Data Structures

## Other method
### Parallemism

### Broadcasting

