# sqoop数据导入导出应用案例

## 1 sqoop导入数据

将RDBMS中的一个表数据导入到hdfs。表中的每一行被视为hdfs的记录。所有记录都存储为文本文件的文本数据（或者Avro、sequence文件等二进制数据）。

### 1.1 语法

下面的命令用于将数据导入到hdfs上。
`$sqoop import (generic-args) (import-args)`

### 1.2 测试数据

在MySQL有一个userdb的数据库，其中有一张usertable表，该表结构如下：

 id | name   | age
----|--------|----
 2  | tom    | 15
 3  | toms   | 25
 4  | tomslee| 17
 5  | bob    | 16

### 1.3 导入表中数据到HDFS

下面的命令用于从MySQL数据库服务器中的usertable表导入数据到hdfs。
```
sqoop import \
--connect jdbc:mysql://mysqlhost:3306/userdb \
--username root \
--password root \
--table usertable \
--m 1
```

上面的命令中`--connect`指的是连接地址，这里面是mysql服务器的地址；`--table usertable`是MySQL数据库的数据表；`--m 1`是指定MapReduce的数量。

如果执行成功，会显示如下输出：
```
14/12/22 15:24:54 INFO sqoop.Sqoop: Running Sqoop version: 1.4.5
14/12/22 15:24:56 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
INFO orm.CompilationManager: Writing jar file: /tmp/sqoop-hadoop/compile/cebe706d23ebb1fd99c1f063ad51ebd7/emp.jar
-----------------------------------------------------
O mapreduce.Job: map 0% reduce 0%
14/12/22 15:28:08 INFO mapreduce.Job: map 100% reduce 0%
14/12/22 15:28:16 INFO mapreduce.Job: Job job_1419242001831_0001 completed successfully
-----------------------------------------------------
-----------------------------------------------------
14/12/22 15:28:17 INFO mapreduce.ImportJobBase: Transferred 145 bytes in 177.5849 seconds (0.8165 bytes/sec)
14/12/22 15:28:17 INFO mapreduce.ImportJobBase: Retrieved 5 records.
```

