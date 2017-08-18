# sqoop介绍与安装

## 1 概述

sqoop是Apache旗下的一款“hadoop和关系型数据库服务器之间传送数据”的工具。

**导入数据**：将关系型结构化数据如MySQL，oracle数据导入到hadoop的hdfs、hive、hbase的数据存储系统。

**导出数据**：从hadoop的文件系统中导出数据到关系型数据库。

![](http://i.imgur.com/9h46sBy.png)

## 2 工作机制

将导入导出命令解析成MapReduce程序来实现，解析出的MapReduce中主要是对inputformat和outputformat进行定制。

## 3 sqoop安装

安装sqoop的前提是已经具备java和hadoop的环境。

### 3.1 下载并解压

最新版下载地址

[http://ftp.wayne.edu/apache/sqoop/1.4.6/](http://ftp.wayne.edu/apache/sqoop/1.4.6/)

### 3.2 修改配置文件

```
$ cd $SQOOP_HOME/conf
$ mv sqoop-env-template.sh sqoop-env.sh
# 打开sqoop-env.sh并编辑下面几行：
export HADOOP_COMMON_HOME=/soft/hadoop-2.6.1/ 
export HADOOP_MAPRED_HOME=/soft/hadoop-2.6.1/
export HIVE_HOME=/soft/hive-1.2.1
```

### 3.3 加入mysql的jdbc驱动包
可以从hive的lib中拷贝MySQL驱动到sqoop的lib目录下。
`cp /soft/hive/lib/mysql-connector-java-5.1.28.jar $SQOOP_HOME/lib`
如果没有mysql的驱动的话需要自己到Apache官网去下载。

### 3.4 验证启动
输入`$sqoop version`命令如果成功应该显示如下信息：
<pre>
15/12/17 14:52:32 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6
Sqoop 1.4.6 git commit id 5b34accaca7de251fc91161733f906af2eddbe83
Compiled by abe on Fri Aug 1 11:19:26 PDT 2015
</pre>

OK! 到这里sqoop的安装已经完成。
