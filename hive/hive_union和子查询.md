# UNION的使用

union用于联合多个select语句的结果集，合并为一个独立的结果集。当前只支持UNION ALL(bag union)。不能消除重复行，每个select语句返回的列的数量和名字必须一样，否则会抛出语法错误。

```
select_statement UNION ALL select_statement UNION ALL select_statement.....
```
如果必须对union的结果集做一些额外的处理，整个语句可以被嵌入在from子句中。
```
select * from from(
    select_statement
        UNION ALL
    select_statement
    ) unionResult
```

# Hive子查询

子查询语法
```
select .... from (subquery) name  ...
```
Hive只在from子句中支持子查询。子查询必须给定一个名字，因为每个表在from子句中必须有一个名字。子查询的查询列表的列，必须有唯一的名字。子查询的查询列表在外面的查询是可用的，就向表的列。子查询也可以一个UNION查询表达式，Hive支持任意层次的子查询。

示例1：
```
select col from (
    select a+b as col from t1
    ) t2;
```

示例2：
```
select t3.col from(
    select a+b as col from t1
      UNION ALL
    select c+d as col from t2
    ) as t3;
```
