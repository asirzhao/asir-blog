---
title: HDFS Architecture
date: 2019-04-28 18:25:30
tags: 
    - hdfs
    - hadoop
categories: hadoop
toc: true
---
![](https://github.com/JoeAsir/blog-image/raw/master/blog/background/clouds-dark-dark-clouds-2308671.jpg)
I've been using HDFS as storage for almost 3 years reading data from and writing data to it by HIVE and Spark, but I've never learned the detail. Finally I have some time to watch the [Big Data Essentials](https://www.coursera.org/learn/big-data-essentials/) on [Coursera](https://www.coursera.org/), which inspired me to have a deep dive in HDFS architecture. This blog contains so much about HDFS that I spent 3 days to sum up and mark them down. If anything is worng, it's very nice of you to tell me and I'll figure it out! Let's take a look.
<!--more-->
## Namenode
Namenode is the **master node** of the HDFS, it contains the metadata of the filesystem, such as the number and location of the block, replica and so on. Notice the metadata is stored in-memory for the fast retrival of data.
> Task of Namenode
* Manage file system namespace.
* Regulates client’s access to files.
* It also executes file system execution such as naming, closing, opening files/directories.
* All DataNodes sends a Heartbeat and block report to the NameNode in the Hadoop cluster. It ensures that the DataNodes are alive. A block report contains a list of all blocks on a datanode.
* NameNode is also responsible for taking care of the Replication Factor of all the blocks.

### FsImage & EditLogs
The HDFS Namenade, as the Master Node, manages the whole architecture of the filesystem by the metadata. Data in the metadata are present as FsImage and EditLogs. Let's the detail about them.
* FsImage: FsImage contains the image of the entire filesystem, including serialized form of all the **directories** and file **inodes**. The FsImage is actually stored as a **local file** in the filesystem in Namenode and actually, you can treat FsImage as a snapshot of the present filesystem architecture.
  > Each inode is an internal representation of file or directory’s metadata.  
  
  * EditLogs: Editlogs contains the modification made to the entire filesystem on the most recently FsImage, such as creating, moving, updating, deleting and so on. Also, EditLogs is stored on the Namenode as a local file, similar to the FsImage.
  
As you may get it, FsImage and EditLogs, one for storing the present situation and one for the modification. With the help of these two files, Namenode could can recover the matadata in case of something unexpected. Let's go ahead to the specific strategy.
  
### Secondary Namenode(Checkpoint Node) & Backup Node
FsImage and EditLogs can help us recover the entire HDFS, but is there any problem? Suppose the Namenode has been running for a month, and once it restarted, the Namenode would read the FsImage and EditLogs to rebuild the state of the HDFS. However, since the Namenode has been running for such a long time that the EditLogs could be so large that it would spend a long time to load and parse, even for few hours, which is unacceptable for us. To solve this problem, let's have a look at the Secondary Namenode and Backup Node.
* Secondary Namenode(Checkpoint Node): Secondary Namenode runs on another machine apart from the Namenode, it **fetches the FsImage and EditLogs periodically** from the Namenode and merge them to a start-of-art FsImage, aka a checkpoint, and push it back to the Namenode(it may be little confusing that so-called Secondary Namenode would not upload the checkpoint automaticlly while the checkpoint Node would, I'm not quite sure about it actually). 
* Backup Node: Take care that Backup Node is a different term from Secondary Namenode or Checkpoint Node. Backup Node doesn't fetch the FsImage and EditLogs periodically because it receive a filesystem edit stream from the Namenode. As a result, the state image is always stored **in-memory** on the Backup Node.
    > The Backup Node provides the same functionality as the Checkpoint Node, but is synchronized with the NameNode. 
    
## Datanode
Datanode is the **slaver node** of HDFS, which is actually where the data stored.
> Tasks of Datanode 
* Block replica creation, deletion, and replication according to the instruction of Namenode.
* DataNode manages data storage of the system.
* DataNodes send heartbeat to the NameNode to report the health of HDFS. By default, this frequency is set to 3 seconds.

## Block & Replica 
### Block
Block is unit of data stored on HDFS, which cannot be controlled by us and the value is often 128M by default. Why we need data block? Suppose we have two files stored on the HDFS, one is bigger, 1G for instance, and another is 129M. As these two files been read synchronously, it could be an imbalanced progress. But what if we split all the files to a same unit size and these splitted units would be read in balance. That's why we need data block in HDFS. 
Since we've learnt why we need data block, you may ask, why is data block 128M? When files splitted by data block size, instead of one huge single file, few small chunks are stored on HDFS, and the main information of these chunks, aka metadata, are stored in-memory on the Namenode, including block size, block location and so on. So if the block size is too small that the chunks would be too many, then the Namenode in-memory stroage would be under great pressure. And that's also why the **small files problem** damage to your HDFS. However, on the other hand, too large block size would make the reading data process on datanode slow, which is not a good situation for HDFS. It's a trade-off, and that's why we choose 128M as an eclectic solution for HDFS. 
### Replication Management via Rack Awareness
As written to the HDFS, a single file would be divided into many blocks and these blocks would be stored across the cluster, at the same time, the replica of each block is created and there are 3 replicas by default which is can be modified in setting. The replica is actually the backup data from the blocks in case of the potentially unfavorable conditions of the Datanode, aka HDFS fault tolerance. Let's take a look at how HDFS manage the replication under the rack awareness.
Every Datanode in a cluster is actually a single machine, and several Datanodes are put on one rack for better management and they share the network. Several racks are set in one data center and one cluster may be built across several data centers, which could be in different areas even counties, as results, the network distance between each nodes are different. See the figure below.
When the cluster start to write data to the HDFS, Namenode chooses the Datanode which is closer to the same rack or nearby rack to the write request. This distance is calculated by the rules below. Rack Awareness will choose the Datanode which is closer to get rid of too much network commuication cost.
* Distance is 0 when data in the same node;
* Distance is 2 when data in two different nodes but the same rack;
* Distance is 4 when data in two different racks but the same data center;
* Distance is 6 when data in two different data centers.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/21/21-1.png)

Via Rack Awareness, Namenode will not only choose the namenode to store the data, but also the replicas. Let's make a sample with one data block, once the data block has been already stored on the Datanode, the first replica will be stored in the local Datanode, then the second replica will be cast to another Datanode in the different rack and the third replica will be stored at the different Datanode on the local rack of the second replica. 
![](https://github.com/JoeAsir/blog-image/raw/master/blog/21/21-2.png)
> A simple but nonoptimal policy is to place replicas on the different racks. This prevents losing data when an entire rack fails and allows us to use bandwidth from multiple racks while reading the data. This policy evenly distributes the data among replicas in the cluster which makes it easy to balance load in case of component failure. But the biggest drawback of this policy is that it will increase the cost of write operation because a writer needs to transfer blocks to multiple racks and communication between the two nodes in different racks has to go through switches.

## Read & Write 
### Read Operation 
1. Client opens the file by the *DistributedFileSystem* object;
2. *DistributedFileSystem* calls the Namenode via RPC and get the blocks and replicas location according to the distance between datanode and client, and a *FSDataInputStream* is also returned;
3. With the address of Datanotes, *FSDataInputStream* open the I/O stream and bring data from Datanodes back to the client;
4. Once the reading is finished, client will call *close()* to end up the stream.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/21/21-3.png)

> If the *DFSInputStream* encounters an error while communicating with a datanode, it will try the next closest one for that block. It will also remember datanodes that have failed so that it doesn’t needlessly retry them for later blocks. The *DFSInputStream* also verifies checksums for the data transferred to it from the datanode. If it finds a corrupt block, it reports this to the namenode before the *DFSInputStream* attempts to read a replica of the block from another datanode.

### Write Operation
1. Client sends a **create** request on the *DistributedFileSystem*, and *DistributedFileSystem* makes a RPC call to the Namenode to create a new file in the filesystem's namespace, and Namenode would check for the file names, permission and so on. And a *FSDataOutputStream* containing the Datanode location is returned by the Namenode if everything is OK;
2. *FSDataOutputStream* would split the data into packets and make them a queue, aka data queue, consumed by the *DataStreamer*, which would allocate new blocks by picking a list of suitable Datanodes to store the replica from the Namenode;
3. Assume that the replication factor is set to 3, the list of Datanodes form a pipeline containing 3 Datanodes(the first replica is stored in the local Datanode, so there are 3 Datanodes instead of 4);
4. *DataStreamer* streams the packet to the first Datanode in the pipeline and then forwards it to the second one, then the third one.
5. *FSDataOutputStream* also maintains an interal queue of packets waiting for the acknowledge by Datanodes. Once the acknowledge is send from Datanode in the pipeline, which is sent when the block is stored and the replicas are created, the packet is removed from the packet queue.
6. All the blocks are stored and replicated on the different datanodes, the data blocks are copied in parallel.
7. Client calls *close()* when writing operation finished.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/21/21-4.png)
 
 ## References
 * [Hadoop - NameNode, Checkpoint Node and Backup Node](https://morrisjobke.de/2013/12/11/Hadoop-NameNode-and-siblings/)
 * [Hadoop HDFS Architecture Explanation and Assumptions](https://data-flair.training/blogs/hadoop-hdfs-architecture/)
 * [Big Data Essentials: HDFS, MapReduce and Spark RDD](https://www.coursera.org/learn/big-data-essentials/)
