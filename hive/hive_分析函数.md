# Hive分析函数和窗口函数

在Hive 0.11之后支持的，扫描多个输入的行计算每行的结果。通常和OVER,PARTITION BY, ORDER BY, WINDOWING配合使用。和传统的分组结果不一样，传统的结果每组中只有一个结果。分析函数的结果会出现多次，和每条记录都连接输出。

语法形式如下：
```
Function(arg1,....argn) OVER([PARTITION BY<...>] [ORDER BY<...>] [window_clause])
```
## 窗口函数

|        函数名         |                                                                  说明                                                                  |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| FIRST_VALUE           | 取出分组内排序后，截止到当前行，第一个值                                                                                               |
| LAST_VALUE            | 取出分组内排序后，截止到当前行，最后一个值                                                                                             |
| LEAD(col, n, DEFAULT) | 用于统计窗口内往下第n行的值。第一个参数为列名，第二个参数为往下第n行(可选，默认为1)，第三个参数为默认值(当往下第n行为NULL时，取默认值) |
| LAG(col,n,DEFAULT)    | 与lead相反，用于统计窗口内往下第n个值。第一个参数为列名，第二个参数为往上第n行(可选，默认为1)                                          |

## OVER从句

1. 使用标准的聚合函数COUNT,SUM,MIN,MAX,AVG
2. 使用PARTITION BY语句，使用一个或者多个原始数据类型的列
3. 使用PARTITION BY与ORDER BY语句，使用一个或者多个数据类型的分区或者拍序列
4. 使用窗口规范，窗口规范支持一下格式：

```
(ROW | RANGE) BETWEEN (UNBOUNDED | [num]) PRECEDING AND ([num] PRECEDING | CURRENT ROW | (UNBOUNDED | [num]) FOLLOWING)
(ROW | RANGE) BETWEEN CURRENT ROW AND (CURRENT ROW | (UNBOUNDED | [num]) FOLLOWING)
(ROW | RANGE) BETWEEN [num] PRECEDING AND (UNBOUNDED | [num]) FOLLOWING
```

当ORDER BY后面缺少窗口从句条件，窗口规范默认是
```
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

当ORDER BY和窗口从句都缺失，窗口规范默认是：
```
ROW BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
```

## 分析函数

|      函数      |                                                                                                           说明                                                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| ROW_NUMBER()   | 从1开始，按照顺序，生成分组内记录的序列，比如，按照pv降序排列，生成分组内每天的pv名次，ROW_NUMBER()的应用 场景非常多，再比如，获取分组内排序第一的记录，获取一个session中的第一条refer等                                 |
| RANK()         | 生成数据项在分组中的排名，排名相等会在名次中留下空位                                                                                                                                                                     |
| DENSE_RANK()   | 生成数据项在分组中的排名，排名相等会在名次中不会留下空位                                                                                                                                                                 |
| CUME_DIST()    | 小于等于当前值的行数除以分组内总行数。比如，统计小于等于当前薪水的人数所占总人数的比例                                                                                                                                   |
| PERCENT_RANK() | 分组内当前行的RANK值-1/分组内 总行数-1                                                                                                                                                                                   |
| NTILE(n)       | 用于将分组数据按照顺序切分成n片，返回当前切片值，如果切片不均匀，默认增加第一个切片的分布。NTILE不支持ROWS BETWEEN,比如NTILE(2) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PERCEDING AND CURRENT ROW) |

### Hive2.1.0及以后版本支持distinct

在聚合函数(sum, count, avg)中支持distinct，但是在order by或者 窗口限制中不支持。
```conut(distinct a) over(partition by c)```

### Hive2.1.0以后支持在OVER从句中支持聚合函数

```
select rank() over(order by sum(b))
```

### Hive2.2.0中在使用ORDER BY和窗口限制时支持distinct

```count(distinct a) over (partition by c order by d rows between 1 preceding and 1 following)```

## 通过实例深入理解窗口函数和分析函数

### COUNT、SUM、MIN、MAX、AVG案例分析

--------------------------------------

```
## 创建数据表
create table orders(
    user_id string,
    device_id string,
    user_type string,
    price float,
    sales int);

## 添加数据orders.txt
zhangsa test1   new     67.1    2
lisi    test2   old     43.32   1
wanger  test3   new     88.88   3
liliu   test4   new     66.0    1
tom     test5   new     54.32   1
tomas   test6   old     77.77   2
tomson  test7   old     88.44   3
tom1    test8   new     56.55   6
tom2    test9   new     88.88   5
tom3    test10  new     66.66   5

