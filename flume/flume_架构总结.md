# flume架构总结

介绍flume之前先看一下Hadoop业务的整体流程开发：
![](http://i.imgur.com/freI4fd.png)
从hadoop的业务流程图中可以看出，在大数据的业务逻辑处理过程中，对于数据的搜集是十分重要的一步，也是不可避免的一步，本文下面将对flume的架构进行详细的介绍。

## 1.flume概念

flume是一个分布式、可靠和高可用的海量日志聚合的系统，支持在系统中地址各类数据发送方，用于手机数据；同时，flume提供对数据进行简单处理，并写到各种数据接收方（可定制）的能力。

![](http://flume.apache.org/_images/DevGuide_image00.png)

### 1.1设计目标

#### 可靠性

当节点出现故障时，日志能够被传送到其他节点上而不丢失。flume提供三种级别的可靠性保障，从强到弱依次分为：
- end-to-end
> 收集数据agent首先将event写到磁盘上，当数据传送成功后，再删除；如果数据发送失败，可以从新发送。
- Store on failure
> 这也是scribe采用的策略，当数据接收方crash时，将数据写到本地，待恢复后，继续发送。
- Best effort
> 数据发送到接收方后，不会进行确认。

### 可扩展性

Flume采用了三层架构，分别为agent，collector和storage，每一层均可以水平扩展。其中，所有agent和collector由master统一管理，这使得系统容易监控和维护，且master允许有多个（使用ZooKeeper进行管理和负载均衡），这就避免了单点故障问题。


### 可管理性

所有agent和colletor由master统一管理，这使得系统便于维护。多master情况，Flume利用ZooKeeper和gossip，保证动态配置数据的一致性。用户可以在master上查看各个数据源或者数据流执行情况，且可以对各个数据源配置和动态加载。Flume提供了web 和shell script command两种形式对数据流进行管理。

### 功能可扩展性

用户可以根据需要添加自己的agent，collector或者storage。此外，Flume自带了很多组件，包括各种agent（file， syslog等），collector和storage（file，HDFS等）。

## 2.Event概念

flume的核心就是把数据从数据源（source）收集过来，再将收集到的数据送到指定的目的地（sink）。为了保证输送的过程一定成功，在送到目的地之前，会先缓存数据（channel），待数据真正的送到目的地（sink）后，flume再删除缓存中的数据（channel中的数据）。
在整个数据的传输过程中，流动的是event，即事务保证是在event级别进行的。那么什么是event呢？——event将传输的数据进行封装，是flume传输数据的基本单位，如果是文本文件，通常是一行记录，event也是事务的基本单位。event从source，流向channel，再到sink，本身为一个字节数组，并可携带header信息，event代表着一个数据的最小完整单元，从外部数据源来，想外部的目的地去。形象展示如下图：
![](http://i.imgur.com/lmMGnFc.png)
一个完整的event包括：event headers,event body, event信息（即文本文件中的单行记录）。
![](http://img.blog.csdn.net/20160530163629374)
其中event就是flume收集到的日志记录。

## 3.flume架构介绍

flume的关键就是它的设计，这个设计就是agent，啊跟他本身就是一个java进程，运行在日志收集节点（所谓的日志收集节点就是服务器节点）。
agent里面包含三大核心组件：source--->channel--->sink,类似生产者、仓库、消费者的架构。
- source
> source组件是专门用来收集数据的，可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy、自定义数据等。
- channel
> source组件把数据收集以后，临时存放在channel中，即channel组件在agent中是专门用来存放临时数据的——对采集到的数据进行简单的缓存，可以存放在memory、jdbc、file等。
- sink
> sink组件是用于把数据发送大目的地 的组件，目的地包括hdfs、logger、avro、thrift、ipc、file、null、hbse、solr、自定义。

## 4.flume的运行机制

flume的核心就是agent，这个agent对外有两个进行交互的地方，一个是接收数据的输入source，一个是数据的数据sink，sink负责将数据发送到外部指定的目的地。source接收到数据之后，将数据发送给channel，channel作为一个数据缓冲区会临时存放这些数据，随后sink会将channel中的数据发送到指定的地方——例如hdfs等。**注意：**只有在sink将channel中的数据发送成功后，channel才会将临时的数据进行删除，这种机制保证了数据传输的可靠性与安全性。

## 5.flume的广义用法

flume之所以这么神奇，其原因也在于flume可以支持多级flume的agent，即flume可以先后相继，例如sink可以将数据写到下一个agent的source中，这样的话就可以连成串，可以整体处理。flume还支持扇入（fan-in）、扇出（fan-out）。所谓扇入就是source可以接受多个输入，所谓扇出就是sink可以将数据输出多个目的地destination中。
![](http://flume.apache.org/_images/UserGuide_image02.png)

## 参考博文

[http://www.cnblogs.com/oubo/archive/2012/05/25/2517751.html](http://www.cnblogs.com/oubo/archive/2012/05/25/2517751.html)
[http://blog.csdn.net/a2011480169/article/details/51544664](http://blog.csdn.net/a2011480169/article/details/51544664)
