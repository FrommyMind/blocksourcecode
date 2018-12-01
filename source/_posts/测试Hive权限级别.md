---
title: 测试Hive权限级别
date: 2018-05-12 14:34:44
tags: 
    - Hive
---

# 测试Hive权限级别

## Hive权限分类

|名称	|解释
|-----|-----|ALL	 |所有权限|ALTER|	允许修改元数据（modify metadata data of object）---表信息数据 |UPDATE|	允许修改物理数据（modify physical data of object）---实际数据 |CREATE|	允许进行Create操作 |DROP|	允许进行DROP操作 |INDEX|	允许建索引（目前还没有实现）|LOCK|	当出现并发的使用允许用户进行LOCK和UNLOCK操作 |SELECT|	允许用户进行SELECT操作|SHOW_DATABASE|	允许用户查看可用的数据库


**Sentry能管理的权限**

SELECT、INSERT、ALL 即Sentry只能分这三种权限进行赋值。

## Hive User、Group、Role

**Role**

可以使用Grant语法，赋权限给Role。

**User**

操作系统上的用户，Hive权限里无法创建用户

**Group**

操作系统上的组，Hive权限里如法创建组

**关系**：

* 一个用户就是操作系统上的用户
* 可以把一个用户分配到一个组或者多个组上
* 一个组就是操作系统上的组
* 一个组可以有多个角色
* 角色只能被赋值给组，不能赋值给用户
* 一个角色可以被赋值多种权限
* 一个用户，继承它所在的所有组的所有权限

## 权限管理与分类

不同的部门、应用在操作系统上属于不同的组 
角色和角色的名称应该是有标准的
一个部门的用户，在操作系统上属于一个组，用户名不相同 可以有不同的权限

如:   
部门 IT_DEV 有读取数据库内容的权限  
Manager 有写数据库的权限   
如果user1 同时属于2个组，那么user1 具有读写数据库的权限   
如果user2 只属于ITDEV4组，那么user2 只有读取数据库内容的权限

## 测试准备
创建一个测试库

```
create database test;
```

创建表并插入数据

```
use test;
create table test.test_partition (id int, name string) partitioned by (dt int);
insert into test.test_partition partition(dt=20180101) select '1','daniel';
select * from test_partition;
+--------------------+----------------------+--------------------+--+
| test_partition.id  | test_partition.name  | test_partition.dt  |
+--------------------+----------------------+--------------------+--+
| 1                  | daniel               | 20180101           |
+--------------------+----------------------+--------------------+--+
```

创建一个测试用户

```
create role test_auto;
```
在每个操作系统上都创建这个用户
添加一个组

```
groupadd it_dev    ## 创建一个用户组
useradd test_auto  ## 创建一个用户
usermod -g it_dev test_auto  ## 将这个用户只加入 itdev4 组
## 检查用户所属的组
[root@dn3 ~]# id test_auto;
uid=1008(test_auto) gid=1011(itdev4) groups=1011(itdev4)
[root@dn3 ~]#
```

在Hive用户下，beeline执行，添加角色并分配权限

```
create role reader;
grant select on database test to role reader;
create role writer;
grant insert on database test to role writer;
```

将只读角色赋到组上

```
grant role reader to group itdev4;

show role grant  group  itdev4;
+---------+---------------+-------------+----------+--+
|  role   | grant_option  | grant_time  | grantor  |
+---------+---------------+-------------+----------+--+
| reader  | false         | NULL        | --       |
+---------+---------------+-------------+----------+--+
```

使用用户test_auto连接hive

```
beeline -n test_auto -u jdbc:hive2://nn2.htsec.com:10000/test
show tables; ## 执行成功
```
将写的权限赋到组上

```
grant role writer to group itdev4;
show role grant  group  itdev4;
+---------+---------------+-------------+----------+--+
|  role   | grant_option  | grant_time  | grantor  |
+---------+---------------+-------------+----------+--+
| reader  | false         | NULL        | --       |
| writer  | false         | NULL        | --       |
+---------+---------------+-------------+----------+--+
```