## 开窗函数案例
select
    user_id,
    user_type,
    sales,
    -- 默认从起点到当前所有重复行
    sum(sales) over(partition by user_type order by sales asc) as sales_1,
    -- 从起点到当前所有重复行与sales_1结果相同
    sum(sales) over(partition by user_type order by sales asc range between unbounded preceding and current row) as sales_2,
    -- 从起点到当前行，结果与sale_1结果不同
    sum(sales) over(partition by user_type order by sales asc rows between unbounded preceding and current row) as sales_3,
    -- 当前行加上往前3行
    sum(sales) over(partition by user_type order by sales asc rows between 3 preceding and current row) as sales_4,
    -- 当前范围往上加3行
    sum(sales) over(partition by user_type order by sales asc range between 3 preceding and current row) as sales_5,
    -- 当前行+往前3行+往后1行
    sum(sales) over(partition by user_type order by sales asc rows between 3 preceding and 1 following) as sales_6,
    --
    sum(sales) over(partition by user_type order by sales asc range between 3 preceding and 1 following) as sales_7,
    -- 当前行+之后所有行
    sum(sales) over(partition by user_type order by sales asc rows between current row and unbounded following) as sales_8,
    --
    sum(sales) over(partition by user_type order by sales asc range between current row and unbounded following) as sales_9,
    -- 分组内所有行
    sum(sales) over(partition by user_type) as sales_10
from
    orders
order by
    user_type,
    sales,
    user_id;

##上述查询结果如下：

| user_id  | user_type  | sales  | sales_1  | sales_2  | sales_3  | sales_4  | sales_5  | sales_6  | sales_7  | sales_8  | sales_9  | sales_10  |
|----------|------------|--------|----------|----------|----------|----------|----------|----------|----------|----------|----------|-----------|
| liliu    | new        | 1      | 2        | 2        | 2        | 2        | 2        | 4        | 4        | 22       | 23       | 23        |
| tom      | new        | 1      | 2        | 2        | 1        | 1        | 2        | 2        | 4        | 23       | 23       | 23        |
| zhangsa  | new        | 2      | 4        | 4        | 4        | 4        | 4        | 7        | 7        | 21       | 21       | 23        |
| wanger   | new        | 3      | 7        | 7        | 7        | 7        | 7        | 12       | 7        | 19       | 19       | 23        |
| tom2     | new        | 5      | 17       | 17       | 17       | 15       | 15       | 21       | 21       | 11       | 16       | 23        |
| tom3     | new        | 5      | 17       | 17       | 12       | 11       | 15       | 16       | 21       | 16       | 16       | 23        |
| tom1     | new        | 6      | 23       | 23       | 23       | 19       | 19       | 19       | 19       | 6        | 6        | 23        |
| lisi     | old        | 1      | 1        | 1        | 1        | 1        | 1        | 3        | 3        | 6        | 6        | 6         |
| tomas    | old        | 2      | 3        | 3        | 3        | 3        | 3        | 6        | 6        | 5        | 5        | 6         |
| tomson   | old        | 3      | 6        | 6        | 6        | 6        | 6        | 6        | 6        | 3        | 3        | 6         |
```



**注意**
1. 结果和ORDER BY相关,默认为升序
2. 如果不指定ROWS BETWEEN,默认为从起点到当前行;
3. 如果不指定ORDER BY，则将分组内所有值累加;
4. PRECEDING：往前
5. FOLLOWING：往后
6. CURRENT ROW：当前行
7. UNBOUNDED：无界限（起点或终点）
8. UNBOUNDED PRECEDING：表示从前面的起点
9. UNBOUNDED FOLLOWING：表示到后面的终点
10. 其他COUNT、AVG，MIN，MAX，和SUM用法一样。

### FIRST_VALUE和LAST_VALUE案例分析

----------------------

```
select
    user_id,
    user_type,
    sales,
    ROW_NUMBER() OVER(PARTITION BY user_type ORDER BY sales) AS row_num,
    first_value(user_id) over (partition by user_type order by sales desc) as max_sales_user,
    first_value(user_id) over (partition by user_type order by sales asc) as min_sales_user,
    last_value(user_id) over (partition by user_type order by sales desc) as curr_last_min_user,
    last_value(user_id) over (partition by user_type order by sales asc) as curr_last_max_user
