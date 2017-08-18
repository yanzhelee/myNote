# hive分区表

## 1 为什么出现分区表？
假设有海量的数据保存在hdfs的某一个hive表明对应的目录下，使用hive进行操作的时候，往往会搜索这个目录下的所有文件，这有时会非常的耗时，如果我们知道 这些数据的某些特征，可以事先对他们进行分裂，再把数据load到hdfs上的时候，他们就会被放到不同的目录下，然后使用hive进行操作的时候，就可以在where子句中对这些特征进行过滤，那么对数据的操作就只会在符合条件的子目录下进行，其他不符合条件的目录下的内容就不会被读取，在 数据量非常大的时候，这样节省大量的时间，这种把表中的数据分散到子目录下的方式就是分区表。

## 2 实际例子：
某个电商网站的订单信息，全部的信息量是非常多的，其中有月份的字段，如果想对月份进行汇总分析，找到哪个月的订单最多，hive语句中使用where条件进行过滤即可，但是这样就会扫描这个表中所有的文件，不是这个月的记录也会被扫描，极大的浪费了时间，如果能把数据按照月份分别存放，那么在查询的时候，就只查询这个月的数据了。

分区表是hive中一种常见的**优化手段**。

## 3 创建分区表语法
```hive
hive>create table emp(name string, age int) partitioned by (provice string,city string);
```
用partitioned by指定创建的分区，多个分区意味着多级目录。
创建之后hdfs上的目录结构并不会立即发生变化，因为此时表中还没有数据，往表中插入数据之后，会发现在表名对应的目录下，会多出两级目录。
```hive
hive>insert into emp partition(provice = "hebei", city = "baoding") values("tom",22);
hive>dfs -ls -R /user/warehouse/;
```
输入命令之后会大致显示如下信息：
```
/user/hive/warehouse/base1.db/emp/provice=hebei
/user/hive/warehouse/base1.db/emp/provice=hebei/city=baoding
/user/hive/warehouse/base1.db/emp/provice=hebei/city=baoding/000000_0
```
插入的数据指定了不同的分区之后，会生成不同的文件夹。

### 3.1 批量load数据到不同的分区上
```hive
hive>load data local inpath '/home/user1/emp.txt' overwrite into table t1 partition(provice = "hebei",city = "baoding");
hive>load data local inpath '/home/user1/emp.txt' overwrite into table t1 partition(provice = "hebei",city = "handan");
```
由于指定了不同的分区，数据在上传时就会进入不同的目录下。

## 4 查看表的分区信息
在插入两个分区的数据之后，查看表的分区信息，就会将这个表的所有分区都显示出来。
```hive
hive>show partitions database1.table;
OK
provice=hebei/city=baoding
provice=hebei/city=handan
```

## 5 使用分区对数据进行过滤
分区表使用的分区字段不是在数据中存在的（比如创建了一个国家分区，但是数据中并没有这个字段），分区字段只是为了在HDFS上产生了对应的子目录。在使用hive查询的时候，完全可以使用where对分区进行过滤。
```hvie
hive> select * from t3 where city='changchun';
OK
xiaobai 2       jilin   changchun
toms    34      jilin   changchun
haoling 27      jilin   changchun
Time taken: 0.786 seconds, Fetched: 3 row(s)
---------------------------------------------------
hive> select * from t3 where provice = 'jilin';
OK
xiaobai 2       jilin   changchun
toms    34      jilin   changchun
haoling 27      jilin   changchun
Time taken: 0.22 seconds, Fetched: 3 row(s)
```

## 6 修改分区（重构分区）
修改分区意味着对原有的分区进行增减，即重构分区。

### 6.1 添加分区
不改变分区的级数，而是改变分区字段的值。
两种方式：一种是先插入分区值，再往上放数据；二是插入分区和值同时进行。

**方式一**
先添加一个分区值city = 'shijiazhuang'，再往这个分区下插入数据
```hive
hive>alter table emp add partition(provice = "hebei", city = "shijiazhuang");
hive>show partitions emp;
OK
provice=hebei/city=baoding
provice=hebei/city=handan
provice=hebei/city=shijiazhuang

hive>insert into partition(provice = 'hebei', city = 'shijiazhuang') values('tomslee',26);
hive>select * from emp where city = 'shijiazhuang';
tomslee	26	hebei	shijiazhuang
```

**方式二**
在插入数据的时候直接指定新的分区，即把创建分区和插入数据两个步骤在一次完成
```hive
hive>insert into emp partition(provice = 'hebei',city = 'aa') values ('hello',40);
hive> show partitions emp;
OK
provice=hebei/city=aa
provice=hebei/city=baoding
provice=hebei/city=handan
provice=hebei/city=shijiazhuang
Time taken: 0.162 seconds, Fetched: 4 row(s)
```