上述的过程由于没有指定hdfs的保存位置，所以系统会分配一个默认的地址，该地址根据当前的用户名和表名生成的。
为了验证在hdfs导入的数据，使用下面的命令可以查看：
`hadoop fs -cat /user/hadoop/userdb/part-m-00000`
默认情况下hdfs上面的数据字段之间用逗号（,）分割。
![sqoop1](http://i.imgur.com/1ocr81G.png)

### 1.4 导入到hive表中

```
sqoop import \
--connect jdbc:mysql://mysqlhost:3306/userdb \
--username root \
--password root \
--table usertable \
--hive-import \
--m 1;
```
调用上述命令之前不用先建立hive数据表，由于没有指定hive的数据库，所以系统会在hive的default数据库下面建立一张usertable数据表。
**数据传输过程：**
>1 从MySQL到hdfs上（通过MapReduce）
>2 从hdfs迁移到hive中

```
hive> select * from usertable;
OK
2       tom     15
3       toms    25
4       tomslee 17
5       bob     16
Time taken: 0.191 seconds, Fetched: 4 row(s)
hive> dfs -cat /user/hive/warehouse/usertable/part-m-00000;
2tom15
3toms25
4tomslee17
5bob16
hive>

```
通过查看hdfs上的数据可知hive中的字段之间默认是用`'\001'`进行分割的，所以字段之间看起来紧挨着。

### 1.5 导入到hdfs指定的目录上

在导入表数据到hdfs使用sqoop工具，我们可以指定目标目录。
以下是指定目标目录选项的sqoop导入命令到的语法。
`--target-dir<new directory in HDFS>`
下面的命令是用来导入MySQL数据库的user数据表到hdfs的/user/test目录。

```
sqoop import \
--connect jdbc:mysql://mysqlhost:3306/userdb \
--username root \
--password root \
--target-dir /uer/test \
--table usertable \
--m 1;
```

**注意：**指定的hdfs的目录不能存在，因为sqoop会将这个目录作为MapReduce的输出目录。
导入到hdfs上的输出数据格式如下：
```
2,tom,15
3,toms,25
4,tomslee,17
5,bob,16

```

### 1.6 导入表数据子集

我们可以导入表的"where"子句的一个子集通过sqoop工具。它执行在各自的数据库服务器相应的sql查询中，并将结果储存在hdfs的目标目录上。
where子句的语法如下：
`--where <condition>`
下面的命令用来 导入usertable表的数据子集。子集查询用户的姓名和年龄。
```
sqoop import \
--connect jdbc:mysql://mysqlhost:3306/userdb \
--username root \
--password root \
--table usertable \
--where "name='tom' and age=15" \
--target-dir /user/test \
--m 1;
```

**注意：**指定的hdfs的目录不能存在，因为sqoop会将这个目录作为MapReduce的输出目录。
导入到hdfs上的输出数据格式如下：
```
2,tom,15
```

### 1.7 增量导入数据

增量导入是仅导入新添加的表中的行的技术。
它需要添加'incremental','check-column','last-value'选项来执行增量导入。
下面的语法用于sqoop导入命令 增量的选项。
```
--incremental <mode>
--check_column <column name>
--last-value <last check column value>
```
下面命令用于在user表执行增量导入。
```
sqoop import \
--connect jdbc:mysql://mysqlhost:3306/userdb \
--username root \
--password root \
--table usertable \
--incremental append \
--check-column id \
--last-value 2 \
--target-dir /user/test \
--m 1;
```
**注意：**这里面指定的hdfs路径不但可以存在而且在该目录下还可以有文件存在。

## 2 sqoop数据导出

将数据从hdfs导出到RDBMS数据库。
导出前，目标表必须存在于目标数据库中。
>默认操作是从将文件中的数据使用insert语句插入到mysql数据表中。
>更新模式下，是生成update语句更新表数据。

### 2.1 语法

以下是export命令的语法。
`sqoop export (generic-args) (export-args)`

### 2.2 案例一

将hdfs中的数据导出到MySQL的usertable表中。
```
sqoop export \
--connect jdbc:mysql://mysqlhost:3306/userdb \
--username root \
--password root \
--table usertable \
-export-dir /user/hive/warehouse/usertable \
--input-fields-terminated-by '\001'
--m 1;
```
上述命令中的`input-fields-terminated-by '\001'`指的是输入的字段之间的分隔符，在hive中的默认分隔符为'\001'。

验证：在MySQL中输入`select * from usertable;`

## 3 sqoop数据导入导出命令详解

### 3.1 sqoop import导入数据命令参数详解

输入`sqoop import --help`命令可以查看所有导入命令的参数详解：

```

常用命令：
   --connect <jdbc-uri>                         JDBC连接字符串                                         
   --connection-manager <class-name>            连接管理者                                             
   --driver <class-name>                        驱动类
   --hadoop-home <dir>                          指定$HADOOP_HOME路径
   -P                                           从命令行输入密码（这样可以保证数据库密码的安全性）
   --password <password>                        密码
   --username <username>                        用户名
   --verbose                                    打印信息


Import control arguments:
   --append                        添加到hdfs中已经存在的dataset上
                                   直接使用该参数就可向一个已经存在的目录追加内容了
   --as-avrodatafile               导入数据作为avrodata
   --as-sequencefile               导入数据作为SequenceFiles
   --as-textfile                   默认导入数据为文本
   --boundary-query <statement>    Set boundary query for retrieving max
                                   and min value of the primary key
   --columns <col,col,col...>      选择导入的列
   --compression-codec <codec>     压缩方式，默认是gzip
   --direct                        使用直接导入快速路径
   --direct-split-size <n>         在快速模式下每n字节使用一个split
-e,--query <statement>             通过查询语句导入
   --fetch-size <n>                一次读入的数量                            
   --inline-lob-limit <n>          Set the maximum size for an inline LOB
-m,--num-mappers <n>               通过实行多少个map，默认是4个，某些数据库8 or 16性能不错
   --split-by <column-name>        创建split的列，默认是主键
   --table <table-name>            导入的数据表
   --target-dir <dir>              HDFS 目标路径
   --warehouse-dir <dir>           HDFS parent for table destination
   --where <where clause>          WHERE clause to use during import
-z,--compress                      Enable compression


增量导入参数:
   --check-column <column>        Source column to check for incremental
                                  change
   --incremental <import-type>    Define an incremental import of type
                                  'append' or 'lastmodified'
   --last-value <value>           Last imported value in the incremental
                                  check column

输出行格式参数:
   --enclosed-by <char>               设置字段结束符号
   --escaped-by <char>                用哪个字符来转义
   --fields-terminated-by <char>      输出字段之间的分隔符
   --lines-terminated-by <char>       输出行分隔符
   --mysql-delimiters                 使用mysql的默认分隔符: , lines: \n escaped-by: \ optionally-enclosed-by: '

   --optionally-enclosed-by <char>    Sets a field enclosing character

输入参数解析:
   --input-enclosed-by <char>               Sets a required field encloser
   --input-escaped-by <char>                Sets the input escape
                                            character
   --input-fields-terminated-by <char>      输入字段之间的分隔符
   --input-lines-terminated-by <char>       输入行分隔符
                                            char
   --input-optionally-enclosed-by <char>    Sets a field enclosing
                                            character

Hive arguments:
   --create-hive-table                         创建hive表,如果目标表存在则失败

   --hive-delims-replacement <arg>             导入到hive时用自定义的字符替换掉 \n, \r, and \001

   --hive-drop-import-delims                   导入到hive时删除 \n, \r, and \001
   --hive-home <dir>                           重写$HIVE_HOME
   --hive-import                               Import tables into Hive
                                               (Uses Hive's default
                                               delimiters if none are
                                               set.)
   --hive-overwrite                            Overwrite existing data in
                                               the Hive table
   --hive-partition-key <partition-key>        hive分区的key
   --hive-partition-value <partition-value>    hive分区的值
                                              
   --hive-table <table-name>                   Sets the table name to use
                                               when importing to hive
   --map-column-hive <arg>                     类型匹配，sql类型对应到hive类型
```

### 3.2 sqoop export导出数据命令参数详解

输入`sqoop export --help`命令可以查看所有导入命令的参数详解：

```
export主要参数
--direct     快速导入
--export-dir <dir>                            HDFS到处数据的目录
-m,--num-mappers <n>                          都少个map线程
--table <table-name>                          导出哪个表
--call <stored-proc-name>                     存储过程
--update-key <col-name>                       通过哪个字段来判断更新
--update-mode <mode>                          插入模式，默认是只更新，可以设置为allowinsert.
--input-null-string <null-string>             字符类型null处理
--input-null-non-string <null-string>         非字符类型null处理
--staging-table <staging-table-name>          临时表
--clear-staging-table                         清空临时表
--batch                                       批量模式
```

## 4 参考博文

[http://www.cnblogs.com/cenyuhai/p/3306056.html](http://www.cnblogs.com/cenyuhai/p/3306056.html)
[http://www.aboutyun.com/thread-9983-1-1.html](http://www.aboutyun.com/thread-9983-1-1.html)
[http://blog.csdn.net/wangyang1354/article/details/52936400](http://blog.csdn.net/wangyang1354/article/details/52936400)