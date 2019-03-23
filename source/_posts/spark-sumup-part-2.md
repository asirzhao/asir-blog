---
title: Spark Tips Sum-up Part-2
date: 2018-10-13 09:53:02
tags: spark
categories: spark
toc: true
---
![](https://github.com/JoeAsir/blog-image/raw/master/blog/background/boulders-calm-conifers-426894.jpg)
This article presents some tips of Apache Spark, which is part 2 of the series. All the tips below are based on the real problems which I met. Despite the background, the tips below are of valuable reference. I've tried a lot to learn about Apache Spark but can't know the detail of every part of it. I'd appreciate it if you figure out the mistakes in this article.
<!--more-->
## Coalesce
Changing the number of partitions could benefit the performance of your Spark application. A large number of partitions may increase the parallelism of the process while too many tasks would be executed on a single executor, which could cost more running time. In contrast, a small number of partition might improve the complexity of each task and slow down the whole process. To solve this trade-off problem, *repartition()* and *coalesce()* is proposed in Apache Spark. 
Before talking about the detail about *coalesce()*, let's review the concept of transformation with wide-dependencies and narrow-dependencies.
> * Wide-dependencies: each partition of the parent RDD is used by at most one partition of the child RDD.
* Narrow-dependencies: multiple child partitions may depend on each partition in the parent RDD.

According to the definition above, *repartition()* is a typical transformation with wide-dependencies and thus can bring in shuffle operation when triggered. But what about *coalesce()*? To find out more about it, let's see the definition first.
```scala
// coalesce() for RDD is defined in org.apache.spark.rdd.RDD
def coalesce(numPartitions: Int, shuffle: Boolean = false,
    partitionCoalescer: Option[PartitionCoalescer] = Option.empty) 
    (implicit ord: Ordering[T] = null)
   : RDD[T] = withScope {...}

// coalesce() for Dataset is defined in org.apache.spark.sql.Dataset
def coalesce(numPartitions: Int): Dataset[T] = withTypedPlan {
    Repartition(numPartitions, shuffle = false, logicalPlan)
}
```
For RDDs, *coalesce()* has a boolean typed parameter called *shuffle*. The *coalesce()* can be treated as *repartition()* as *shuffle* is set to *True*, which is a transformation with wide-dependencies. In contrast, when *shuffle* is *False*, *coalesce()* is a transformation with narrow-dependencies and only can reduce the number of partitions, which means you cannot increase the number of partitions by setting the parameter *numPartitions*. As for DataFrame/Dataset API, *coalesce()* is a transformation with narrow-dependencies as there is no shuffle operation at all. As results, *coalesce()* cannot increase the number of DataFrame/Dataset's partitions and can only be used to reduce DataFrame/Dataset's partitions.
After understanding the above, there is a crucial tip for you. When you use *coalesce()* and reduce the number of partitions, which may cause a problem that the partition of the whole stage would be decreased and the computation could be even slower than you expect. since no shuffle operation performed, the stage executes with the level of parallelism assigned by *coalesce()*. To avoid this problem, you can set *shuffle=True* for RDDs or use *repartition()* instead for DataFrame/Dataset to split the whole stage by a shuffle.
## Read ORC Table
Reading data from and writing data to HIVE tables is quite often when I use Apache Spark. And we are now using ROC format to store HIVE table.
> ORC is a self-describing type-aware columnar file format designed for Hadoop workloads. It is optimized for large streaming reads, but with integrated support for finding required rows quickly. 

Generally, when Spark reads data from HDFS, the initial number of partitions are determined by the number of blocks of data store in HDFS, with one block represents one partition in Spark. However, when I read the ORC format HIVE table, the partition number is equal to the number of files stored on HDFS, not number block. That's caused by ORC split strategy set by *hive.exec.orc.split.strategy*, which determines what strategy ORC should use to create splits for execution. The available option includes "BI", "ETL" and "HYBRID"
> The HYBRID mode reads the footers for all files if there are fewer files than expected mapper count, switching over to generating 1 split per file if the average file sizes are smaller than the default HDFS block size. ETL strategy always reads the ORC footers before generating splits, while the BI strategy generates per-file splits fast without reading any data from HDFS.

As results, when the split strategy is set to ETL, Spark would take more time to generate tasks and the number of partitions is based on the size of HDFS blocks. In contrast, BI strategy would make Spark generate tasks immediately and the number of tasks is equal to the number of files of HIVE table. These two strategies seem to be a trade-off for us and it's better for us to decide by the actual situation.  

## Reference
* [Managing Spark Partitions with Coalesce and Repartition](https://hackernoon.com/managing-spark-partitions-with-coalesce-and-repartition-4050c57ad5c4)
* [High Performence Spark](http://opencarts.org/sachlaptrinh/pdf/28044.pdf)
* [Apache ORC](https://orc.apache.org/)
* [Apache ORC Configuration Properties](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.exec.orc.split.strategy)
