# MapReduce模型初探（二）
## 目录:
[TOC]
## 一、MR执行流程

1. 最简单过程：map --> reduce
2. 定制了Partitioner分区的过程：map-->partition-->reduce
3. 增加了本地优化(本地reduce)过程：map-->combin(本地reduce)-->partition-->reduce

MR详细的执行流程图：

![MR详细的执行流程图](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hadoop/hadoop_mapRdcude%E6%A8%A1%E5%9E%8B%E5%88%9D%E6%8E%A2_1.jpg)

改图一共分为两大部分：
1. map阶段
	1. 读取输入文件内容，解析成k-v对。对输入文件的每一行，解析成k-v对。
	2. 执行自定义map函数过程，对输入k-v进行处理，每一个k-v对调用一次map函数，然后输出新的k-v对。
	3. 对输出的k-v进行分区。
	4. 对不同分区的数据，按照key进行排序和分组。相同的Key的value放在同一个集合中。
	5. 本地reduce过程，对分组后的数据进行规约。（该过程不是必需的）
2. reduce阶段
	1. 对多个map任务的输出（或者是多个combin过程的输出），按照不同的分区，通过网络传输到不同的reduce节点上。
	2. 对map任务的输出进行合并、排序。
	3. 自定义reduce阶段，对输入的k-v进行处理，转换成新的k-v。
	4. 把reduce的输出保存到hdfs上。

## 二、shuffle阶段

shuffle过程如图所示：

![shuffle过程](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hadoop/hadoop_mapRdcude%E6%A8%A1%E5%9E%8B%E5%88%9D%E6%8E%A2_2.png)

通过改图可知shuffle共分为五个阶段，下面进行详解。

### 2.1 shuffle中的分区(Partition)

1. 作用与原理
> 得到map的输出k-v之后，hadoop默认会根据key的``` hashcode%reducetask ```的值来进行分区。这样的分区方式在实际中可能会导致数据倾斜，也就是说有的节点处理的数据量特别大，有的节点的数据量特别小。另外还有一些特殊的需求，比如：在一个实际项目中需要对不同地区的手机的上网流量进行统计和分析，如果实现该过程就得需要通过以手机号码作为key，然后根据手机号前三位进行分区处理。
> 所以针对特定的需求我们需要自定义Partition处理逻辑。

2. 如何自定义Partitioner
> 自定义partitioner很简单，只要自定义一个类，并且继承Partitioner类，重写其getPartition方法就好了。在使用的时候通过调用Job的setPartitionerClass()方法来指定对应的Partitioner类。具体使用方法参见我的blog《MapRedece中的分区Partitioner》：[http://blog.csdn.net/u010521842/article/details/74908443](http://blog.csdn.net/u010521842/article/details/74908443 "MapRedece中的分区Partitioner")

### 2.2 shuffle中的排序和分组

1. 排序
> 就是将map阶段输出的Key按照一定的规则进行排序，具体的排序方式是根据Key类型的类中定义的``` compareTo() ```方法定义的，如果想自定义排序规则,就得需要自定义Key类型并且实现``` WritableComparable ```接口。

2. 分组
> 分区的目的是根据Key值决定Mapper的输出记录被送到哪一个Reducer上去处理。而分组就是在同一个分区里面，具有相同Key值的记录是属于同一个组的。

### 2.3 shuffle中的Combiner

每一个map可能会产生大量的输出，combiner的作用就是在map端对输出先做一次合并，以减少传输到reducer的数据量。 所以combiner的功能往往根reducer的功能一样。combiner最基本是实现本地key的归并，combiner具有类似本地的reduce功能。如果不用combiner，那么，所有的结果都是reduce完成，效率会相对低下。使用combiner，先完成的map会在本地聚合，提升速度。

### 2.4 Shuffle阶段排序流程详解

![Shuffle阶段排序流程](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hadoop/hadoop_mapRdcude%E6%A8%A1%E5%9E%8B%E5%88%9D%E6%8E%A2_3.jpg)

MapReduce中的排序的总体流程
> MapReduce框架会确保每一个Reducer的输入都是按Key进行排序的。一般，**将排序以及Map的输出传输到Reduce的过程称为混洗（shuffle)**。每一个Map都包含一个环形的缓存，默认100M，Map首先将输出写到缓存当中。当缓存的内容达到“阈值”时（阈值默认的大小是缓存的80%），一个后台线程负责将结果写到硬盘，这个过程称为“spill”。Spill过程中，Map仍可以向缓存写入结果，如果缓存已经写满，那么Map进行等待。

Spill的具体过程如下：
> 首先，后台线程根据Reducer的个数将输出结果进行分组，每一个分组对应一个Reducer。其次，对于每一个分组后台线程对输出结果的Key进行排序。在排序过程中，如果有Combiner函数，则对排序结果进行Combiner函数进行调用。每一次spill都会在硬盘产生一个spill文件。因此，一个Map task有可能会产生多个spill文件，当Map写出最后一个输出时，会将所有的spill文件进行合并与排序，输出最终的结果文件。在这个过程中Combiner函数仍然会被调用。从整个过程来看，Combiner函数的调用次数是不确定的。下面我们重点分析下Shuffle阶段的排序过程：

Shuffle阶段的排序可以理解成两部分
> 一个是对spill进行分区时，由于一个分区包含多个key值，所以要对分区内的key-value按照key进行排序，即key值相同的一串key-value存放在一起，这样一个partition内按照key值整体有序了。

> 第二部分并不是排序，而是进行merge，merge有两次，一次是map端将多个spill 按照分区和分区内的key进行merge，形成一个大的文件。第二次merge是在reduce端，进入同一个reduce的多个map的输出 merge在一起，该merge理解起来有点复杂，最终不是形成一个大文件，而且期间数据在内存和磁盘上都有。所以shuffle阶段的merge并不是严格的排序意义，只是将多个整体有序的文件merge成一个大的文件，由于不同的task执行map的输出会有所不同，所以merge后的结果不是每次都相同，不过还是严格要求按照分区划分，同时每个分区内的具有相同key的key-value对挨在一起。

Shuffle排序综述：
> 如果只定义了map函数，没有定义reduce函数，那么输入数据经过shuffle的排序后，结果为key值相同的输出挨在一起，且key值小的一定在前面，这样整体来看key值有序（宏观意义的，不一定是按从大到小，因为如果采用默认的HashPartitioner，则key 的hash值相等的在一个分区，如果key为IntWritable的话，每个分区内的key会排序好的），而每个key对应的value不是有序的。

## 参考博文：
[http://blog.sina.com.cn/s/blog_d76227260101d948.html](http://blog.sina.com.cn/s/blog_d76227260101d948.html)

[http://blog.csdn.net/u014307117/article/details/45223291](http://blog.csdn.net/u014307117/article/details/45223291)
