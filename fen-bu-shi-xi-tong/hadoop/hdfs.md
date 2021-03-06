# HDFS

HDFS是一个分布式文件系统，具有高容错的特点。它可以部署在廉价的通用硬件上，提供高吞吐率的数据访问，适合需要处理海量数据集的应用程序。

主要特点：

1、支持超大文件：支持TB级的数据文件。

2、检测和快速应对硬件故障：HDFS的检测和冗余机制很好克服了大量通用硬件平台上的硬件故障问题。

3、高吞吐量：批量处理数据。

4、简化一致性模型：一次写入多次读取的文件处理模型有利于提高吞吐量。

HDFS不适合的场景：低延迟数据访问；大量的小文件；多用户写入文件、修改文件。

HDFS的构成：NameNode保存着HDFS的名字空间，对于任何对文件系统元数据产生修改的操作；DataNode将HDFS数据以文件的形式存储在本地文件系统中，它并不知道有关HDFS文件的信息。

数据块：数据块是HDFS的文件存储处理单元，在Hadoop 2.0中默认大小为128MB，可根据业务情况进行配置。数据块的存在，使得HDFS可以保存比存储节点单一磁盘大的文件，而且简化了存储管理，方便容错，有利于数据复制。

## reference

* [https://blog.csdn.net/carl810224/article/details/51910975](https://blog.csdn.net/carl810224/article/details/51910975) 

