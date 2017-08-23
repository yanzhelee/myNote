# HIVE 排序总结

## ORDER BY

Hive中的order by语句用于对查询结果集执行一个**全局排序**。这也就是说会有一个所有的数据都通过特格reducer进行处理的过程。对于大数据集，这个过程可能消耗太过漫长的时间来执行。

使用语法
```
select * from table order by id desc limit 5;
```
在严格模式下必须使用limit限定条件，因为如果数据量特别大的话会出现无法输出结果的情况，如果惊醒limit n限定，那么只有 `(n * map number)`条记录进行处理。

设置hive MapReduce模式

```
set hive.mapred.mode = nonstrict;   // 设置非严格模式，默认情况下就是非严格模式
set hive.mapred.mode = strict ;     // 设置为严格模式
```

## SORT BY

sort by**不是全局排序**，其只会在每个reduce中对数据进行排序，也就是执行一个局部排序过程。这可以保证每个reduce的输出数据都是有序的（但并非全局有序）。这样可以提高后面进行的全局排序的效率。
sort by不受hive.mapred.mode是否为strict的影响。
使用sort by你可以通过指定执行的reduce个数(set mapred.reduce.tasks=<number>)对输出的数据再执行归并排序，既可以得到全部结果。

使用语法

```
select s.ymd, s.ymbol, s.price from stocks s sort by s.ymd ASC, s.symbol DESC;
```
总结如果设置的reduce个数为1的话那么sort by 语句和order by语句输出的结果就一样。

## DISTRIBUTE BY

DISTRIBUTE BY控制map的输出在reducer中是如何划分的。MapReduce job中传输的所有数据都是 按照key/value对的形式进行组织的，因此Hive在将用户的查询语句转换成MapReduce job时，其必须在内部使用这个功能。

默认情况下，MapRecude计算框架会依据map输入的键计算相应的哈希值，然后按照得到的hash值将内容分发到多个reduce中去，不过当使用sort by时，不同的reducer的输出会有明显的重叠，至少对于排列顺序而言是这样的，即使每个reducer的输出的数据都是有序的。

加入我们希望具有相同的股票交易码的数据在一起处理，那么我们可以使用DISTRIBUTE BY来保证具有相同股票交易码的记录会分发 到同一个reducer中进行处理，然后使用sort by来按照 我们的期望对数据进行排序。

使用方法
```
select s.ymd, s.ymbol, s.price from stocks s DISTRIBUTE BY s.ymbol sort by s.symbol ASC;
```

**注意**：Hive要求DISTRIBUTE BY 语句要写在sort by语句之前。

## CLUSTER BY

CLUSTER BY的功能就是DISTRIBUTE BY 和SORT BY相结合，如下语句等效：
```
select cid , price from orders DISTRIBUTE BY cid SORT BY cid ;
select cid , price from orders CLUSTER BY cid ;
```
**注意**：被cluster by指定列只能呢按照降序进行排列，不能指定asc和desc。