向表中插入数据，执行成功

```
insert into test_partition partition(dt=20180101) select '2','tom';
select * from test_partition;
+--------------------+----------------------+--------------------+--+
| test_partition.id  | test_partition.name  | test_partition.dt  |
+--------------------+----------------------+--------------------+--+
| 1                  | daniel               | 20180101           |
| 2                  | tom                  | 20180101           |
+--------------------+----------------------+--------------------+--+
```

创建用户test2，并添加到it_dev 下

```
useradd -G it_dev test2
[root@nn1 ~]# id test2
uid=1009(test2) gid=1010(itdev4) groups=1010(itdev4)
```
使用beeline连接并测试

```
beeline -n test2 -u jdbc:hive2://nn2.htsec.com:10000/test
select * from test_partition;
+--------------------+----------------------+--------------------+--+
| test_partition.id  | test_partition.name  | test_partition.dt  |
+--------------------+----------------------+--------------------+--+
| 1                  | daniel               | 20180101           |
| 2                  | tom                  | 20180101           |
+--------------------+----------------------+--------------------+--+
insert into test_partition partition(dt=20180101) select '3','Monkey';

select * from test_partition;
+--------------------+----------------------+--------------------+--+
| test_partition.id  | test_partition.name  | test_partition.dt  |
+--------------------+----------------------+--------------------+--+
| 1                  | daniel               | 20180101           |
| 2                  | tom                  | 20180101           |
| 3                  | Monkey               | 20180101           |
+--------------------+----------------------+--------------------+--+
```

## 测试

### 测试SELECT


使用test_auto连接hiveserver2,并测试

```
beeline -n test_auto -u jdbc:hive2://xxx:10000

show databases;   ##看不到test数据库
+----------------+--+
| database_name  |
+----------------+--+
| default        |
+----------------+--+

use test; ## 报错，无法切换数据库
User test_auto does not have privileges for SWITCHDATABASE
```

```
beeline -n test_auto -u jdbc:hive2://xxx:10000/test
show tables;  ## 无法查看任何表。
+-----------+--+
| tab_name  |
+-----------+--+
+-----------+--+
```
在hive用户下，查看test_auto是否在test_auto 组里。

```
show role grant  group test_auto;  ## test_auto组下没有任何的role；
+-------+---------------+-------------+----------+--+
| role  | grant_option  | grant_time  | grantor  |
+-------+---------------+-------------+----------+--+
+-------+---------------+-------------+----------+--+

grant role test_auto to group test_auto;
show role grant  group test_auto;
+------------+---------------+-------------+----------+--+
|    role    | grant_option  | grant_time  | grantor  |
+------------+---------------+-------------+----------+--+
| test_auto  | false         | NULL        | --       |
+------------+---------------+-------------+----------+--+
```
再在test_auto用户下，执行命令

```
show databases;
+----------------+--+
| database_name  |
+----------------+--+
| default        |
| test           |
+----------------+--+

show tables;  ##查看到表
+------------------------+--+
|        tab_name        |
+------------------------+--+
| test_partition         |
+------------------------+--+

select * from test_partition;
+--------------------+----------------------+--------------------+--+
| test_partition.id  | test_partition.name  | test_partition.dt  |
+--------------------+----------------------+--------------------+--+
| 1                  | daniel               | 20180101           |
+--------------------+----------------------+--------------------+--+

```
查看表的表结构
**DESCRIBE TABLE**

