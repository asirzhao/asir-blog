---
title: Arvo, Parquet and ORC
date: 2019-06-05 16:34:37
tags: 
    - hdfs
    - hive
categories: hadoop
toc: true
---
![](https://github.com/JoeAsir/blog-image/raw/master/blog/background/architecture-body-of-water-buildings-2255985.jpg)
Dealing with HIVE is one of my daily work with which I read data from and write back to the HDFS. There are many storage formats in HIVE, such as textFile, Avro,  and so on. Today we will talk about three popular formats that are widely use in HIVE world, also in Spark and even the entire distributed file system world. Not talking about some classical formats like textFile , SequenceFile, RCFile, doesn't mean that they are not important or good enough, so you'd better have some look at them to help you make sense of the storage formats in HIVE.
<!--more-->
### Row Formats & Columnar Formats
Row formats and Columnar formats are two methods of seriallizing and storing a table in Database. And the following figure presents the difference of data stored in Row formats and in Columnar fromats.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/22/22-1.png)

### Text formats & Binary formats
Text and Binary Formats are two methods of storing data in another dimension. Text formats are human-readable, easy to generate and easy to parse, however, they occupy a lot of disk space because of the readablity and redundancy. The most popular Text formats includs CSV, TSV, JSON and so on. And, Binary formats, occupy much less storage space than Test formats while they are machine-readable rather than human-readable. However, saving in  Binary formats can make all the storage in HDFS more effcient. Also, the formats we are going to talk about, Avro, Parquet and ORC formats, are all Binary formats.

## Avro
Avor is one of **row-based** formats, and boldly speaking, Avro is a binary alternative to JSON. Avor provides rich data strucures with the **schema**, and the schema file is separately stored from the data file. Also, Avro can generate the serialization and deserialization code form the schema. The figure below presents the sturcture of Avro.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/22/22-2.png)

As presented, Avor firstly splits data into multiple **blocks** rather than stores data record by record. The schema infomation is stored in the header and the data information are in blocks followed by a **sync marker** for each block. The sync markers make the Avor format splittable and the schema make it extensibility.  
## Parquet
Parquet is a **Columnar format**, which is based on the Google Dremel paper, and it's one of the most popular Columnar formats in Hadoop ecosystem and it's well integrated with Apache Spark. What's more, Parquet can easily deal with the nested schema. The following figure shows the structure of Parquet. 
![](https://github.com/JoeAsir/blog-image/raw/master/blog/22/22-3.png)

First we need to learn some terms about Parquet:
* Blocks: The block is actually the block in HDFS, which is unchanged for Parquet;
* File: The HDFS file with metadata;
* Row group: A logical horizontal partitioning of the data into rows. A row group consists of a column chunk for each column in the dataset;
* Column chunk: A chunk of data for a specific column in one Row group. A column chunk consists of one or more than one page;
* Page: Column chunks are devided into pages. And it is conceptually indivisible unit.

All the metadata for file, row groups, and column chunks are all stored in a footer, which is the tail of the whole file. The figure presents the structure of Parquet clearly and if you want to learn the detail, you can take a look at the [offcial doc](https://parquet.apache.org/documentation/latest/)

## ORC
ORC(Optimized Row Columnar) is also a **Columnar format** and it is optimized from RCFile format, which is also a Columar format. ORC is now widely used in Hadoop ecosystem especially for HIVE. Let's start from the ORC structure.
![](https://github.com/JoeAsir/blog-image/raw/master/blog/22/22-4.png)

First we have to make some terms clearly:
* Stripe: Simialr to the row groups in Parquet, stripe in ORC is a group of row data. Each stripe holds index data, row data and a stripe footer;
* Index data: Include the min and max values and row positions within each column in one stripe;
* Row data: The real data value;
* Stripe footer: Contains the directoty of stream location.

Now let's talk about an awesome feature in ORC, which is based on the index.
### Indexes
There are three level of indexes in ORC, which are file, stride and row level index. They contain some statistics like the max/min value and the position information of each vale for specific dimesion. 
> * file level - statistics about the values in each column across the entire file
> * stripe level - statistics about the values in each column for each stripe
> * row level - statistics about the values in each column for each set of 10,000 rows within a stripe(the value can be modified by *orc.row.index.stride*)

File and stride index are in the file footer at the tail of the whole file, and as soon as the file is read, the information can be loaded directly and the useless part of data would be ignored immediately. The row level index is stored in the Index data in each stride and it contains the count of the values and whether there are null value present, and also the min/max values. Besides, the row index can include bloom filter, which will be talked in the next sub-section.
To make leverage of ORC index, we'd better insert the data into ORC table with sorting first, usually *distributed by* and *sort by*, to make the data are sorted well in each reducer result. Let's have a simple example. Suppose we have a query with *WHERE id > 10*, the entired file would be filter by the file level index, stride index and row level index. If the file is inserted sorted by *id*, the min/max value in each index would not be overlapping.

### Bloom Filter
Since Hive 1.2, the bloom filter is supported in row index. If you've never heared about bloom filter, don't worry about it and click [here](https://en.wikipedia.org/wiki/Bloom_filter) to have a quick look.
As the last sub-section, the indexes are in good use to filter data and make the query more efficient. But the statistics value like min/max are more efficient for those continuously numerical type columns. When you have some operation like *=* or *in*, bloom filter would be a better choice. 
With the configuration *orc.bloom.filter.columns* and *orc.bloom.filter.fpp*, we can figure out the colunms(splitted by comma) with bloom filter and specify the positive probability for the bloom filter. 

## References
* [Parquet Documentation](https://parquet.apache.org/documentation/latest/)
* [ORC Documentation](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-orc-spec)
* [Apache Parquet: Parquet File Internals and Inspecting Parquet File Structure](https://www.youtube.com/watch?v=rVC9F1y38oU)
* [ORC Specification v1](https://orc.apache.org/specification/ORCv1/)
* [ORC Indexes](https://orc.apache.org/docs/indexes.html)
* [HIVE Optimizations with Indexes, BLOOM-FILTERS and Statistics](https://snippetessay.wordpress.com/2015/07/25/hive-optimizations-with-indexes-bloom-filters-and-statistics/)
