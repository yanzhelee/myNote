# hive桶表

## 1 桶表的概念

在hive中，数据库、表、分区都是对应到hdfs上的路径，当往表中上传数据的时候，数据会传到对应的路径下，形成新的文件，文件名的格式类似为00000_0...每次插入文件都会形成新的文件，命名也是有规律的，桶表就是对应不同的文件的。
hive中有桶的概念，对于每一个表或者分区来说，可以进一步组织成桶，其实就是更细粒度的数据范围。
hive采用列值哈希，然后除以桶的个数以求余数的方式确定该条记录是存放在那个表中。

**公式：whichBucket = hash(columnValue) % numberOfBuckets**

hive桶表最大限度的保证了每个桶中的文件中的数据量大致相同，不会造成数据倾斜。

**总结**:桶表就是对一次进入表的数据进行文件级别的划分。

## 2 使用桶表的好处

1. 获得更高的查询处理效率，桶表加上额外的结构，hivee在处理有些查询的时候能利用这个结构。具体而言，连接两个在（包含连接列的）相同列上划分桶的表，可以使用map端连接（map-side join）高效的实现。比如join操作。对于join操作两个表有一个相同的列，如果对这两个表都进行桶的操作。那么僵保存相同列值得桶进行join操作就可以了。可以大大尖山join的数据量。
2. 使取样（sampling）更高效；在处理大规模数据集时，在开发和修改查询阶段，如果能在数据集的一小部分数据上试运行查询，会带来很多方便。

**注意** clustered by 和sorted by不会影响数据的导入，这意味着，用户必须自己负责数据是如何导入的，包括数据的分桶和排序。

桶表通常是和抽样联合使用的，桶表可以使数据分散存放，这是对每个文件进行抽样的话，就极大的保证了抽样的均衡性。如果数据姓谢的话就会导致抽样的不均匀。

## 3 创建桶表的语法

```
create table emp(id int, name string) 
CLUSTERED BY (id) INTO 2 BUCKETS 
row format delimited 
fields terminated by '\t'
lines terminated by '\n'
stored as textfile;
```

clustered by 后面加的列一定是在表中存在的列，后面接的是桶的个数，2意味着一次上传数据会根据id的hash值再与2取模，根据这个值决定这条数据落入那个文件中。

```
hive> desc formatted emp;
OK
# col_name              data_type               comment

id                      int
name                    string

# Detailed Table Information
Database:               test
Owner:                  yanzhelee
CreateTime:             Sun Jul 23 08:15:16 PDT 2017
LastAccessTime:         UNKNOWN
Retention:              0
Location:               hdfs://s200/user/hive/warehouse/test.db/emp
Table Type:             MANAGED_TABLE
Table Parameters:
        COLUMN_STATS_ACCURATE   {\"BASIC_STATS\":\"true\"}
        numFiles                0
        numRows                 0
        rawDataSize             0
        totalSize               0
        transient_lastDdlTime   1500822916

# Storage Information
SerDe Library:          org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe
InputFormat:            org.apache.hadoop.mapred.TextInputFormat
OutputFormat:           org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat
Compressed:             No
Num Buckets:            2
Bucket Columns:         [id]
Sort Columns:           []
Storage Desc Params:
        field.delim             ,
        line.delim              \n
        serialization.format    ,
Time taken: 0.157 seconds, Fetched: 33 row(s)

```
准备一个原始表src_emp,其中的字段也是id和name，里面的内容随意，id递增。


```
1,tom
2,toms
3,jerry
4,bob
5,tomas
```

## 4 插入数据

桶的数量意味着产生文件的数量，那么两个桶就应该使用2个reduce任务来完成，但是默认情况下hive至启动一个reducer，所以要修改reducer的数量，可以通过设置强制分桶机制来保证reducer数量和桶的数量一致。
`set hive.enforce.bucketing = true;`
这个一定要改成true，hive就会根据桶的数量启动reducer数量。

**注意:**参数在设置的时候一定不能写错，hive是不提示错误的。

然后将这个表中的数据查询出来插入到桶表中。
`insert into emp select * from src_emp;`
从启动的作业信息来看，reducer的数量被改成2，这样的结果 是会产生和桶数相同的文件数量。

## 5 思考

当使用了强制分桶的参数后，如果一次插入的数据量很少，那么会不会生成和桶数相同数量的文件呢？
会的，强制分桶就是强制产生桶文件，不论一次插入的数量是多少，可能会有空的文件产生。

## 参考博文
[http://www.cnblogs.com/wujin/p/6093401.html](http://www.cnblogs.com/wujin/p/6093401.html)