```
desc test_table; ## 报错,因为test_table是一个Kudu的表。
 FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. java.lang.ClassNotFoundException Class  not found (state=08S01,code=1)

desc test_partition; 
+--------------------------+-----------------------+-----------------------+--+
|         col_name         |       data_type       |        comment        |
+--------------------------+-----------------------+-----------------------+--+
| id                       | int                   |                       |
| name                     | string                |                       |
| dt                       | int                   |                       |
|                          | NULL                  | NULL                  |
| # Partition Information  | NULL                  | NULL                  |
| # col_name               | data_type             | comment               |
|                          | NULL                  | NULL                  |
| dt                       | int                   |                       |
+--------------------------+-----------------------+-----------------------+--+
```

查看分区
**SHOW PARTITIONS**

```
show partitions test_partition; ## 有查看表分区的权限
+--------------+--+
|  partition   |
+--------------+--+
| dt=20180101  |
+--------------+--+
```

创建视图的权限 **CREATE VIEW**

```
create view test_view as select * from test_partition;
ser test_auto does not have privileges for CREATEVIEW
 The required privileges: Server=server1->Db=test->action=*; (state=42000,code=40000)
```


分析表 **ANALYZE TABLE**

```
analyze table test_partition partition(dt=20180101) COMPUTE STATISTICS; ## 需要Insert 权限
Error: Error while compiling statement: FAILED: SemanticException No valid privileges
 User test_auto does not have privileges for QUERY
 The required privileges: Server=server1->Db=test->Table=test_partition->action=insert; (state=42000,code=40000)
 
analyze table test_partition compute statistics for columns; ## 执行成功
```

查看表中的列 **SHOW COLUMNS**

```
show columns in test_partition;
+--------+--+
| field  |
+--------+--+
| id     |
| name   |
| dt     |
+--------+--+

```查看表的统计信息 **SHOW TABLE STATUS**

```
DESCRIBE EXTENDED test_partition;

+-----------------------------+----------------------------------------------------+-----------------------+--+
|          col_name           |                     data_type                      |        comment        |
+-----------------------------+----------------------------------------------------+-----------------------+--+
| id                          | int                                                |                       |
| name                        | string                                             |                       |
| dt                          | int                                                |                       |
|                             | NULL                                               | NULL                  |
| # Partition Information     | NULL                                               | NULL                  |
| # col_name                  | data_type                                          | comment               |
|                             | NULL                                               | NULL                  |
| dt                          | int                                                |                       |
|                             | NULL                                               | NULL                  |
| Detailed Table Information  | Table(tableName:test_partition, dbName:test, owner:hive, createTime:1516768367, lastAccessTime:0, retention:0, sd:StorageDescriptor(cols:[FieldSchema(name:id, type:int, comment:null), FieldSchema(name:name, type:string, comment:null), FieldSchema(name:dt, type:int, comment:null)], location:hdfs://xxx/user/hive/dw/test.db/test_partition, inputFormat:org.apache.hadoop.mapred.TextInputFormat, outputFormat:org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat, compressed:false, numBuckets:-1, serdeInfo:SerDeInfo(name:null, serializationLib:org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, parameters:{serialization.format=1}), bucketCols:[], sortCols:[], parameters:{}, skewedInfo:SkewedInfo(skewedColNames:[], skewedColValues:[], skewedColValueLocationMaps:{}), storedAsSubDirectories:false), partitionKeys:[FieldSchema(name:dt, type:int, comment:null)], parameters:{numPartitions=1, transient_lastDdlTime=1516768367}, viewOriginalText:null, viewExpandedText:null, tableType:MANAGED_TABLE) |


DESCRIBE EXTENDED test_partition.id partition(dt=20180101);

+-----------+------------+--------------------+--+
| col_name  | data_type  |      comment       |
+-----------+------------+--------------------+--+
| id        | int        | from deserializer  |
+-----------+------------+--------------------+--+

describe formatted test_partition.id; ## 执行成功
```查看表的属性信息 **SHOW TABLE PROPERTIES**```
show tblproperties test_partition;
+------------------------+-------------+--+
|       prpt_name        | prpt_value  |
+------------------------+-------------+--+
| transient_lastDdlTime  | 1516768367  |
+------------------------+-------------+--+

show tblproperties test_partition('transient_lastDdlTime');

+-------------+-------------+--+
|  prpt_name  | prpt_value  |
+-------------+-------------+--+
| 1516768367  | NULL        |
+-------------+-------------+--+
```
复制表 **CREATE TABLE AS SELECT**

