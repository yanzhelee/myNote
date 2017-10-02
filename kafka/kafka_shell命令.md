# Kafka的基本shell命令

## 1 创建Kafka topic

```
bin/kafka-topics.sh --zookeeper s100:2181 --create --topic test1 --partitions 3 --replication-factor 2
```
**注**：partitions指定topic分区数，replication-factor指定topic每个分区的副本数。

> partitions分区数：
> - partitions：分区数，控制topic将分片成多少个log。可以显示指定，如果不指定则会使用broker(server.properties)中的num.partitions配置的数量。
> - 虽然增加分区数可以提供kafka集群的吞吐量，但是过多的分区数或者是单台服务区上的分区数过多，会增加不可用及延迟的风险。因为多的分区数，意味着需要打开更多的文件句柄、增加点到点的延时、增加客户端的内存消耗。
> - 分区数也限制了consumer的并行度，即限制了并行consumer消息的线程数不能大于分区数。
> - 分区数也限制了producer发送消息是指定的分区。如创建topic时分区设置为1，producer发送消息时通过自定义的分区方法指定分区为2或以上的数都会出错的；这种情况可以通过alter-partitions来增加分区数。
>
> replication-factor副本
> - replication factor控制消息保存在几个broker(服务器)上，一般情况下等于broker的个数,replication-factor设置的值不能大于broker的个数。
> - 如果没有在创建时显示指定或通过api向一个不存在的topic生产消息时会使用broker(server.properties)中的default.replication.factor配置的数量。

## 2 查看所有的topic列表

```
bin/kafka-topics.sh --zookeeper s100:2181 --list
```

## 3 查看指定的topic信息

```
bin/kafka-topics.sh --zookeeper s100:2181 --describe --topic test1
```

## 4 控制台向topic生产数据

```
bin/kafka-console-producer.sh --broker-list s101:9092 --topic test1
```

## 5 控制台消费topic的数据

```
bin/kafka-console-consumer.sh --zookeeper s100:2181 --topic test1 --from-beginning
```

## 6 查看topic某分区偏移量最大(小)值

```
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --topic hive-mdatabase-hostsltable --time -1 --broker-list s101:9092 --partitions 0
```
**注**：time为-1时表示最大值，time为-2时表示最小值

## 7 增加topic分区数(kafka的分区只允许增加，不允许减少)

为topic test1设置为10个分区
```
bin/kafka-topics.sh --zookeeper s101:2181 --alter --topic test1 --partitions 10
```

## 8 删除topic

默认情况下kafka的topic是没法直接删除的，需要进行相关参数的设置

```
# 在默认情况下删除是标记删除，没有实际删除topic
bin/kafka-run-class.sh kafka.admin.DeleteTopicCommand --zookeeper s101:2181 --topic test1
```
**实际删除topic的两种方式**
1. 通过delete命令删除后，手动将本地磁盘以及zk上的相关topic信息删除即可
2. 配置server.properties文件，给定参数delete.topic.enable=true,重启kafka服务，此时执行delete命令表示允许进行topic的删除

## 9 查看topic消费进度

这个会显示出consumer group的offset情况，必须参数为--group，不指定--topic，默认为所有的topic

```
bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --group pv

Group           Topic              Pid Offset   logSize    Lag    Owner
pv              page_visits        0   21       21         0      none
pv              page_visits        1   19       19         0      none
pv              page_visits        2   20       20         0      none
```






## 参考博文

[http://www.cnblogs.com/xiaodf/p/6093261.html](http://www.cnblogs.com/xiaodf/p/6093261.html)
