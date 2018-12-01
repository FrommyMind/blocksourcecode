---
title: Hive窗口函数和分析函数
date: 2018-05-13 11:53:26
tags: 
  - Hive
  - 函数
---
# Hive窗口函数和分析函数

## 创建测试表并插入测试数据

```
create table test_analy_funs (
stu_id int
,stu_nm varchar(20)
,class int
,course varchar(20)
,score decimal(6,2));

insert into table test_analy_funs
select 1,'Daniel',201,'maths',85 union all
select 2,'Tom',201,'maths',82 union all
select 3,'Macal',201,'maths',26 union all
select 4,'Lily',201,'maths',99 union all
select 5,'Onel',201,'maths',100 union all
select 6,'Kebe',201,'maths',67 union all
select 7,'Kaka',201,'maths',89 union all
select 8,'Piye',201,'maths',93 union all
select 9,'Luo',202,'maths',73 union all
select 10,'Lufi', 202,'maths',98 union all
select 11,'Tony', 202,'maths',100 union all
select 12,'Miya', 202,'maths',100 union all
select 13,'Joy', 202,'maths',87 union all
select 14,'Tong', 202,'maths',79 union all
select 15,'Roy', 202,'maths',84 union all
select 16,'Erw', 202,'maths',84 union all
select 1,'Daniel',201,'english',82 union all
select 2,'Tom',201,'english',26 union all
select 3,'Macal',201,'english',99 union all
select 4,'Lily',201,'english',100 union all
select 5,'Onel',201,'english',77 union all
select 6,'Kebe',201,'english',89 union all
select 7,'Kaka',201,'english',77 union all
select 8,'Piye',201,'english',85 union all
select 9,'Luo',202,'english',80 union all
select 10,'Lufi', 202,'english',100 union all
select 11,'Tony', 202,'english',59 union all
select 12,'Miya', 202,'english',84 union all
select 13,'Joy', 202,'english',82 union all
select 14,'Tong', 202,'english',89 union all
select 15,'Roy', 202,'english',90 union all
select 16,'Erw', 202,'english',95 union all
select 1,'Daniel',201,'chinese',28 union all
select 2,'Tom',201,'chinese',99 union all
select 3,'Macal',201,'chinese',89 union all
select 4,'Lily',201,'chinese',100 union all
select 5,'Onel',201,'chinese',88 union all
select 6,'Kebe',201,'chinese',83 union all
select 7,'Kaka',201,'chinese',92 union all
select 8,'Piye',201,'chinese',95 union all
select 9,'Luo',202,'chinese',82 union all
select 10,'Lufi', 202,'chinese',92 union all
select 11,'Tony', 202,'chinese',99 union all
select 12,'Miya', 202,'chinese',78 union all
select 13,'Joy', 202,'chinese',28 union all
select 14,'Tong', 202,'chinese',50 union all
select 15,'Roy', 202,'chinese',30 union all
select 16,'Erw', 202,'chinese',88 



```

<!-- more -->
## 排序

dense_rank() OVER([partition_by_clause] order_by_clause)
返回从1开始的升序序列，根据不同的order by的字段生成不同的值，如果值一样则排序结果也一样。
Returns an ascending sequence of integers, starting with 1. The output sequence produces duplicate integers for duplicate values of the ORDER BY expressions.

rank() OVER([partition_by_clause] order_by_clause)
返回从1开始的升序序列，根据不同的order by的字段生成不同的值，如果值一样则排序结果也一样。然后序列值会根据实际的数值增加。
Returns an ascending sequence of integers, starting with 1. The output sequence produces duplicate integers for duplicate values of the ORDER BY expressions. After generating duplicate output values for the "tied" input values, the function increments the sequence by the number of tied values.

row_number() OVER([partition_by_clause] order_by_clause)
返回从1开始的升序序列，根据不同的order by的字段生成不同的值，即使值一样，排序结果也不一样。
Returns an ascending sequence of integers, starting with 1. Starts the sequence over for each group produced by the PARTITIONED BY clause. The output sequence includes different values for duplicate input values. Therefore, the sequence never contains any duplicates or gaps, regardless of duplicate input values.

