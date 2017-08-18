#《hive编程指南》笔记二

## 1 Group by 和having

**注意：**select后的字段，必须要么包含在group by中，或者使用聚合函数。

1 group by是分组查询，一般group by是和聚合函数配合使用
>group by有一个个原则，就是select后面的所有列中，没有使用聚合函数的列，必须出现在group by后面。

2 Having和where

>where字句的作用是在对查询结果进行分组前，将不符合where条件的行去掉，即在分组之前过滤数据，条件中不能有包含聚合函数，使用where条件显示特定的行。
>having字句的作用是筛选满足条件的组，即在分组之后过滤数据，条件中经常包含聚合函数，使用having条件显示特定的组，也可以使用多个分组标准进行分组。

----------

## 2 JOIN
Hive支持通常的sql join，但是**只支持等值连接，不支持非等值连接。**
### 2.1 INNER JOIN

内连接中，只有进行连接的两个表中都存在于连接标准相匹配的数据才会被保留下来。

```sql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
JOIN Orders
ON Persons.Id_P = Orders.Id_P
ORDER BY Persons.LastName
```

hive同时假定查询中最后的一个表时最大的那个表。在对每行记录进行连接操作是，他会尝试将其他表缓存起来，然后扫描最后那张表进行计算。因此，用户需要保证连续查询中的表的大小从左到右是一次增加的。

### 2.2 LEFT OUTER JOIN

在这种join连接操作中，join操作符左边表中符合where字句的所有记录将会被返回。join操作符右边的表中如果没有on后面的连接条件记录时，那么从右边表指定的选择列的值将会是NULL。

### 2.3 RIGHT OUTER JOIN

余左外连接相反。

### 2.4 FULL OUTER JOIN

将会返回所有表中符合where语句条件的所有记录。如果任意一张表的指定字段没有符合条件查询的值的话，那么就返回NULL值替代。

### 2.5 LEFT SEMI JOIN

左半开连接（LEFT SEMI-JOIN）会返回左边表的记录，前提是其记录对于右边表满足on语句中的判定条件。
```sql
select a.id, a.name from table1 a
left semi join table2 b
on (a.id=b.id)

--等价于：
select a.id, a.name from table1 a 
where a.id in (select id from table2);

--也等价于：
select a.id, a.name from table1 a
join table2 b
on(a.id = b.id);

--页等价于：
select a.id, a.name from table1 a
where exists (select 1 from table2 b where a.id = b.id);
```

### 2.6 笛卡尔积JOIN

笛卡尔积是一种连接，表示左边表的行数乘以右边表的行数等于笛卡尔积结果集的大小。

```sql
select a.id, a.name ,b.age 
from table1 a
cross join table2 b;
```

### 2.7 map-side JON

如果所有表中只有一张表时小表，那么可以在最大的表通过mapper的时候将小表完全放到内存中。hive可以在map端执行连接过程（称为map-side JOIN），这是因为hive可以和内存中的小表进行逐一匹配，从而省略掉常规连接操作所需要的reduce过程。