```
create table createas_test as select * from test_partition;
Error: Error while compiling statement: FAILED: SemanticException No valid privileges
 User test_auto does not have privileges for CREATETABLE_AS_SELECT
 The required privileges: Server=server1->Db=test->action=*; (state=42000,code=40000)
```
查看表的创建语句 **SHOW CREATE TABLE**

```
show create table test_partition;
+----------------------------------------------------+--+
|                   createtab_stmt                   |
+----------------------------------------------------+--+
| CREATE TABLE `test_partition`(                     |
|   `id` int,                                        |
|   `name` string)                                   |
| PARTITIONED BY (                                   |
|   `dt` int)                                        |
| ROW FORMAT SERDE                                   |
|   'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'  |
| STORED AS INPUTFORMAT                              |
|   'org.apache.hadoop.mapred.TextInputFormat'       |
| OUTPUTFORMAT                                       |
|   'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat' |
| LOCATION                                           |
|   'hdfs://xxx/user/hive/dw/test.db/test_partition' |
| TBLPROPERTIES (                                    |
|   'transient_lastDdlTime'='1516768367')            |
+----------------------------------------------------+--+

```

分析 **EXPLAIN**

```
EXPLAIN select * from test_partition;

+----------------------------------------------------+--+
|                      Explain                       |
+----------------------------------------------------+--+
| STAGE DEPENDENCIES:                                |
|   Stage-0 is a root stage                          |
|                                                    |
| STAGE PLANS:                                       |
|   Stage: Stage-0                                   |
|     Fetch Operator                                 |
|       limit: -1                                    |
|       Processor Tree:                              |
|         TableScan                                  |
|           alias: test_partition                    |
|           Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: COMPLETE |
|           Select Operator                          |
|             expressions: id (type: int), name (type: string), dt (type: int) |
|             outputColumnNames: _col0, _col1, _col2 |
|             Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: COMPLETE |
|             ListSink                               |
|                                                    |
+----------------------------------------------------+--+
```

### 测试 INSERT

在hive用户下

```
show grant role test_auto;
+-----------+--------+------------+---------+-----------------+-----------------+------------+---------------+-------------------+----------+--+
| database  | table  | partition  | column  | principal_name  | principal_type  | privilege  | grant_option  |    grant_time     | grantor  |
+-----------+--------+------------+---------+-----------------+-----------------+------------+---------------+-------------------+----------+--+
| test      |        |            |         | test_auto       | ROLE            | select     | false         | 1516762810118000  | --       |
+-----------+--------+------------+---------+-----------------+-----------------+------------+---------------+-------------------+----------+--+

## 回收权限
revoke select on database test from role test_auto;

## 赋予Insert权限
grant insert on database test to role test_auto;
show grant role test_auto;
+-----------+--------+------------+---------+-----------------+-----------------+------------+---------------+-------------------+----------+--+
| database  | table  | partition  | column  | principal_name  | principal_type  | privilege  | grant_option  |    grant_time     | grantor  |
+-----------+--------+------------+---------+-----------------+-----------------+------------+---------------+-------------------+----------+--+
| test      |        |            |         | test_auto       | ROLE            | insert     | false         | 1516771174805000  | --       |
+-----------+--------+------------+---------+-----------------+-----------------+------------+---------------+-------------------+----------+--+
```

在只有INSERT权限，测试如下几个在SELECT命令成功，而现在执行失败或者有问题的仍无法成功

