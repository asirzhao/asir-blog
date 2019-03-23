---
title: Second Generation Tungsten Engine in Spark 2.x
date: 2018-11-14 15:41:15
tags: spark
categories: spark
toc: true
---
![](https://github.com/JoeAsir/blog-image/raw/master/blog/background/adventure-climb-691668.jpg)
This article is about the 2nd generation Tungsten engine, which is the core project to optimize Spark performance. Compared with the 1st generation Tungsten engine, the 2nd one mainly focuses on optimizing query plan and speeding up query execution, which is a pretty aggressive goal to get orders of magnitude faster performance. Let's take a look!
<!--more-->
## Project Tungsten
In the past several years, both storage and network IO bandwidth have been largely improved, while CPU efficiency bound has not. As results, CPU now becomes the new bottleneck and we have to substantially try to improve the efficiency of memory and CPU and push the performance of Spark closer to the limits of modern hardware, which is the main propose of Tungsten. To achieve it, three initiatives in 1st generation Tungsten engine are proposed, including:
> * Memory Management and Binary Processing: leveraging application semantics to manage memory explicitly and eliminate the overhead of JVM object model and garbage collection
* Cache-aware computation: algorithms and data structures to exploit memory hierarchy
* Code generation: using code generation to exploit modern compilers and CPUs

As we can see, the first two initiatives mainly foucus on memory, and the last one are for CPUs. In the 2nd generation Tungsten engine, rather than code generation, WholeStageCodeGen and Vectorization are proposed for the order of magnitude faster.
## WholeStageCodeGen
As the name presents, WholeStageCodeGen, aka whole-stage-code-generation, collapses the entire query into a single stage, or maybe a single function. So why combining all the query into a single stage could significantly improve the CPU efficiency and gain performance? We may take a quick look at what it looks like in 1st Tungsten engine.
### Volcano Iterator Model
What a vividly name! Volcano iterator model, as presents in the figure, would generate an interface for each operator, and the each operator would get the results from its parent one by one, just like the volcano eruption from bottom to top. 
![](https://github.com/JoeAsir/blog-image/raw/master/blog/19/19-1.png)
Although Volcano Model can combine arbitrary operators together without worry about the data type of each operator provides, which makes it a popular classic query evaluation strategy in the past 20 years, there are still many downsides and we will take about it later. 
### Bottom-up Model
In [this blog](https://databricks.com/blog/2016/05/23/apache-spark-as-a-compiler-joining-a-billion-rows-per-second-on-a-laptop.html), a hand-written code is proposed to implement the query in the figure above, it's just a so simple for-loop that even a college freshman can complete, which is:
```scala
var count = 0
for (ss_item_sk in store_sales) {
  if (ss_item_sk == 1000) {
      count += 1
  }
}        
```
Even though the code is pretty simple, the comparison of performance  between Volcano Iterator Model and Bottom-up Model will do shake you. 
![](https://github.com/JoeAsir/blog-image/raw/master/blog/19/19-2.png)
But why is that? Seems it turn out it to be caused by following downsides of Volcano Iterator Model:
* Too many virtual functions calls:
In Volcano Iterator Model, when one operator call for the next operator, a virtual function **next()** would be called. But in Bottom-up Model, there is no virtual function called because all the operations are combined in a single loop function.
* Intermediate data of Volcano Iterator Model are in memory while of Bottom-up Model are in CPU registers:
As one operator transforms data to another one in Volcano Iterator Model, the intermediate data can only be cached in Memory. But for Bottom-up Model, as there is no need to be transformed, data are always in CPU registers.
* Volcano Iterator Model don't take advantage of modern techniques, which Bottom-up Model do, such as loop-pipelining, loop-unrolling and so on:
As Bottom-up Model evaluates the whole query into a single loop function, some modern technique can be used in the Bottom-up Model, I will show you two of them, which are loop-pipelining and loop-unrolling.

#### Loop-pipelining 
In a loop iteration function, one iteration of loop usually begins when the previous one is complete, which means the iteration of the loop should be executed sequentially one by one. While loop-pipelining can make some differences. The figure below could learn about it clearly. Loop-pipelining increases the parallelism of the loop iteration by implementing a concurrent manner.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/19/19-3.png)
#### Loop-unrolling
Loop-unrolling is another technique to exploit parallelism between loop iterations. Let's learn about it by the code:
```java
// without loop-unrolling
int sum=0;
for (int i=0; i<10; i++) {
    sum+=a[i];
}
// with loop-unrolling
int sum = 0;
for (int i=0; i<10; i+=2) {
    sum += a[i];
    sum += a[i+1];
}
```
As shown above, loop-unrolling creates multiple copies of the loop body and also changes the loop iteration counter. Reducing loop iteration number, loop-unrolling also create more operation in each loop iteration and, as results, more parallisim could be exploited. 

### Whole Stage Code Generation
Fusing operators together to make the generated code looks like the hand-writing bottom-up model, WholeStageCodeGen makes chains of operators as a single stage, and it has been the alternatives for the Code Generation in Catalyst Optimization. It takes advantages of hand-writing and significantly optimizes the query evaluation and can be easily found in the DAG of your Spark application.
## Vectorization
Although the WholeStageCodeGen makes a huge optimization of the query plan, there are still some problems. For example, when we import some external integrations, such as tensorflow, scikit-learn, and some python packages, these code cannot be optimized by the WholeStageCodeGen cause they cannot be merged in our code. What's more, the complicated IO cannot be fused, reading Parquet or ORC for instance. How to speed up this excution? The answer is improving the in-core parallelism for an operation of data, so **Vector Processing** and **Column Format** are used in 2nd generation Tungsten engine.
### Vector Processing
> In computing, a vector processor or array processor is a central processing unit (CPU) that implements an instruction set containing instructions that operate on one-dimensional arrays of data called vectors, compared to scalar processors, whose instructions operate on single data items.

The following figure presents the differences between Scalar and Vector Processing. 
![](https://github.com/JoeAsir/blog-image/raw/master/blog/19/19-4.png)

And we find out that Vector Processing really fasten the operation. How does Vector Processing be implemented? We may need to learn something about SIMD(Single Instruction, Multiple Data)
> Single instruction, multiple data (SIMD) is a class of parallel computers in Flynn's taxonomy. It describes computers with multiple processing elements that perform the same operation on multiple data points simultaneously.

Let me show one figure to show what's SIMD breifly.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/19/19-5.png)
As presented above, SIMD can process multuple data via single instruction, and the data are all in an one-dimesional vector. As results, take advantage of SMID, Vector Processing can improve the in-core parallelism and thus make the operation faster. For the sake of the data are all in an one-dimesonal vector in SIMD, Colunm Format could be a better choice for Spark.

### Column Format
Column Format has been widely used in many fields, such as disk storage. The following figure shows the differences between Row Format and Column Format.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/19/19-6.png)

## Summary
WholeStageCodeGen and Vectorization in 2nd generation Tungsten engine really optimize the query plan and speed up the query execution. Compared with 1st generation Tungsten engine, the 2nd one mainly foucses on improving the CPU parallelism to take advtange of some modern techniques. There are many terms and konwledges about CPU exection, which I learn so little that may make some mistakes in this blog, so it would be very nice of you to figure out any single mistake.
## References
* [Project Tungsten: Bringing Apache Spark Closer to Bare Metal](https://databricks.com/blog/2015/04/28/project-tungsten-bringing-spark-closer-to-bare-metal.html)
* [Apache Spark as a Compiler: Joining a Billion Rows per Second on a Laptop
Deep dive into the new Tungsten execution engine](https://databricks.com/blog/2016/05/23/apache-spark-as-a-compiler-joining-a-billion-rows-per-second-on-a-laptop.html)
* [Spark 2.x - 2nd generation Tungsten Engine](https://spoddutur.github.io/spark-notes/second_generation_tungsten_engine.html)
* [Loop Pipelining and Loop Unrolling](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2015_2/sdsoc_doc/topics/calling-coding-guidelines/concept_pipelining_loop_unrolling.html#)
* [Vectorization: Ranger to Stampede Transition](http://www.cac.cornell.edu/education/training/ParallelFall2012/Vectorization.pdf)
