# hive内部表与外部表

## hive的内部表与外部表之间的区别

| 区别	| 创建表过程 | 删除表过程 |
|:-----|:----------|:--------|
| 内部表 |会将数据移动到数据仓库指向的路径|元数据和实际数据一起删除|
| 外部表 |仅记录数据所在的路径，不会对数据的位置坐任何改变|只删除元数据，不删除实际数据，相对比较安全。|

## 传统数据库和hive之间的区别

传统数据库对表的验证是schema on write（写时模式），而hive在load时是不检查数据是否是符合schema的，hive遵循的是schema on read（读时模式），只有在读的时候才检查、解析具体的数据字段、schema。

## 内部表概念
在hive中默认情况创建一个表的语句是：
```sql
hive>create table inner_table(name string) row format delimited
hive>fields terminated by '\t'
hive>lines terminated by '\n'
hive>stored as textfile;
```
这样创建的表是一种内部表，hdfs上会创建相应的文件夹，如果在其中上传数据的话，数据就会保存在对应的文件夹下，当删除这个表时，在此文件夹下的所有数据也会删除，内部表的特点之一就是删除表的时候会把对应的文件也一并删除。
大数据场景下，hive处理的数据往往是非常巨大的，特点是一次上传，多次使用，往往是多个人都会使用到，如果轻易被删除的话，是非常危险的，为了避免对数据的误删，出现了外部表。

## 外部表概念
在创建外部表的时候，指定external关键字，这样创建的表就是外部表。

### 创建方式一（不指定存放目录）
这是默认创建外部表，数据存放在默认的hdfs相应的目录下。
```sql
hive>create external table outer_talbe(name string) row format delimited
hive>fields terminated by '\t'
hive>lines terminated by '\n'
hive>stored as textfile;
```
### 创建方式二（指定存放目录）
```sql
hive>create external table outer_talbe(name string) row format hive>delimited
hive>fields terminated by '\t'
hive>lines terminated by '\n'
hive>stored as textfile
hive>location 'hdfs_folder';
```

## 演示外部表删除后，其数据是否会被删除
```sql
hive>create external table outter_table(name stirng);
hive>drop table outter_table;
hive>dfs -ls -R /;
```

**注意**
删除外部表后再次建立一个内部表，其表名与被删除的表名相同的话，它还是会被关联到hdfs上相同的路径下，这时如果再删除这个表的话，那么数据就会被删除，因为此时的表是一个内部变，数据会随着表的删除而删除。

## hive如何查看是外部表还是内部表

### 方式一
进入hive，执行```describe extended tablename;'''查看表的详细信息。
如果是外部表，在详细信息的最后一行，会输出tableType:EXTERNAL_TABLE。
如果是内部表，则会显示tableType：MANAGD_TABLE。

### 方式二
在hive中执行```desc formatted tablename;```可以查看表的格式和详细的信息，这里可以得到Table Type也可以得到表的location。根据Table Type可以知道表是内部表还是外部表。

## 总结
在团队开发中，在hdfs上存放重要数据时不要使用内部表，都是使用外部表关联数据。

## 参考博文

[http://blog.csdn.net/wisgood/article/details/17186139](http://blog.csdn.net/wisgood/article/details/17186139)

[http://blog.chinaunix.net/uid-77311-id-4586440.html](http://blog.chinaunix.net/uid-77311-id-4586440.html)

[http://blog.csdn.net/qq_31382921/article/details/53083201](http://blog.csdn.net/qq_31382921/article/details/53083201)