from
    orders
order by
    user_type,
    sales;

##上述查询结果如下:

| user_id | user_type | sales | row_num | max_sales_user | min_sales_user | curr_last_min_user | curr_last_max_user |
| ------- | --------- | ----- | ------- | -------------- | -------------- | ------------------ | ------------------ |
| tom     | new       | 1     | 1       | tom1           | tom            | tom                | liliu              |
| liliu   | new       | 1     | 2       | tom1           | tom            | tom                | liliu              |
| zhangsa | new       | 2     | 3       | tom1           | tom            | zhangsa            | zhangsa            |
| wanger  | new       | 3     | 4       | tom1           | tom            | wanger             | wanger             |
| tom3    | new       | 5     | 5       | tom1           | tom            | tom3               | tom2               |
| tom2    | new       | 5     | 6       | tom1           | tom            | tom3               | tom2               |
| tom1    | new       | 6     | 7       | tom1           | tom            | tom1               | tom1               |
| lisi    | old       | 1     | 1       | tomson         | lisi           | lisi               | lisi               |
| tomas   | old       | 2     | 2       | tomson         | lisi           | tomas              | tomas              |
| tomson  | old       | 3     | 3       | tomson         | lisi           | tomson             | tomson             |
```



### LEAD与LAG

-----------------

```
select
    user_id,
    device_id,
    sales,
    ROW_NUMBER() OVER(ORDER BY sales) AS row_num,
    lead(device_id) over (order by sales) as default_after_one_line,
    lag(device_id) over (order by sales) as default_before_one_line,
    lead(device_id,2) over (order by sales) as after_two_line,
    lag(device_id,2,'abc') over (order by sales) as before_two_line
from
    orders
order by
    sales;

上述查询结果如下

| user_id  | device_id  | sales  | row_num  | default_after_one_line  | default_before_one_line  | after_two_line  | before_two_line  |
|----------|------------|--------|----------|-------------------------|--------------------------|-----------------|------------------|
| lisi     | test2      | 1      | 3        | test6                   | test4                    | test1           | test5            |
| liliu    | test4      | 1      | 2        | test2                   | test5                    | test6           | abc              |
| tom      | test5      | 1      | 1        | test4                   | NULL                     | test2           | abc              |
| zhangsa  | test1      | 2      | 5        | test7                   | test6                    | test3           | test2            |
| tomas    | test6      | 2      | 4        | test1                   | test2                    | test7           | test4            |
| wanger   | test3      | 3      | 7        | test10                  | test7                    | test9           | test1            |
| tomson   | test7      | 3      | 6        | test3                   | test1                    | test10          | test6            |
| tom2     | test9      | 5      | 9        | test8                   | test10                   | NULL            | test3            |
| tom3     | test10     | 5      | 8        | test9                   | test3                    | test8           | test7            |
| tom1     | test8      | 6      | 10       | NULL                    | test9                    | NULL            | test10           |
```


### RANK、ROW_NUMBER、DENSE_RANK

--------------------------------------

```
select
user_id,user_type,sales,
RANK() over (partition by user_type order by sales desc) as r,
ROW_NUMBER() over (partition by user_type order by sales desc) as rn,
DENSE_RANK() over (partition by user_type order by sales desc) as dr
from
orders;

##上述查询结果如下

| user_id | user_type | sales |  r  | rn  | dr  |
| ------- | --------- | ----- | --- | --- | --- |
| tom1    | new       | 6     | 1   | 1   | 1   |
| tom3    | new       | 5     | 2   | 2   | 2   |
| tom2    | new       | 5     | 2   | 3   | 2   |
| wanger  | new       | 3     | 4   | 4   | 3   |
| zhangsa | new       | 2     | 5   | 5   | 4   |
| tom     | new       | 1     | 6   | 6   | 5   |
| liliu   | new       | 1     | 6   | 7   | 5   |
| tomson  | old       | 3     | 1   | 1   | 1   |
| tomas   | old       | 2     | 2   | 2   | 2   |
| lisi    | old       | 1     | 3   | 3   | 3   |
```


### NTILE

-----------

```
select
    user_type,sales,
    --分组内将数据分成2片
    NTILE(2) OVER(PARTITION BY user_type ORDER BY sales) AS nt2,
    --分组内将数据分成3片
    NTILE(3) OVER(PARTITION BY user_type ORDER BY sales) AS nt3,
    --分组内将数据分成4片
    NTILE(4) OVER(PARTITION BY user_type ORDER BY sales) AS nt4,
    --将所有数据分成4片
    NTILE(4) OVER(ORDER BY sales) AS all_nt4
