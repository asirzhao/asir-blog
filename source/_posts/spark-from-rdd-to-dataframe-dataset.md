---
title: From Spark RDD to DataFrame/Dataset
date: 2018-09-22 16:49:15
tags: spark
categories: spark
---
![](http://otmy7guvn.bkt.clouddn.com/blog/background/blur-close-up-code.jpg)
This article presents the relationship between Spark RDD, DataFrame and Dataset, and talks about both the advantages and disadvantages of them. RDD is the fundamental API since the inception of Spark and DataFrame/Dataset API is also pretty popular since Spark 2.0. What's the differences between them and how to decide which API to be imported, let's have a quick look.
<!--more-->
## RDD
RDD (aka Resilient Distributed Dataset) is the most fundamental API, it's so critical that all the computation in Spark are based on it. The RDD is distributed, immutable, type-safed, unstructured, partitioned and low-leveled API, which offers transformations and actions. Let's have a quick review to the attributes of RDD. RDD has some awesome characteristics which make it the foundation of the Apache Spark. Let's take a look and learn about the details one by one.
### Distributed data abstraction
The first attribute of RDD is the logical distributed abstraction, which makes RDD can run over the entire clusters. The RDD can be set across the storage and divided into many partitions so that the lambda function or any computation function you provide will execute on each partition separately. That's really awesome as RDD is the logical distributed abstraction, the computation will be much faster as data is divided and run in parallel on several executors.
### Resilient and immutable
RDD is also resilient and immutable. When an RDD is transformed to the next RDD, and then to the third one, and all the RDDs get recorded as a lineage of where they come from. The lineage can also be recorded as the acyclic graph of the RDDs, from which we can recreate any RDD in it when something goes wrong, and that's why RDD is resilient.
As for the immutability, when you make a transformation, the original RDD remains unaltered so that you can go back through the acyclic graph and recreate it at any time and point during the execution. Remember, the transformation operation creates a new RDD from the previous one instead of altering the previous one. 
### Compile-time type-safe
RDD is compiled type-safe and you can name the RDD with particular type briefly. Spark application could be complicated and debugging in the distributed environment could be cumbersome, the compiled type-safe really save time as it could find the type error in the compile time.
### Unstructured/Structured data
The fourth attribute is that data can be unstructured, streaming data from the media for instance, or semi-structured, log files with some particular date, time or url information for example. Since RDD would not care about the structure or schema of the data, it's good for those data without structures. Also, RDD can manipulate structured data, though it doesn't understand the different kinds of types and all depends on how you parse the data.
### Lazy evaluation
Lazy evaluation in Apache Spark means the execution will not start until an action is triggered. In another word, the RDD will not be loaded and computed until it is necessary. And there are some benefits of lazy evaluation. Lazy evaluation really reduces the time and space complexities and speeds up the whole execution. 
## DataFrame/Dataset
DataFrame/Dataset, unlike RDD, in high-level API dealing with structured data. Data is organized into named columns in DataFrame and can be manipulated type-safely. In Scala, DataFrame is just an alias for *Dataset[Row]*, and in Java, there is only Dataset API, as for Python and R, there is only DataFrame API provided since Python and R have no compile-time type-safety.
There are several benefits of DataFrame/Dataset API. I just want to talk about two of them, which I think are pretty awesome.
### Static-typing and runtime type-safety
DataFrame/Dataset presents static-typing and runtime type-safety. You may have a spelling error when you are typing a SQL such as typing *form* rather than *from*, and you would not find the syntax errors until the runtime, however, you will catch these errors at compile time in the DataFrame/Dataset API. Also, as for some analysis errors, the column you queried is not in the schema for example, you can catch these errors when compiling in Dataset while until running in SQL and DataFrame.
![](http://otmy7guvn.bkt.clouddn.com/blog/17/17-1.png)
### Nice performance
DataFrame/Dataset API can make the execution more intelligent and efficient. You are telling Spark how-to-do a operation when using RDD, while what-to-do using DataFrame/Dataset. Let's have a look at the example.
```scala
rdd.filter{case(project, page, numRequests) => project=='en'}.
    map{case(_,page,numRequests) => (page, numRequests)}.
    reduceByKey(_+_).
    filter{case(page,_) => !isSpecialPage(page)}.
    take(100).foreach {case (project, requests) => println(s"projec:$requests"")}
```
The code above can be run perfectly without any bug. But think about it, the RDD execute a *filter* followed by *reduceByKey* transformation, which means we filter some data after shuffling the entire data. That's really a waste because shuffle operation is very expensive. However, this problem will be solved in DataFrame/Dataset. Based on the Catalyst, DataFrame/Dataset will optimize the query operation by rules and cost, thus the execution is much smarter than the raw RDD.
## When to Use
Since we take a look at the RDD and DataFrame/Dataset, I make a list about when to use RDD and when to use DataFrame/Dataset.
### When to use RDD
* When you want more about the low-level control of dataset
* When you are dealing with some unstructred data
* When you prefer manipulate data with lambda function
* When you don't care about schema or structure of data

### When to use DataFrame/Dataset
* When you are dealing with structured data
* When you want more code optimization and better performance

All in all, I do recommend you to use DataFrame/Dataset API as you can for their easy-using and better optimization. Supporting by Catalyst and Tungsten, DataFrame/Dataset can reduce your time of optimization, thus you can pay more attention to the data itself. 

## References
* [A Tale of Three Apache Spark APIs: RDDs, DataFrames, and Datasets](https://databricks.com/blog/2016/07/14/a-tale-of-three-apache-spark-apis-rdds-dataframes-and-datasets.html)
* [Apache Spark RDD vs DataFrame vs DataSet](https://data-flair.training/blogs/apache-spark-rdd-vs-dataframe-vs-dataset/)
