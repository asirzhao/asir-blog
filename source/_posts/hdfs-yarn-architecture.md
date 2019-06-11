---
title: YARN Architecture
date: 2019-06-11 20:18:54
tags:
    - hdfs
    - yarn
categories: hadoop
toc: true
---
![](https://github.com/JoeAsir/blog-image/raw/master/blog/background/aerial-shot-bird-s-eye-view-forest-2415927.jpg)
YARN, the abbreviation of *Yet Another Resource Negotiator*, is introduced in Hadoop 2.0. Compared with MRV1(MapReduce Version 1), YARN takes over the responsibility of **resource management and job scheduling** in MRV1, and make **non-MapReduce** jobs run on the Hadoop, Apache Spark for example. Although there are some rising technology that been treated as alternative of YARN by more and more developers, Kubernetes for instance, YARN is still widely used. Let's talk about YARN today. 
<!--more-->
## Components
YARN consists of one Resource Manager as master deamon, several slave daemon called Node Manager for each slave node and Application Master for each application.
### Resource Master(RM)
Resouce Manager(RM) is replacement of **JobTracker** in MapReduce Version 1 and it is the master deamon in YARN and manages the global resources among all the applications. Resource Master has two main components, Scheduler and Application Manager
#### Scheduler
Scheduler is mainly desgined to negotiate resource and it is responsible for allocating the resource for the running applications based on the abstract notion of a resource Container, such as memory, CPU etc. Notice that Container is the basic unit of resource in YARN. Scheduler allocate the resource based on the resource requirment of the application and the resource availabality. Scheduler doesn't care too much about the application monitoring or tracking, even the restarting failure no matter application or hardware failure.
#### Applicaiton Manager
Application Manager is responsible for the maintaining a collection of submitted applications. Application Manager accept the submission of one application from the client and allocate one Container for the application to load its Application Master. What's more, Application Manager provides the monitoring and tracking service for the running applications and also restarting them if any failure come out.
### Node Manager(NM)
Node Manager, the update version of **TaskTrack** in MRV1, is the slave deamon in YARN. Node Manager is mainly responsible for the individual Containers management and monitoring. Node Manager tracks the usage of resource among all the Containers in the node and replay the detailed information to Resource Manager and Node Manager also monitors the heath of the running Containers(Node Manager only manage the resource and running status rather than application context information).
Node Manager always keep in touch with Resouce Manager with heartbeats, 
### Application Master(AM)
One Application Mater runs for one application in YARN, which means each application has a unique Applicaiton Master. Application Master is responsible to send request to Resouce Manager and acquire resource from the Scheduler. What's more, Application Master is the process that coordinates the application's execution and manage the applicaiton faults along with the Node Manager. Also, Application Master sends heartbeats to the Resource Manager periodly to update the health condiction and resource demands.
Notice that there is a main difference between the NM and AM, NM only monitors and tracks the status of the dividul Containers while AM the application itself. When an application is running, both NM and AM contributes to the monitoring and tracking, but they focus on the different part.

## Applicaiton Submission on YARN
As we've learned each component of the YARN, let's take look at how these components work together to submit an application. The workflow is presented below.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/23/23-1.png)

* Client submits an application to the Resource Manager and request for building an Application Master;
* Rsource Manager, Application Manager actually, allocates a free Container to load the Application Master for the submitted application and then the Application Master then will start;
* Application Master registers to the Resource Manager, after which the client can communicate with the Application Master directly;
* Application Master send request to acquire resource Container for the application to Resouce Manager;
* Resource Manager(Scheduler) notify the Node Manager to launch Containers, and the launched Containers can communicate with Application Master;
* Application code will executed in the Container, and the status can be updated to the Application Master by the *application-specific*;
* While the application is running, client contacts to the Resource Manager(Application Manager) to get the application's running status, health condiction and so on;
* As soon as the application finishes, Application Master unregisters to the Resource Manager and shut down the connection, and all the Containers will also be free.

## Scheduling in YARN
Scheduling is one of the most important points in YARN. As we talked about in the Resouce Manager, the Scheduler in RM is responsible for scheduling, and actully there are mainly three kinds of scheduler in YARN, which are **FIFO**, **capacity** and **fair** scheduler. The figure below presents the differences among them.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/23/23-2.png)

### FIFO Scheduler
As name presents, FIFO, abbr of *firt in first out*, is the most basic scheduler, the submitted applications are all in a queue and executed one by one in order. The drawback is pretty significant, as shown in the figure, the whloe queue could be easily blocked by the **BIG** applications. Big application always need more time to be executed and the whole queue have to be waiting for the big application execution.

### Capacity Scheduler
Capacity schdeuler can solve the blocked queue problem in FIFO in some terms. Capacity Scheduler devide the source into multiple queues and some of them are for big applications specifically. Big applications in Capacity take less resource than in the FIFO, which makes the big application take longer time to execute in Capacity Scheduler.

### Fair Scheduler
Hence neither FIFO nor Capacity are good enough for Scheduling, the Fair Scheduler are proposed and it's the most popular scheduler in YARN at present. Similar to the Capacity Scheduler, Fari also divide the cluster into multiple queues. However, the capacity, or the resource, of each queues is dynamical. Also the fraction of the queue source deviding can be modified in configuration.
Suppose we have two queues, which are A and B, and the YARN is configured to Fari Scheduler. A start the application when B is free, then A can take over all the resource of queue B, but when B start an application while A is free, B can also take over all the resource. The key point of Fair Scheduler is that the resouce is fairly shared for all queues. 

## References
* [Hadoop YARN Tutorial – Learn the Fundamentals of YARN Architecture](https://www.edureka.co/blog/hadoop-yarn-tutorial/)
* [Hadoop YARN Resource Manager – A Yarn Tutorial](https://data-flair.training/blogs/hadoop-yarn-resource-manager/)
* [Hadoop Yarn Concept with Fifo, Capacity and Fair Scheduling](http://timepasstechies.com/hadoop-yarn-concept-fifocapacity-fair-scheduling/)
