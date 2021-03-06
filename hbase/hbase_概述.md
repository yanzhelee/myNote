# HBASE 概述

## HBase是什么

HBase是一种构建在HDFS之上的分布式、面向列的存储系统。在需要实时读写、随机访问超大规模数据集时，可以使用HBase。

HBase通过线性方式从下之上增加节点进行扩展。HBase不是关系型数据库，也不支持SQL，但是它有自己的特长，这是RDBMS不能处理的，HBase巧妙地将大而稀疏的表放在商用的服务器集群上。

HBase是Google Bigtable的开源实现，与Google Bigtable利用GFS作为其文件存储系统类似，HBase利用HDFS作为其文件存储系统；Google运行MapRedue来处理Bigtable中的海量数据，HBase同样利用Hadoop MapReduce来处理HBase中的海量数据；Google Bigtable利用Chubby作为协同服务，HBase利用Zookeeper作为对应。

## HBase和RDBMS

|                       HBase                       |                        RDBMS                        |
| ------------------------------------------------- | --------------------------------------------------- |
| HBase无模式，它不具有固定模式的概念；仅定义列簇。 | RDBMS有模式，它的模式是用于描述表的整体结构的约束。 |
| 它专门创建为宽表。HBase是横向扩展。               | 这些都是细而专的小表。很难形成规模                  |
| 没有任何事物存在于HBase                           | RDBMS是事务性的。                                   |
| 它反规范化数据                                    | 它具有规范化的数据                                  |
| 它用于半结构以及结构化数据                        | 用于结构化数据。                                    |


## HBase的特点

> 大 ：一个表可以有数十亿行和上百万列
> 面向列 ：面向列表（簇）的存储和权限控制，列簇独立检索。
> 稀疏 ：对于为空的列，并不是占用存储空间，因此表可以设计的非常稀疏。
> 无模式 ：每一行都有一个可以排序的主键和任意多的列，列可以根据需要动态增加，同一张表中不同的行可以有截然不同的列。
> 数据多版本 ：每个单元中的数据可以有多个版本，默认情况下，版本号自动分配，版本号就是单元格插入时的时间戳。
> 数据类型单一 ：HBase中的数据都是字符串，没有类型。

## HBase的高并发和实时处理数据

Hadoop是一个高容错，高延时的分布式文件系统和高并发的皮处理系统，不适用于提供实时计算；HBase是一个可以提供实时计算的分布式数据库，数据被保存在hdfs分布式文件系统中，有hdfs保证高容错性，但是在生产环境中，HBase是如何基于Hadoop提供实时性呢？HBase上的数据是以StroeFile(HFile)二进制文件，也就是说，HBase的存储数据对于hdfs文件系统是透明的。下面是HBase文件在HDFS上的存储示意图。

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/hbase_%E6%A6%82%E8%BF%B01.png)

HBase HRegion servers集群中的所有的region的数据在服务器启动时都是被打开的，并且在内存初始化一些memstore，相应的这就在一定程度上加快系统相应；而Hadoop中的block中的数据文件默认是关闭的，只有在需要的时候才打开，处理完数据后就 关闭，这在一定程度上就增加了响应时间。

从根本上说，HBase能提供实时计算服务主要原因是由其架构和底层的数据结构决定的，即由LSM-Tree + HTable(region分区) + Chche决定。客户端可以直接 定位到要查数据所在的HRegion server服务器，然后直接在服务器的一个region上查找要匹配的数据，并且这些数据部分是经过cache缓存的。具体查询 流程如下图所示：

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/hbase_%E6%A6%82%E8%BF%B02.png)

具体数据访问流程如下：

1. Client会通过内部缓存的相关的ROOT中的信息和.META中的信息直接与请求数据匹配的HRegion server；
2. 然后直接定位到该服务器上与客户端请求对应 的Region，客户端请求首先会查询该Region在内存中的缓存，即Memstore(Memstore是一个按key排序的树形结构的缓冲区);
3. 如果在Memostore中查到结果则直接将结果返回给Client；
4. 在Memstore中没有查到匹配的数据，接下来会读已持久化的StoreFile文件中的数据(storeFile也是按key排序的树形结构的文件，并且是特别为范围查询或block查询优化过的 ；另外HBase读取磁盘 文件是按其基本I/O单元读数据的)。

