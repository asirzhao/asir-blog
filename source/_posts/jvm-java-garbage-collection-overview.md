---
title: Java Garbage Collection Overview
date: 2019-04-26 16:35:08
tags: jvm
categories: jvm
toc: true
---
![](https://github.com/JoeAsir/blog-image/raw/master/blog/background/beach-boats-clouds.jpg)
Java Garbage Collection has confused me for such a long time when I try to tune my Spark Application, but unfortunately I'm not a good Java developer. I really feel terrible when staring at the red blocks representing high GC time in my SparkUI while having no idea how to fix it up. So I spent some time digging in GC and finally got to learn about what GC is and how to analysis the GC logs. So today I'm sharing you something I learnt and let's move on.
<!--more-->
Just as name presenting, Java Garbage Collection is used to collect the garbage, which is actually the unused objects, in JVM Heap memory. If you don't know what does Heap memory represents, you'd better make sense of JVM memory management first.
## GC overview
Let's firstly take a quick look at on how GC track and remove the so-called grabage. The whole porcess, consisting of two steps, marking and deletion, might be much simpler than you can imagine.
* Marking - It's easy to understand what **Marking** does if you know what GC does. Yep! **Marking** is purposed to distingrish and mark down which objects are still referenced and which are to be collected. And it's a time consuming process for all the objects all scanned.
* Deletion - **Deleteing** removes the unreferenced objects by moving the referenced objects together, it compact the memory to make benefis to the further memory allocation.

## GC Process
### JVM Generations
The Heap memory in JVM is devided into manly three parts, which are Eden, Survivor and Tenured Space(and Permanet space in formal JDK version). Eden and Survivor spaces are called Yong Generation while the Tenured space Old Generation. I have to say the names of JVM Generations are really vivid, the Young Generation is where all the new objects allocated, and Old Generation is used to store long-living objects.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/20/20-1.png)
### Young Generation
There are two parts of Heap spaces in Young Generation, which are Eden and Survivor spaces as we talked above. And the Survivor space is actually made up with two parts, Survivor 0 and Survivor 1, aka "from" Survivor and "to" Survivor, or s0 and s1. The key point is that, those two Survivors space are not immutable as Eden space, actually they are relative just as the name "from" and "to" shows. Once an object is first allocated, it wil be stored in the Eden space. Time goes by, the Eden space is gradually filled up with so many new-born objects, then the **Minor Garbage Collection(Minor GC)** is triggered. Hey, you see? the **Minor GC** finally comes out here and it's actually the GC process in the Young Generation. 
In the **Minor GC** process, all the unreferenced or unused objects are removed, and all the referenced ones are moved to one of the Survivor spaces, which is actually the "from" Survivor space. As new objects are continuously born(or allocated), with the Eden space gradually filled up with those objects again, the **Minor GC** is triggered another time, all the unreferenced objects in the Eden space and "from" Survivor space are all removed while the referenced ones are moved to the "to" Survivor space. That's why the two Survivor spaces are called "from" and "to", really vivid, isn't it? Hahaha. The survived objects(stored in Survivor space) are moved between these two Survivors space gathering with new-born ones moved from the Eden It's interesting to find out that the "from" Survivor space will be called as "to" Survivor space in the next data moving time, which always keep the survived objects are all stored in one Survivor, whil another empty.
Once the referenced objects are moved, they are also aged once. As soon as the age of the objects reach a certain threshold(15 as the default value, and you can custom it as you wish), they would be moved to a new world, that's the Old Generation(this moving stage is aka so-called promotion), which I'm about to talk below.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/20/20-2.png)
### Old Generation
As we talked above, when referenced objects are "old" enough(couldn't stop laughing), or one of the Survivor space is full-filled, they will be then promoted to the the Tenured space, which means the long-time survived objects will be stored in the Old Generation. As **Minor GC** keeps invoked once again and again, survived objects will be continuous promoted to the Old Generation. And the **Major Garbage Collection(Major GC)** would be triggered eventually on the Old Generation to clean up and compact that space.
What's more, what if the Old Generation is full? Wow, that's really not a good news for your JVM application because the **Full Garbage Collection(Full GC)** is triggered, which clean up all the objects on **both** the Young Generation and Old Generation.
I have to say that there is no brief definitions for **Major GC** and **Full GC**, which really confuses me, see more details in [this blog](https://plumbr.io/blog/garbage-collection/minor-gc-vs-major-gc-vs-full-gc).
### Stop-the-wordld(STW)
Now we almostly get the generatinal garbage collection process of JVM GC, but why we foucs a lot on it, and how does it ruins our application? The answer is quite simple, that is **Stop-the-world** aka STW. Once a GC process triggered, the JVM has to move and update those references object before the application manipulate them, otherwise there could be something wrong. [See this answers on Stackoverflow to learn more](https://stackoverflow.com/questions/40182392/does-java-garbage-collect-always-has-to-stop-the-world). 
Actually, all the GC process could bring in STW problem, and especially for **Full GC** process, which could be a nightmare your application proformance. 

## Java Garbage Collector
* Serial Garbage Collector
As name presents, Serial Garbage Collector is basicaly works with a single thread. Serial Garbage Collector would stop all the other application threads because of the memory compaction, also known as the **STW** we talked about.  As a result, Serial Garbage often be used in the applications without low pause time requirements and run on client-style machines.

* Parallel Garbage Collector
Instead of the single thread in Serial Garbage Collector, Parallel Garbage Collector uses multiple threads to preform GC. And Parallel Collector could also bring the **STW** as the Serial Garbage Collector does. 

* CMS Garbage Collector
The Concurrent Mark Sweep Garbage Collector, aka CMS, uses multiple garbage collector threads to reduce the pauses time in application when collecting Old Generation. Normally the CMS Garbage Collector dosen't compact the memory and move the referenced objects together after collection, thus that's the mainly disadvtange of CMS.

* G1 Garbage Collector
Garbage First or G1 Collector is purposed to be the alternative of CMS Garbage Collector. G1 Collector seems to be a perfect choice as it's a parallel, concurrent and compacting low-pause collector.

> Unlike other collectors, G1 collector partitions the heap into a set of equal-sized heap regions, each a contiguous range of virtual memory. When performing garbage collections, G1 shows a concurrent global marking phase (i.e. phase 1 known as Marking) to determine the liveness of objects throughout the heap.
After the mark phase is completed, G1 knows which regions are mostly empty. It collects in these areas first, which usually yields a significant amount of free space (i.e. phase 2 known as Sweeping). It is why this method of garbage collection is called Garbage-First.

## Conclusion
This blog mainly talks about the overview of the JVM Garbage Collection from the generation based process to garbage collector. Honestly speaking, GC is much more complex than presented in the blog, thus there is still long way to go to make a deep dive in GC, even JVM. 

## References
* [Basics of Java Garbage Collection](https://codeahoy.com/2017/08/06/basics-of-java-garbage-collection/)
* [JVM Garbage Collectors](https://www.baeldung.com/jvm-garbage-collectors)
* [Java Garbage Collection Basics](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)
* [Minor GC vs Major GC vs Full GC](https://plumbr.io/blog/garbage-collection/minor-gc-vs-major-gc-vs-full-gc)