from
    orders
order by
    user_type,
    sales;

##上述查询结果如下
| user_type | sales | nt2 | nt3 | nt4 | all_nt4 |
| --------- | ----- | --- | --- | --- | ------- |
| new       | 1     | 1   | 1   | 1   | 1       |
| new       | 1     | 1   | 1   | 1   | 1       |
| new       | 2     | 1   | 1   | 2   | 2       |
| new       | 3     | 1   | 2   | 2   | 3       |
| new       | 5     | 2   | 2   | 3   | 4       |
| new       | 5     | 2   | 3   | 3   | 3       |
| new       | 6     | 2   | 3   | 4   | 4       |
| old       | 1     | 1   | 1   | 1   | 1       |
| old       | 2     | 1   | 2   | 2   | 2       |
| old       | 3     | 2   | 3   | 3   | 2       |
```


**求取sale前20%的用户ID**

```
select
    user_id
from
(
    select
        user_id,
        NTILE(5) OVER(ORDER BY sales desc) AS nt
    from
        orders
)A
where nt=1;
## 结果如下
+----------+
| user_id  |
+----------+
| tom1     |
| tom3     |
+----------+
```

### CUME_DIST、PERCENT_RANK

---------------------------------

```
select
    user_id,user_type,sales,
    --没有partition,所有数据均为1组
    CUME_DIST() OVER(ORDER BY sales) AS cd1,
    --按照user_type进行分组
    CUME_DIST() OVER(PARTITION BY user_type ORDER BY sales) AS cd2
from
    orders;

## 上述结果如下
+----------+------------+--------+------+----------------------+--+
| user_id  | user_type  | sales  | cd1  |         cd2          |
+----------+------------+--------+------+----------------------+--+
| liliu    | new        | 1      | 0.3  | 0.2857142857142857   |
| tom      | new        | 1      | 0.3  | 0.2857142857142857   |
| zhangsa  | new        | 2      | 0.5  | 0.42857142857142855  |
| wanger   | new        | 3      | 0.7  | 0.5714285714285714   |
| tom2     | new        | 5      | 0.9  | 0.8571428571428571   |
| tom3     | new        | 5      | 0.9  | 0.8571428571428571   |
| tom1     | new        | 6      | 1.0  | 1.0                  |
| lisi     | old        | 1      | 0.3  | 0.3333333333333333   |
| tomas    | old        | 2      | 0.5  | 0.6666666666666666   |
| tomson   | old        | 3      | 0.7  | 1.0                  |
+----------+------------+--------+------+----------------------+--+

select
    user_type,sales,
    --分组内总行数
    SUM(1) OVER(PARTITION BY user_type) AS s,
    --RANK值
    RANK() OVER(ORDER BY sales) AS r,
    PERCENT_RANK() OVER(ORDER BY sales) AS pr,
    --分组内
    PERCENT_RANK() OVER(PARTITION BY user_type ORDER BY sales) AS prg
from
    orders;

## 上述结果如下
+------------+--------+----+-----+---------------------+---------------------+--+
| user_type  | sales  | s  |  r  |         pr          |         prg         |
+------------+--------+----+-----+---------------------+---------------------+--+
| new        | 1      | 7  | 1   | 0.0                 | 0.0                 |
| new        | 1      | 7  | 1   | 0.0                 | 0.0                 |
| new        | 2      | 7  | 4   | 0.3333333333333333  | 0.3333333333333333  |
| new        | 3      | 7  | 6   | 0.5555555555555556  | 0.5                 |
| new        | 5      | 7  | 8   | 0.7777777777777778  | 0.6666666666666666  |
| new        | 5      | 7  | 8   | 0.7777777777777778  | 0.6666666666666666  |
| new        | 6      | 7  | 10  | 1.0                 | 1.0                 |
| old        | 1      | 3  | 1   | 0.0                 | 0.0                 |
| old        | 2      | 3  | 4   | 0.3333333333333333  | 0.5                 |
| old        | 3      | 3  | 6   | 0.5555555555555556  | 1.0                 |
+------------+--------+----+-----+---------------------+---------------------+--+

```

## 参考博文

[http://blog.csdn.net/scgaliguodong123_/article/details/60135385](http://blog.csdn.net/scgaliguodong123_/article/details/60135385)
