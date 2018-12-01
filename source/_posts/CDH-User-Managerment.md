---
title: CDH 用户管理
date: 2018-05-13 10:15:26
tags: 
  - CDH
  - User
---
# CDH User Managerment

## Hive

**创建一个普通用户 hive_test**

- 在OS上创建这个普通用户 hive_test

```	
nodes=`cat /etc/hosts  | grep -v ^# | grep htsec | awk '{print $2}'`
for target in $nodes; do
ssh root@$target <<comd
useradd hive_test
usermod -a -G hive hive_test
comd
done
```
<!-- more -->
- 在hdfs上创建用户文件夹
  如果集群启用了Kerberos，那么需要首先kinit，否则不需要   
	
```
kinit -kt hdfs.keytab hdfs/nn1.example.com
hdfs dfs -mkdir -p /user/hive_test
hdfs dfs -chown -R dev_user:hive_test /user/hive_test
```
	
**创建role**

只有admin用户可以创建和删除roles；hive、hue、impala都是默认的admin用户。

```
kinit -kt hive.keytab hive/nn1.example.com 
beeline -u "jdbc:hive2://nn1.example.com:10000/default;principal=hive/nn1.example.com@EXAMPLE.COM"
create role hive_test;  //创建用户 
```

**删除role**

	drop role hive_test；
	
**grant role**

	grant role hive_test to group hive_test;  ## 将用户分到一个组，在Sentry里用户必须在组里才能使用。
**revoke role**

	revoke role hive_test from group hive_test；

- 在hive上grant权限

**grant权限**

```
GRANT    
    <PRIVILEGE> [, <PRIVILEGE> ]    
    ON <OBJECT> <object_name>    
    TO ROLE <roleName> [,ROLE <roleName>]

e.g. grant all on server server1 to role hive_test;
```

- hive的权限分类

* **ALL**： 所有权限
* **ALTER**：允许修改元数据（modify metadata data of  object）---表信息数据
* **UPDATE**： 允许修改物理数据（modify physical data of  object）---实际数据
* **CREATE**： 允许进行Create操作
* **DROP**： 允许进行DROP操作
* **INDEX**： 允许建索引（目前还没有实现）
* **LOCK**： 当出现并发的使用允许用户进行LOCK和UNLOCK操作
* **SELECT**：允许用户进行SELECT操作
* **SHOW_DATABASE**： 允许用户查看可用的数据库

- 查询权限

```
show roles;
show grant role hive_test;
```
**revoke 权限**

	REVOKE    
    <PRIVILEGE> [, <PRIVILEGE> ]    
    ON <OBJECT> <object_name>    
    FROM ROLE <roleName> [,ROLE <roleName>]
	e.g. REVOKE SELECT(column_name) ON TABLE table_name FROM ROLE 
	     REVOKE SELECT(column_name) ON TABLE table_name FROM ROLE role_name; // 将列的查询权限赋值给用户
	     REVOKE ROLE role_name from GROUP group_name[,GROUP group_name] // 将用户移除一个组


**grant URIs**
	
	GRANT ALL ON URI 'hdfs://namenode:XXX/path/to/table'
	CREATE EXTERNAL TABLE foo LOCATION 'namenode:XXX/path/to/table'


**SET ROLE Statement**

*SET ROLE NONE*
   使所有的role变成非活动状态。

*SET ROLE ALL*
   使ROLE拥有所有的权限。

*SET ROLE <role name>*
   激活一个role，并将分配其的权限激活。


## Impala
impala 的权限和hive一致

## Hbase
- 在OS上创建用户 hive_test （同Hive)
	需要添加用户到组hbase
	usermod -a -G base hive_test
- 在hdfs上创建用户文件夹 （同Hive)
- 在Hbase上grant权限
	如果集群启用了Kerberos，那么需要首先kinit，否则不需要 
```
kinit -kt hbase.keytab hbase/nn1.example.com
grant 'hive_test' ,'RWXCA','@hbase'
```
- Hbase权限分类
  HBase提供的五个权限标识符：RWXCA,分别对应着READ('R'), WRITE('W'), EXEC('X'), CREATE('C'), ADMIN('A')
   HBase提供的五个权限标识符：RWXCA,分别对应着READ('R'), WRITE('W'), EXEC('X'), CREATE('C'), ADMIN('A')
HBase提供的安全管控级别包括：
Superuser：拥有所有权限的超级管理员用户。通过hbase.superuser参数配置
Global：全局权限可以作用在集群所有的表上。
Namespace ：命名空间级。
Table：表级。
ColumnFamily：列簇级权限。
Cell：单元级。
	
将namespace权限符给用户
    
    grant 'user', 'RWXCA', '@namespace'
将表的权限赋给用户

    grant 'user', 'RWXCA', 'tablename'
将表的一个字段的权限赋给用户
    
    grant 'user', 'RWXCA', 'TABLE', 'CF', 'CQ'
http://hbase.apache.org/book.html

- 查看用户权限
```
    user_permission '@namespace'
```
