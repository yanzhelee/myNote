# 《Hive编程指南》笔记一

1、hive不支持行级插入操作、更新操作和删除操作。hive不支持事务。

----------

2、用户还可以为数据库增加一些相关的键-值对属性信息，

```
create database test
with dbproperties('creator'='Mark','date'='2012-01-02');
#通过下面语句查看描述信息
describe database extended test;

test hdfs://master-server/user/hive/warehouse/test.db
{date=2012-01-02,creator=Mark}
```
----------

3、默认情况下，hive是不允许用户删除一个包含有表的数据库的，用户要么先删除数据库中的表，然后再删除数据库，要么在删除命令的最后加上关键字cascade，这样可以使hive自行删除数据库中的表。
`DROP DATABASE IF EXISTS testdatabase CASCADE;`

----------

4、`alter database`命令只能修改某个数据库的dbproperties信息，不能修改数据库名和数据库所在的目录位置。

----------

5、拷贝一张已经存在的表模式
```
create external table testtable1 like testable;
```
**注意：**如果语句中省略external关键字而且原表是外部表的话，那么生成的新表也是外部表。如果语句中省略external关键字二期源表是管理表的话，那么生成的新表也是管理表。但是如果语句中包含eternal关键字而且源表是管理表的话，那么生成的新表僵尸外部表。

----------

6、即使不知用use databasexxx也可以列举出指定数据库中的表：
`show tables in mydb;`

----------

7、自定义表的存储格式**没搞懂。。。。**

----------

8、ALTER TABLE

alter table仅仅会修改表的元数据，表数据本身不会有任何修改。需要用户自己确认所有的修改都是和真实的数据是一致的。

- 表重命名
`ALTER TABLE testtable RENAME TO testtable1;`

- 增加、修改和删除表分区

```
ALTER TABLE testtable ADD IF NOT EXISTS
PARTITION(year = 2011,month = 1) LOCATION '/log/2011/01'
PARTITION(year = 2011,month = 2) LOCATION '/log/2011/02'
...;
#添加多芬区语句在hive v0.8.0和之后的版本支持，低版本不支持
```

- 修改列信息

```
ALTER TABLE testtable 
CHANGE COLUMN hms hour_minutes_seconds INT
COMMENT 'The hours, minutes, and seconds'
```

- 增加列

```
ALTER TABLE testtable ADD COLUMNS(
col1 STRING,
col2 INT
...
);
```

- 修改表属性

```
ALTER TABLE testtable SET TBLPROPERTIES(
'note' = 'hello world'
);
```

----------

9、向管理表中装载数据

```
LOAD DATA LOCAL INPATH '/path/.../'
OVERWRITE INTO TABLE  testtable
PARTITION (country='US');
```

若指定OVERWRITE关键字，那么目标文件夹中之前存在的数据将会删除。如果没有指定OVERWRITE关键字，而目标文件夹下已经有同名的文件时，会保留之前的文件并且会重命名新文件为“之前的文件名_序列号”。
> **注：**hivev0.9.0版本之前的版本中存在bug：如果没有使用overwrite，目标文件夹中已经存在和装载文件同名的文件的话，之前的文件会被覆盖重写。这样数据会丢失。这个bug在0.9.0版本中已经修复了。


----------

10、通过查询语句向表中插入数据

```
INSERT OVERWRITE(INTO) TALBE employees
PARTITION(country='US',state='OR')
SELECT * FROM staged_employees se
WHERE se.cnty='US' AND se.st='OR';
```

**说明:**
INSERT INTO 是简单的添加数据
INSERT OVERWRITE 是删除原有数据然后再新增数据，如果有分区，那么指定的分区数据将会删除，其他分区不受影响。

----------

11、动态分区属性说明
动态分区功能默认 情况下是没有开启的。开启后，默认是以“严格”模式执行的，在这种模式下要求至少有哦一列分区字段是静态的。这有助于阻止因设计错误导致查询中产生大量的分区。
动态分区属性

属性名称 | 缺省值 | 描述
--------|-------|---------
hvie.exec.dynamic.partition | false | 设置成true开启动态分区功能
hive.exec.dynamic.partition.mode | strict | 设置成nonstrict，表示允许所有分区都是动态的
hive.exec.max.dynamic.partitions.pernode | 100 | 每个mapper或reducer可以创建的最大动态分区个数。如果某个mapper会reducer尝试创建大于这个数的分区的话则会抛出一个致命的错误信息
hive.exec.dynamic.partitions | +1000 | 一个动态分区创建语句可以创建的最大分区个数。如果超过这个值则会抛出致命错误
hive.exec.max.created.files | 100000 | 全局可以创建的最大文件个数。有一个hadoop计数器会跟踪记录创建了多少个文件，如果超过这个值会抛出致命错误。


----------

12、单个查询语句中创建表并加载数据

```
CREATE TABLE testtable
AS SELECT name, age, sex 
FROM people;
```

这个功能不能用于外部表。可以回想使用ALTER TABLE语句可以为外部表“引用”到一个分区，这里本身没有进行数据“装载”，而是将元数据中指定一个指向数据路径。

----------

13、数据导出

两种方式：一种是通过hdfs命令进行数据的导出，另一种是使用INSERT...DIRECTORY...

```
INSERT OVERWRITE LOCAL DIRECTORY '/tmp/test'
SELECT * FROM testtable;
```
这种方式下，不过在源表中数据实际是怎么存储的，hive会将所有的字段序列化成字符串写入到文件中。hive使用和hvie内部存储的表相同的编码方式来生成数据文件。


----------

14、表生成函数没看懂。。。。。

----------

15、CASE ... WHEN ... THEN句式

```
SELECT name
CASE 
	WHEN salary < 5000 THEN 'low'
	WHEN salary >= 5000 AND salary < 7000 THEN 'middle'
	WHEN salary >= 7000 AND salary < 10000 THEN 'high'
	ELSE 'verry high'
END
AS bracket FROM employees;
```

----------

16、hql语句中如果含有NULL，那么无论经过什么运算都为空
`nvl(columnname,0)`将columnname中为空的字段转换为0.

----------

