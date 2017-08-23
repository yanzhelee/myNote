# GROUPING SETS

该关键字可以实现同一数据集的多重group by操作。事实上GROUPING SETS是多个GROUP BY进行UNION ALL操作的简单表达，它仅仅使用一个stage完成这些操作。GROUPING SETS的子句中如果包含()数据集，则表示整体聚合。

示例：
```
select name, work_space[0] as main_place, count(employee_id) as emp_id_cnt
from employee
group by name, work_space[0]
GROUPING SETS((name,work_space[0]), name, ());

// 上面语句与下面语句等效

select name, work_space[0] as main_place, count(employee_id) as emp_id_cnt
from employee
group by name, work_space[0]
UNION ALL
select name, work_space[0] as main_place, count(employee_id) as emp_id_cnt
from employee
group by name
UNION ALL
select name, work_space[0] as main_place, count(employee_id) as emp_id_cnt
from employee;
```

# ROLLUP

扩展了GROUTING SETS。

```
select a, b, c from table group by a, b, c WITH ROLLUP;
// 等价于下面语句
select a, b, c from table group by a, b, c
GROUPING SETS((a,b,c),(a,b),(a),());
```
# CUBE

扩展了GROUTING SETS，对各种条件进行聚合。
```
select a, b, c from table group by a, b, c WITH ROLLUP;
// 等价于下面语句
select a, b, c from table group by a, b, c
GROUPING SETS((a,b,c),(a,b),(a,c),(b,c),(a),(b),(c),());
```

# 聚合条件 HAVING

having用于在组内进行过滤。
```
select cid,max(price) mx from orders group by cid having mx  > 1000;
//等价于下面的子查询语句
select t.cid, t.mx from (
        select cid, max(price) mx from orders group by cid
    ) t
where t.mx > 1000;
```