```
select * from test_partition;

Error: Error while compiling statement: FAILED: SemanticException No valid privileges
 User test_auto does not have privileges for QUERY
 The required privileges: Server=server1->Db=test->Table=test_partition->Column=dt->action=select; (state=42000,code=40000)
 
analyze table test_partition partition(dt=20180101) COMPUTE STATISTICS;

Error: Error while compiling statement: FAILED: SemanticException No valid privileges
 User test_auto does not have privileges for QUERY
 The required privileges: Server=server1->Db=test->Table=test_partition->Column=RAW__DATA__SIZE->action=select; (state=42000,code=40000)

create table createas_test as select * from test_partition;

Error: Error while compiling statement: FAILED: SemanticException No valid privileges
 User test_auto does not have privileges for CREATETABLE_AS_SELECT
 The required privileges: Server=server1->Db=test->Table=test_partition->Column=dt->action=select; (state=42000,code=40000)
```

插入 **INSERT**

```
insert into table test_partition partition(dt=20180102) select '2','tom' ; ## 执行成功

```

导入文件 **LOAD**

```
load data local inpath  '/tmp/hosts' into table test_partition partition(dt=20180103); ## 此时/tmp/hosts文件的所有者是test_auto,但仍旧报错。
Error: Error while compiling statement: FAILED: SemanticException No valid privileges
 User test_auto does not have privileges for LOAD
 The required privileges: Server=server1->URI=file:///tmp/hosts->action=*; (state=42000,code=40000)
```

更新 **UPDATE**

```
update  test_pratition set id =3 where id=1 ;
Error: Error while compiling statement: FAILED: SemanticException [Error 10294]: Attempt to do update or delete using transaction manager that does not support these operations. (state=42000,code=10294)
```

在hive用户下，给用户赋予SELECT 权限测试以上几个失败的命令

```
grant select on base test from role test_auto;
```

```
select * from test_partition;
+--------------------+----------------------+--------------------+--+
| test_partition.id  | test_partition.name  | test_partition.dt  |
+--------------------+----------------------+--------------------+--+
| 1                  | daniel               | 20180101           |
| 2                  | tom                  | 20180102           |
+--------------------+----------------------+--------------------+--+
analyze table test_partition partition(dt=20180101) COMPUTE STATISTICS; ## 执行成功

create table createas_test as select * from test_partition; 
Error: Error while compiling statement: FAILED: SemanticException No valid privileges
 User test_auto does not have privileges for CREATETABLE_AS_SELECT
 The required privileges: Server=server1->Db=test->action=*; (state=42000,code=40000)
```

### 测试UPDATE
不支持Update权限的赋值

```
revoke select on database test from role test_auto;
revoke insert on database test from role test_auto;

grant update on database test to role test_auto;
Error: Error while compiling statement: FAILED: SemanticException Sentry does not support privilege: Update (state=42000,code=40000)
```

### 测试DELETE
不支持Delete权限的赋值

```
grant delete on database test to role test_auto;
Error: Error while compiling statement: FAILED: SemanticException Sentry does not support privilege: Delete (state=42000,code=40000)
```

### URI 

```
GRANT ALL ON URI 'hdfs://tmp/hosts' to role test_auto; ## 成功
load data  inpath  '/tmp/hosts' into table test_partition partition(dt=20180103); 


revoke all on uri 'hdfs://tmp/hosts' from role test_auto;
GRANT select ON URI 'hdfs://tmp/hosts' to role test_auto; ## 成功
load data  inpath  '/tmp/hosts' into table test_partition partition(dt=20180103);  

 grant all on uri 'hdfs:/tmp/' to role test_auto;

 insert overwrite directory "/user/test_auto/result/test_partition" row format delimited fields terminated by "\t"   select * from test_partition;
 
 insert overwrite directory "/user/test_auto/result/test_partition"   select * from test_partition;
 
 INSERT OVERWRITE DIRECTORY '/tmp/test_partition' SELECT * FROM test_partition; ## 执行成功

需要hive能在这个路径下创建文件夹，并且test_auto也有权限、

```