```
select
   stu_nm, 
   class,
   course,
   score,
   rank() over(partition by class, course order by score desc) rk1,
   dense_rank() over(partition by class, course order by score desc) rk2,
   row_number() over(partition by class, course order by score desc) rk3,
   percent_rank() over(partition by class, course order by score desc)
from test_analy_funs;

+---------+--------+----------+--------+------+------+------+----------------------+--+
| stu_nm  | class  |  course  | score  | rk1  | rk2  | rk3  |        _wcol3        |
+---------+--------+----------+--------+------+------+------+----------------------+--+
| Lily    | 201    | english  | 100    | 1    | 1    | 1    | 0.0                  |
| Macal   | 201    | english  | 99     | 2    | 2    | 2    | 0.14285714285714285  |
| Kebe    | 201    | english  | 89     | 3    | 3    | 3    | 0.2857142857142857   |
| Piye    | 201    | english  | 85     | 4    | 4    | 4    | 0.42857142857142855  |
| Daniel  | 201    | english  | 82     | 5    | 5    | 5    | 0.5714285714285714   |
| Onel    | 201    | english  | 77     | 6    | 6    | 6    | 0.7142857142857143   |
| Kaka    | 201    | english  | 77     | 6    | 6    | 7    | 0.7142857142857143   |
| Tom     | 201    | english  | 26     | 8    | 7    | 8    | 1.0                  |

| Tony    | 202    | maths    | 100    | 1    | 1    | 1    | 0.0                  |
| Miya    | 202    | maths    | 100    | 1    | 1    | 2    | 0.0                  |
| Lufi    | 202    | maths    | 98     | 3    | 2    | 3    | 0.2857142857142857   |
| Joy     | 202    | maths    | 87     | 4    | 3    | 4    | 0.42857142857142855  |
| Roy     | 202    | maths    | 84     | 5    | 4    | 5    | 0.5714285714285714   |
| Erw     | 202    | maths    | 84     | 5    | 4    | 6    | 0.5714285714285714   |
| Tong    | 202    | maths    | 79     | 7    | 5    | 7    | 0.8571428571428571   |
| Luo     | 202    | maths    | 73     | 8    | 6    | 8    | 1.0                  |
+---------+--------+----------+--------+------+------+------+----------------------+--+

```
## 差值

lag(expr [, offset] [, default]) OVER ([partition_by_clause] order_by_clause)
This function returns the value of an expression using column values from a preceding row. You specify an integer offset, which designates a row position some number of rows previous to the current row. Any column references in the expression argument refer to column values from that prior row.

lead(expr [, offset] [, default]) OVER([partition_by_clause] order_by_clause)
This function returns the value of an expression using column values from a following row. You specify an integer offset, which designates a row position some number of rows after to the current row. Any column references in the expression argument refer to column values from that later row.

```
select
    class,
    course,
    score,
    rank() over(partition by class, course order by score desc) rk1,
    LAG(score,2) over(partition by class, course order by score) as lag,
    lead(score,2) over(partition by class, course order by score) as lead
from test_analy_funs
where course='chinese';
+--------+----------+--------+------+-------+-------+--+
| class  |  course  | score  | rk1  |  lag  | lead  |
+--------+----------+--------+------+-------+-------+--+
| 201    | chinese  | 28     | 8    | NULL  | 88    |
| 201    | chinese  | 83     | 7    | NULL  | 89    |
| 201    | chinese  | 88     | 6    | 28    | 92    |
| 201    | chinese  | 89     | 5    | 83    | 95    |
| 201    | chinese  | 92     | 4    | 88    | 99    |
| 201    | chinese  | 95     | 3    | 89    | 100   |
| 201    | chinese  | 99     | 2    | 92    | NULL  |
| 201    | chinese  | 100    | 1    | 95    | NULL  |
| 202    | chinese  | 28     | 8    | NULL  | 50    |
| 202    | chinese  | 30     | 7    | NULL  | 78    |
| 202    | chinese  | 50     | 6    | 28    | 82    |
| 202    | chinese  | 78     | 5    | 30    | 88    |
| 202    | chinese  | 82     | 4    | 50    | 92    |
| 202    | chinese  | 88     | 3    | 78    | 99    |
| 202    | chinese  | 92     | 2    | 82    | NULL  |
| 202    | chinese  | 99     | 1    | 88    | NULL  |
+--------+----------+--------+------+-------+-------+--+
```

## 第一个或者最后一个值

获取分组集合中的第一个值或者最后一个值，last_value需要加rows between current row and unbounded following

first_value(expr) OVER([partition_by_clause] order_by_clause [window_clause])
Returns the expression value from the first row in the window. The return value is NULL if the input expression is NULL.

last_value(expr) OVER([partition_by_clause] order_by_clause [window_clause])
Returns the expression value from the last row in the window. The return value is NULL if the input expression is NULL.

