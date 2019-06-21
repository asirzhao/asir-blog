---
title: ORC Stored HIVE Tips 
date: 2019-06-22 18:05:12
tags:
    - hdfs
    - hive
categories: hadoop
toc: ture
---
![](https://github.com/JoeAsir/blog-image/raw/master/blog/background/balloon-bright-celebration-2388650.jpg)
ORC stored HIVE table has been very common in my daily work. Having a deep dive in ORC format recently, I realize that there are many awesome features in ORC format, and many of them have never been used before or somehow in the wrong way by me. So I take some time with some little tests and present two of the most exciting features of ORC stored HVE table in this blog. Let's start this! 
BTW, if you are not familiar with ORC, you can take a quick view from my [previous blog](https://joeasir.github.io/2019/05/05/hdfs-arvo-parquet-orc/#ORC).
<!--more-->
## Index Filter
ORC is a kind of columnar format, the basic structure and some terms are introduced in my [previous blog](https://joeasir.github.io/2019/05/05/hdfs-arvo-parquet-orc/#ORC). There are three kinds of the index in ORC file, file level, stripe level, and row level, all of which stores some statistics values such as max/min, sum value, and so on. If you want to learn the specific content of these statistics, you can use `hive --orcfiledump {path-of-your-orc-file}` to lean everything, such as schema, compression, stripe statistics, file statistics, you name it, about your ORC file.
When the ORC file is been queried with a filter execution *WHERE Col1 > 10*, based on the statistics information like max and min value of *Col1*, the stripes whose max value of *Col1* could be skipped and as the result, the input records and mappers could be reduced.
However, there is a little problem, if you want to take advantage of the index filter, you have to set *hive.optimize.index.filter=true*, but the good news is that this configuration is ignored in the HIVE 3.0.
What's more, suppose the min/max value of the filtered column in each stripe is not overlapping, the index filter could be much more efficient, isn't it? So there is a tip for you, if the column is the major filter, it could be better to sort it as soon as it's inserted in the ORC table. I made a simple test and the status of non-sorted and sorted table present different performance under the same following query HIVEQL. 
```sql
SELECT
    SUM(Col1),
    COUNT(1)
FROM table
WHERE Col > 1000
```
There are significantly fewer input records and mappers when querying the sorted table, which is pretty awesome! 
![](https://github.com/JoeAsir/blog-image/raw/master/blog/24/24-1.png)
![](https://github.com/JoeAsir/blog-image/raw/master/blog/24/24-2.png)

Last but not the least, *ORDER BY* in HIVEQL is such an expensive execution that all the records are put into a single reducer if you have a huge data, it's also a good idea to use *DISTRIBUTE BY* to instead.
    
## Vectorization
Vectorization, aka as Vectorized Query Execution, is an awesome feature for ORC stored HIVE table. It reduces the CPU usage by processing a block of 1024 rows instead of row by row. You can find detail information [here](https://cwiki.apache.org/confluence/display/Hive/Vectorized+Query+Execution)
To make leverage of Vectorization, we must set *hive.vectorized.execution.enabled=true*. I also make a comparison between the vectorized and standard execution. As the figures below, processing 1014 rows each time, the amount of input records is 1000 times less than the standard execution. However, I found hardly ever difference about the CPU time spent, maybe the execution of my SQL is too simple to distinguish the CPU status between vectorized and standard execution.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/24/24-2.png)
![](https://github.com/JoeAsir/blog-image/raw/master/blog/24/24-3.png)

The vectorization in ORC is of good efficiency to reduce the input records in HIVE, and this feature has now been supported in Apache Spark since 2.3, see more [here](https://spark.apache.org/docs/latest/sql-data-sources-orc.html).
    
## References
* [Vectorized Query Execution](https://cwiki.apache.org/confluence/display/Hive/Vectorized+Query+Execution)
* [5 Ways to Make Your Hive Queries Run Faster](https://hortonworks.com/blog/5-ways-make-hive-queries-run-faster)
