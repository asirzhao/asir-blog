---
title: Spark Tips Sum-Up Part-1
date: 2018-09-15 22:15:40
tags: spark
categories: spark
---
![](http://otmy7guvn.bkt.clouddn.com/blog/background/blurred-background-close-up-coffee-cup.jpg)
This article is about things I learned about Apache Spark recently. I've been struggling with Spark tuning for more than one month, including shuffle tuning, GC time tuning and so on. Honestly speaking, Apache Spark is just like an untamed horse, it could be a beauty if you tune it well while a nightmare if not. So let me show you some tips I've learned from my work. This article is part 1 and here we go.
<!--more-->
## RDD vs DataFrame Partition Number in Shuffle 
Shuffle plays a pretty crucial role in MapReduce and Spark, which can influence a lot on the performance of the whole job. The shuffle would not only separate one stage but bring much more storage and I/O delay. However, is usually hard to avoid shuffle occurring, so how to make shuffle faster could be the key point.
During my work in tuning the Spark SQL project, the join operation is unavoidable because it is the fundamental process in all the SQL program. While when I try to run the Spark SQL saving the joined DataFrame into a Hive table, I found the number of table partition is constant no matter how many partitions of two joining DataFrame, oh-gee! it would never happen in RDD join process. That makes the size of each file of Hive table on the HDFS very large, and I have to fix it up.
After searching from the Internet, I found that the shuffle of DataFrame in Spark, join included, would have a partition number according to the configuration *spark.sql.shuffle.partitions* with 200 as default. And the problem above can be solved as this configuration changed. So, when you join two DataFrames or other process causing shuffle, remember the configuration *spark.sql.shuffle.partitions* when you want to modify the DataFrame partition number. 
What if I want to modify the partition number of a RDD? Actually, the configuration *spark.default.parallelism*, which seems to only working for raw RDD and is ignored when working with DataFrames in Spark SQL, may be a good choice. Alternatively, you can set the partition number by calling *repartition( )* or *coalese( )*, which is effective for both DataFrames and RDDs.

## Smart Action
As we know, RDDs support two types of operation, which are action and transformation. Transformation creates a new RDD from an existing one, and action returns a value to the Spark driver from computing on a RDD. Based on the lazy evaluation, all the transformations would run as a lineage when an action is triggered by the Spark. A lineage of transformation followed by an action consist of one job in Spark, and one wide-dependency between two transformations separate the job into two stages, which is aka shuffle.
When I'm tuning a Spark project, I found that different action following the same lineage of transformation takes different period of time. For example, the action *show( )* takes shorter time than *createOrReplaceTempView( )*, which makes me confuse a lot. After a long time thinking and searching, the answer finally comes out. Spark would draw a DAGSchedule when the program is submitted, the data would run through all the DAGSchedual and the result is sent to the Spark driver. Different action may generate different DAGSchedule even the the transformations are same, Spark is smart enough to know whether it needs to run everything in the RDD. Showed above, Spark may run less data for the *show( )* action than those for *createOrReplaceTempView( )*.

## Broadcast Joins
Broadcast joins (aka map-side joins) is a good way to abort shuffle and reduce cost. Spark SQL provides two ways for developers, you can not only write SQL but also use DataFrame/Dataset API. When a large table joins a smaller one, a threshold defined by *spark.sql.autoBroadcastJoinThreshold*, with 10M as default value, determines whether the smaller one will be broadcast or not. If you use SQL in Spark SQL, as the smaller table size below the threshold, Spark would automatically broadcast it to all executors. However, if you use DataFrame/Dataset API, the *broadcast* function must be imported or Spark wouldn't broadcast data even if the size is below the threshold.
```scala
val df = largeDF.join(broadcast(smallDF),Seq("col1","col2"),"left")
```
Also, you can enlarge the value of *spark.sql.autoBroadcastJoinThreshold* so that larger table can also be broadcast, but the memory of your application should be paid attention. 
Broadcast joins is really an awesome solution to optimize Spark SQL joins, after using broadcast joins instead of default SortMerge joins, my application runs more than 10 times faster. Avoiding shuffle is quite important for Spark tuning, and broadcast is born to kill shuffle. You will fall in love with her as long as you have a try!

## References
* [Spark SQL Programming guide](http://spark.apache.org/docs/latest/sql-programming-guide.html#other-configuration-options)
* [Mastering Spark SQL](https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-joins-broadcast.html)
