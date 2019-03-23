---
title: Spark Tips Sum-up Part-3
date: 2019-03-06 18:24:15
tags: spark
categories: spark
toc: true
---
![](https://github.com/JoeAsir/blog-image/raw/master/blog/background/beach-dawn-dusk-185927.jpg)
This blog is the third part of Apache Spark tips sum-up learnt from my programing and debugging. Have been busy for such a long time, I have some time to carry on my personal blog. Today I'll show you some tips about some functions in Spark SQL, and there may be some mistakes caused by my misunderstanding. Anyway, thanks if you figure out anything incorrect!  
<!--more-->
## Explode
Spark SQL provides a varority of functions in *org.apache.spark.sql.functions* for you to restruct your data, one of which is the *explode()* function. Since Spark 2.3, *explode()* function has been optimized and it has been much more faster than it in the previous Spark versions. For more details about this optimization, [this issue](https://issues.apache.org/jira/browse/SPARK-21657) may help you a lot.
However, what I want to share about is the number of partitions when you use *explode()*. It's easy to understand that the number of rows grows several times after the exploding process, as a result, the size of data for each partition has become larger than ever, which may cause the jobs taking more time and more memory. So, it could be a wise choice to enlarge the number of partitions by *repartition()*, especially when the rows explode more than 10 times than previous, each task would process much more data and that's possible to get an OOM error, or high GC time.

## Foreach vs ForeachPartition, Map vs MapPartition
Yeah, *foreach()* vs *foreachPartition()* and *map()* vs *mapPartition()*, these four method do confuse me for a long time and let me share you my understanding about them. 
First of all, *foreach()* and *foreachPartition()* are actiona in Spark, while *map()* and *mapPartition()* are transformations. If you have no ideas about the defination of action and transformation, it's better to read about my previous blog or just ask help for dear *google*. *foreach()* and *foreachPartition()* are often used for writing data to external database while *map()* and *mapPartition()* are used to modify the data of each row in the RDD, also DataFrame or DataSet.
Seondly, *foreachPartition()* and *mapPartitionn()* are respectively based on *foreach()* and *map()*. Instead of invoking function for each element, *foreachPartition()* and *mapPartition()* calls for each partition and provide an iterator to invoke the function. So what's the advantages of invoking function for each partition? For example, when you invoke a function connecting to your database, redis for example, *foreach()* will build a connection for each row of your data, which means, number of connection could be pretty huge and you may get connection failed errors. Instead of invokeing for each element, *foreachPartition()* could invoke the function building connection to redis for each partition, and data in each partition would be written to redis iteratively. Amazing, isn't it!

## Reading ORC Table
Actually speaking, I have already talked about reading data from ORC table in Apache Spark in the previous blog, but I find something about reading from ORC table pretty awesome since Spark 2.3. Let's have a look. Spark supports a vectorized ORC reader with a new ORC file format for ORC files since 2.3 version, which means reading from and writing to ORC fromat file could be much faster!
To enbale the vectotized ORC reader, you just need to set these configuration:
* --conf spark.sql.orc.impl=native 
* --conf spark.sql.orc.enableVectorizedReader=true 
* --conf spark.sql.hive.convertMetastoreOrc=true

For more information, you can read the [Spark Doc](https://spark.apache.org/docs/latest/sql-data-sources-orc.html).

## References
* [Apache Spark - foreach Vs foreachPartitions When to use What?](https://stackoverflow.com/questions/30484701/apache-spark-foreach-vs-foreachpartitions-when-to-use-what)
* [Spark SQL Guide - ORC File](https://spark.apache.org/docs/latest/sql-data-sources-orc.html)
