# 深入了解HBase结构

## 1 HBase 架构组成

HBase采用Master/slave结构搭建集群，它属于Hadoop生态系统，由以下类型的节点组成：HMaster节点、HRegion节点、Zookeeper集群，而在底层，它将数据存储与hdfs中，因而涉及hdfs的NameNode、DataNode等。总体结构如下图所示：

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/HBaseStructure.jpg)

### 1.1 HMaster节点

1. 管理HRegionServer，实现其负载均衡。
2. 管理和分配HRegion，比如在HRegion split时分配新的HRegion；在HRegionServer退出时迁移其内的HRegion到其他HRegionServer上。
3. 实现DDL操作。
4. 管理namespace和table的元数据(实际存储在hdfs上)
5. 权限控制(ACL)

### 1.2 HRegionServer节点

1. 存放和管理本地HRegion
2. 读写HDFS，管理Table中的数据。
3. Client直接通过HRegionServer读写数据(从HMaster中获取元数据，找到RowKey所在的HRegion/HRegionServer后)

### 1.3 Zookeeper集群协调系统

1. 存放整个HBase集群的元数据以及集群的状态信息
2. 实现HMaster主从节点的failover

HBase Client通过RPC方式和HMaster、HRegionServer通信；一个HRegionServer可以存放1000个HRegion；底层Table数据存储于HDFS中，而HRegion所处理的数据尽量和数据所在的DataNode在一起，实现数据的本地化；数据本地化并不是总能实现，比如HReigon移动时，需要等下一次Compact才能继续回到本地化。

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/HBaseArchitecture-Blog-Fig1.png)

这个架构图比较清晰的表达了HMaster和NameNode都支持多个热备份，使用Zookeeper来做协调；Zookeeper是由三台机器组成一个集群，内部使用paxos算法支持三台Server中的一台宕机，也有使用五台机器的，此时则可以支持同时两台宕机，即少于半数的宕机，然而随着机器的增加，它的性能也会下降；RegionServer和DataNode一般会放在相同的Server上实现数据的本地化。

## 2 HRegion

HBase使用RowKey将表水平切割成多个HRegion，从HMaster的角度，每个HRegion都记录了它的StartKey和EndKey(第一个HRegion的StartKey为空，最后一个HRegion的EndKey为空)，由于RowKey是排序的，因而Client可以通过HMaster快速的定位每个RowKey在那个HRegion中。HRegion由HMaster分配到相应的HRegionServer中，然后由HRegionServer负责HRegion的启动和管理，和Client的通信，负责数据的都。每个HRegionServer可以同时管理1000个左右的HRegion。

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/HBaseArchitecture-Blog-Fig2.png)

## 3 HMaster

HMaster没有单点故障问题，可以启动多个HMaster，通过Zookeeper的Master Election机制保证同时只有一个HMaster处于Active装填，其他的HMaster处于热备份状态。一般情况下会启动两个HMaster，非Active的HMaster会定期的和Active HMaster通信以获取其最新状态，从而保证它是实时更新的，因而如果启动了多个HMaster反而增加可Active HMaster的分配和管理，DDL的实现等，即它主要有两个方面的职责：
1. 协调HRegionServer
  1. 启动时HRegion的分配，以及负载均衡和修复是HRegion的重新分配。
  2. 监控集群中所有HRegionServer的状态(通过Heartbeat和监听Zookeeper中的状态)。
2. Admin职能
  1. 创建、修改、删除Table的定义。

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/HBaseArchitecture-Blog-Fig3.png)

## 4 ZooKeeper:协调者

Zookeeper为HBase集群提供协调服务，它管理者HMaster和HRegionServer的状态，并且会在他们宕机时通知HMaster，从而HMaster可以实现Hmaster之间的failover，或对宕机的HRegionServer中的HRegion集合的修复(将他们分配给其他的HeegionServer)。ZooKeeper集群本身使用一致性协议(PAXOS协议)保证每个节点状态的一致性。

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/HBaseArchitecture-Blog-Fig4.png)

## 5 组件之间如何协同工作的

ZooKeeper协调集群所有节点的共享信息，在HMaster和HRegionServer连接到Zookeeper后创建Ephemeral节点，并使用Heartbeat机制维持这个节点的存活状态，如果某个Ephemeral节点失效，则HMaster会收到通知，并做相应的处理。

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/HBaseArchitecture-Blog-Fig5.png)

另外，HMaster通过箭筒Zookeeper中的Ephemeral节点来监控HRegionServer的加入和宕机，则该节点消失，因而其他HMaster得到通知，而将自身转换成Active的HMaster，在变为Active的HMaster之前，它汇创建在/hbase/back-masters/下创建自己的Ephemeral节点。

### 5.1 HBase的第一次读写

在HBase 0.96以前，HBase有两个特殊的Table：-ROOT-和.META.（如BigTable中的设计），其中-ROOT- Table的位置存储在ZooKeeper，它存储了.META. Table的RegionInfo信息，并且它只能存在一个HRegion，而.META. Table则存储了用户Table的RegionInfo信息，它可以被切分成多个HRegion，因而对第一次访问用户Table时，首先从ZooKeeper中读取-ROOT- Table所在HRegionServer；然后从该HRegionServer中根据请求的TableName，RowKey读取.META. Table所在HRegionServer；最后从该HRegionServer中读取.META. Table的内容而获取此次请求需要访问的HRegion所在的位置，然后访问该HRegionSever获取请求的数据，这需要三次请求才能找到用户Table所在的位置，然后第四次请求开始获取真正的数据。当然为了提升性能，客户端会缓存-ROOT- Table位置以及-ROOT-/.META. Table的内容。如下图所示：

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/hbase第一次读写.jpg)



## 参考博文

[An In-Depth Look at the HBase Architecture](https://mapr.com/blog/in-depth-look-hbase-architecture/#.VdMxvWSqqko)
[](http://www.blogjava.net/DLevin/archive/2015/08/22/426877.html)