```
select
    stu_id,
    class,
    course,
    score,
    rank() over(partition by class, course order by score desc) rk1,
    first_value(score) over(partition by class, course order by score) first,
    last_value(score) over(partition by class, course order by score) last,
    last_value(score) over(partition by class, course order by score rows between current row and unbounded following) last
from test_analy_funs
where course='chinese'
order by class, stu_id;

+---------+--------+----------+--------+------+--------+-------+-------+--+
| stu_id  | class  |  course  | score  | rk1  | first  | last  | last  |
+---------+--------+----------+--------+------+--------+-------+-------+--+
| 1       | 201    | chinese  | 28     | 8    | 28     | 28    | 100   |
| 2       | 201    | chinese  | 99     | 2    | 28     | 99    | 100   |
| 3       | 201    | chinese  | 89     | 5    | 28     | 89    | 100   |
| 4       | 201    | chinese  | 100    | 1    | 28     | 100   | 100   |
| 5       | 201    | chinese  | 88     | 6    | 28     | 88    | 100   |
| 6       | 201    | chinese  | 83     | 7    | 28     | 83    | 100   |
| 7       | 201    | chinese  | 92     | 4    | 28     | 92    | 100   |
| 8       | 201    | chinese  | 95     | 3    | 28     | 95    | 100   |
| 9       | 202    | chinese  | 82     | 4    | 28     | 82    | 99    |
| 10      | 202    | chinese  | 92     | 2    | 28     | 92    | 99    |
| 11      | 202    | chinese  | 99     | 1    | 28     | 99    | 99    |
| 12      | 202    | chinese  | 78     | 5    | 28     | 78    | 99    |
| 13      | 202    | chinese  | 28     | 8    | 28     | 28    | 99    |
| 14      | 202    | chinese  | 50     | 6    | 28     | 50    | 99    |
| 15      | 202    | chinese  | 30     | 7    | 28     | 30    | 99    |
| 16      | 202    | chinese  | 88     | 3    | 28     | 88    | 99    |
+---------+--------+----------+--------+------+--------+-------+-------+--+
16 rows selected (25.701 seconds)
```


## 分桶

ntile()

Ntile 是Hive很强大的一个分析函数。
可以看成是：它把有序的数据集合 平均分配 到 指定的数量（num）个桶中, 将桶号分配给每一行。如果不能平均分配，则优先分配较小编号的桶，并且各个桶中能放的行数最多相差1。
语法是：
     ntile (num)  over ([partition_clause]  order_by_clause)  as your_bucket_num

   然后可以根据桶号，选取前或后 n分之几的数据。

```  
select 
      stu_id, 
      class ,
      score,
      NTILE(5) OVER( partition by class, course order by score desc) AS rn 
from test_analy_funs
where course='chinese';
+---------+--------+--------+-----+--+
| stu_id  | class  | score  | rn  |
+---------+--------+--------+-----+--+
| 4       | 201    | 100    | 1   |
| 2       | 201    | 99     | 1   |
| 8       | 201    | 95     | 2   |
| 7       | 201    | 92     | 2   |
| 3       | 201    | 89     | 3   |
| 5       | 201    | 88     | 3   |
| 6       | 201    | 83     | 4   |
| 1       | 201    | 28     | 5   |
| 11      | 202    | 99     | 1   |
| 10      | 202    | 92     | 1   |
| 16      | 202    | 88     | 2   |
| 9       | 202    | 82     | 2   |
| 12      | 202    | 78     | 3   |
| 14      | 202    | 50     | 3   |
| 15      | 202    | 30     | 4   |
| 13      | 202    | 28     | 5   |
+---------+--------+--------+-----+--+

```

## 值排序占比

cume_dist()

CUME_DIST 小于等于当前值的行数/分组内总行数

```
SELECT 
class,
stu_id,
score,
CUME_DIST() OVER(partition by class ORDER BY score) AS cm1
from test_analy_funs
where course='chinese';

+--------+---------+--------+--------+--+
| class  | stu_id  | score  |  cm1   |
+--------+---------+--------+--------+--+
| 201    | 1       | 28     | 0.125  |
| 201    | 6       | 83     | 0.25   |
| 201    | 5       | 88     | 0.375  |
| 201    | 3       | 89     | 0.5    |
| 201    | 7       | 92     | 0.625  |
| 201    | 8       | 95     | 0.75   |
| 201    | 2       | 99     | 0.875  |
| 201    | 4       | 100    | 1.0    |
| 202    | 13      | 28     | 0.125  |
| 202    | 15      | 30     | 0.25   |
| 202    | 14      | 50     | 0.375  |
| 202    | 12      | 78     | 0.5    |
| 202    | 9       | 82     | 0.625  |
| 202    | 16      | 88     | 0.75   |
| 202    | 10      | 92     | 0.875  |
| 202    | 11      | 99     | 1.0    |
+--------+---------+--------+--------+--+

```