## 7 删除分区
分区对应的是hdfs上的目录，分区的删除涉及到对应的数据是否会被删除的问题，如果此表是内部表的话，那么分区的删除意味着对应目录下的数据会被删除，如果是外部表的话，分区的删除就不会删除对应的数据文件。

分别创建两个分区表，一个是管理表，一个是外部表，并在其中放入一些数据，然后尝试删除分区，查看HDFS上的数据是否会被删除。

```hive
#创建两个表，一个管理表，一个外部表，都包含一个分区 
hive> create table managed_t(name string) partitioned by (add string);
hive> create external table ex_table(name string) partitioned by (add string);

#往两个表中上传数据
hive> load data local inpath '/home/user1/name.dat' into table managed_table partition(add = 'beijing');
hive> load data local inpath '/home/user1/name.dat' into table ex_table partition(add = 'beijing');

#两个表的目录结构如下：
/user/hive/warehouse/base1.db/ex_table
/user/hive/warehouse/base1.db/ex_table/add=beijing
/user/hive/warehouse/base1.db/ex_table/add=beijing/name.dat
/user/hive/warehouse/base1.db/managed_t
/user/hive/warehouse/base1.db/managed_t/add=beijing
/user/hive/warehouse/base1.db/managed_t/add=beijing/name.dat

#分别删除两个表的分区，再次查看HDFS上的数据是否还在

hive>alter table managed_t drop partition (add = 'beijing');
hive>alter table ex_table drop partition (add = 'beijing');

/user/hive/warehouse/base1.db/ex_table
/user/hive/warehouse/base1.db/ex_table/add=beijing
/user/hive/warehouse/base1.db/ex_table/add=beijing/name.dat
/user/hive/warehouse/base1.db/managed_t
```
从结果看出，管理表的数据已经被删除，只剩下表名（因为删除的是分区，并不是表），但是外部表的数据依然存在。

## 8 动态分区表
当一个分区表创建之后，其分区的值是可以动态修改的（先创建分区值，再插入数据；或者是在插入数据的时候指定一个新的分区值），这两种方式都是需要手动的去指定分区值。
当分区变的非常多的时候（比如气象站的气温记录数据，根据年份分区之后，还有根据月份分区，下面可能还有根据日期分区），当要上传数据到这样的表中的时候，手动去指定分区显然是不现实的。
这个时候，就需要使用到动态分区，动态分区可以在往表中插入数据的时候，动态的根据值来选择数据进入的分区。

### 8.1 动态分区使用场景
假设在HDFS上已经存在一个宽表（例如，职员表，这个表的字段非常多，并且数据量也很大），我们想要根据国家和省份把这些数据放到不同的分区中，使用动态分区是十分合适的。

### 8.2 创建动态分区表
1.首先创建一个分区表，指定两级分区，国家和省份
```hive
hive>create table emp(name string,age int) 
hive>partitioned by (country string,state string) 
hive>row format delimited 
hive>fields terminated by '\t' 
hive>lines terminated by '\n' 
hive>stored as textfile;
```
2.从宽表中查询出所需要的字段加载到分区表中
```hive
hive>insert into table emp partition(country,state) select name,age,country,state from t1;
```
**注意**：如果出现如下错误

```hive
FAILED: SemanticException [Error 10096]: 
Dynamic partition strict mode requires at least one static partition column. 
To turn this off set hive.exec.dynamic.partition.mode=nonstrict
```
严格模式下，不允许所有的分区都被动态指定，目的是为了防止生成太多的目录.通过下面语句可以解决问题:
```hive
hive>set hive.exec.dynamic.partition.mode=nonstrict;
```

这里就是动态分区的关键：
在指定分区的时候，不指定其值，而只是指定分区的名称，在后面的查询语句中，所查询的最后两个字段就会被对应到这两个分区上，也就是说，前面分区不指定值的话，就会到后面的查询语句中去动态的寻找值，这时可以想象，后面的查询语句的字段必须是大于等于前面的分区数；

### 8.3 半动态分区表

所谓半动态分区就是并不是所有的分区值都是动态指定的，其中有一部分是固定值，另一部分需要在查询列中动态赋值，例如：
```hive
hive>insert into table emp partition(country = 'US',state) 
hive>select name,age,country,state from t1;
```
上述语句中，分区country就是一个固定的值US，state的值没有指定，而是动态赋值，半分区模式需要注意的是，动态分区必须放在最后。