如果在BlockCache中能查到要找的数据则将结果返回，否则就去相应的StoreFile文件中读取一个block的数据，如果还没有读到要查的数据，就将该数据block放到HRegion Server的blockcache中，然后接着读下一个block的数据，一直循环block数据直到找到要请求的数据并返回结果；如果该Region中的数据都没有查到要找的数据，最后返回null，表示没有匹配的数据。

## HBase数据模型

HBase以表的形式存储数据。表由行和列组成。列划分为若干个列族(row family),如下图所示。

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/hbase_%E6%A6%82%E8%BF%B03.png)

HBase的逻辑数据模型如下：

HBase是一个面向列的数据库，在表中它由行排序。表模式只能定义列簇，也就是键值对。一个表有多个列簇以及每个列簇可以有任意数量的列 。后续列的值连续存储在磁盘上。总之：
- 表是行的集合
- 行是列簇的集合
- 列簇是列的集合
- 列是键值对的集合

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/hbase_%E6%A6%82%E8%BF%B04.png)

### ROW Key

与NoSQL数据库一样，Row Key是用来检索记录的主键。访问HBase table中的行，只有三种方式：
1. 通过单个Row Key访问。
2. 通过Row Key的range全表扫描。
3. Row Key可以使任意字符串 (最大长度是64kb，实际应用中长度一般是10~100bytes)，在HBase内部，Row Key保存为字节数组。
4. 在存储时，数据按照Row Key的字典序(byte order)排序存储。设计key时，要充分排序存储这个特性，将经常一起读取的行存储在一起(位置相关性).

行的一次读写是原子操作(不论一次读写多少列)。这个设计决策能够使用户很容易理解在第同一行进行并发更新操作时的行为。

### 列簇(Column family)

HBase表中的每个列都是归属于某个列簇。列簇是表的Schema的一部分，必须在使用表之前定义。类名都是以列簇作为前缀，例如 courses:history,courses:math都是属于courses这个列簇。

访问控制、磁盘和内存的使用统计都是在列簇层进行的。在实际应用中，列簇上的控制权限能帮助我们管理不同类型的应用，例如，允许一些应用可以添加新的基本数据、一些应用可以 读取基本数据并创建继承的列簇、一些应用则只允许浏览数据(甚至可能因为隐私的原因不能浏览所有数据)。

### 时间戳

HBase中通过Row和Columns确定的一个存储单元称为Cell。每个Cell都保存着同一份数据的多个版本。版本通过时间戳类索引，时间戳 的类型是64位整型。时间戳 可以由HBase(在数据写入时自动)赋值，此时时间戳是精确到毫秒的当前系统时间。时间戳也可以由客户显示 赋值。如果应用程序要避免数据版本冲突，就必须 自己生成具有唯一性的时间戳。每个Cell中，不同版本的数据按照时间倒序排序，即最新的数据排在最前面。

为了避免数据存储在过多版本造成的管理(包括存储和索引)负担,HBase提供了两种数据版本回收方式。一是保存的最后n个版本，二是保存最近一段时间内的版本。用户可以针对每个列簇进行设置。

### Cell

Cell是由`{row key, column(=<family> + <table>),version}`唯一确定的单元。Cell中的数据是没有类型的，全部是字节码形式存储。

## HBase物理存储

Table在行的方向上分割为多个HRegion，每个HRegion分散在不同的RegionServer中。

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/hbase_%E6%A6%82%E8%BF%B05.png)

每个HRegion由多个Store构成，每个Store由一个memStore和0或者多个StoreFile组成，每个Store保存一个Columns Family.

![](https://raw.githubusercontent.com/yanzhelee/myNote/master/images/hbase/hbase_%E6%A6%82%E8%BF%B06.png)


## 参考博文

[http://blog.csdn.net/u010270403/article/details/51648462](http://blog.csdn.net/u010270403/article/details/51648462)
