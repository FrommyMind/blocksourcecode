---
title: Sqoop 导入数据
date: 2018-05-12 14:27:44
tags: 
    - sqoop
---
# Sqoop 导入数据

## sqoop help

```
Available commands:
  codegen            Generate code to interact with database records
  create-hive-table  Import a table definition into Hive
  eval               Evaluate a SQL statement and display the results
  export             Export an HDFS directory to a database table
  help               List available commands
  import             Import a table from a database to HDFS
  import-all-tables  Import tables from a database to HDFS
  import-mainframe   Import datasets from a mainframe server to HDFS
  job                Work with saved jobs
  list-databases     List available databases on a server
  list-tables        List available tables in a database
  merge              Merge results of incremental imports
  metastore          Run a standalone Sqoop metastore
  version            Display version information

```
<!-- more -->
### sqoop codegen

codegen 命令用来生成Java代码
例如：

```
sqoop codegen --connect jdbc:mysql://quickstart:3306/retail_db  \
--username retail_dba --password cloudera -e 'select * from orders  \
where $CONDITIONS limit 10'
```


### sqoop create-hive-table

根据源数据库表结构在hive中创建表,可以创建别名的表，但是只能创建textfile格式的表

```
sqoop create-hive-table --connect jdbc:mysql://quickstart:3306/retail_db \
 --username retail_dba --password cloudera --table orders --hive-table orders_2
```
在hive中查看创建的表的结构

```
CREATE TABLE `orders_2`(                           
   `order_id` int,                                  
   `order_date` string,                             
   `order_customer_id` int,                         
   `order_status` string)                           
 COMMENT 'Imported by sqoop on 2017/10/21 02:27:38' 
 ROW FORMAT SERDE                                   
   'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'  
 WITH SERDEPROPERTIES (                             
   'field.delim'='\u0001',                          
   'line.delim'='\n',                               
   'serialization.format'='\u0001')                 
 STORED AS INPUTFORMAT                              
   'org.apache.hadoop.mapred.TextInputFormat'       
 OUTPUTFORMAT                                       
   'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat' 
 LOCATION                                           
   'hdfs://quickstart.cloudera:8020/user/hive/warehouse/orders_2' 
 TBLPROPERTIES (                                    
   'transient_lastDdlTime'='1508578063')            

```
### sqoop eval
执行一段sql

```
sqoop eval --connect jdbc:mysql://quickstart:3306/retail_db  \
--username retail_dba --password cloudera --query 'select count(1) from orders'
```
结果

```
------------------------
 count(1)              
------------------------
 68883                 
------------------------
```

### sqoop export
	将hive文件夹中的数据导入到数据库表中
Parquet 表

```
sqoop export --connect jdbc:mysql://quickstart:3306/retail_db --username=retail_dba \
 --password cloudera  --hcatalog-table "products" --hcatalog-database "default"  \
 --table products_1 -m 1
```
Text表

```
sqoop export --connect jdbc:mysql://quickstart:3306/retail_db --username=retail_dba  \
--password cloudera  --export-dir /user/hive/warehouse/products_2 \
 --input-fields-terminated-by '\001' --table products_2 -m 1
```
	
### Sqoop  import

导入一张表到HDFS

```
sqoop import --connect jdbc:mysql://quickstart:3306/retail_db  \
--username=retail_dba --password=cloudera   \
--target-dir /usr/hive/warehouse/orders --table orders --hive-import
```
Hive 表结构

```
 CREATE TABLE `orders`(                             
   `order_id` int,                                  
   `order_date` string,                             
   `order_customer_id` int,                         
   `order_status` string)                           
 COMMENT 'Imported by sqoop on 2017/10/26 05:50:14' 
 ROW FORMAT SERDE                                   
   'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'  
 WITH SERDEPROPERTIES (                             
   'field.delim'='\u0001',                          
   'line.delim'='\n',                               
   'serialization.format'='\u0001')                 
 STORED AS INPUTFORMAT                              
   'org.apache.hadoop.mapred.TextInputFormat'       
 OUTPUTFORMAT                                       
   'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat' 
 LOCATION                                           
   'hdfs://quickstart.cloudera:8020/user/hive/warehouse/orders' 
 TBLPROPERTIES (                                    
   'COLUMN_STATS_ACCURATE'='true',                  
   'numFiles'='1',                                  
   'numRows'='0',                                   
   'rawDataSize'='0',                               
   'totalSize'='2999944',                           
   'transient_lastDdlTime'='1509022219')            
```
HDFS 文件

```
[cloudera@quickstart ~]$ hdfs dfs -ls /user/hive/warehouse/orders
Found 2 items
-rw-r--r--   1 cloudera supergroup          0 2017-10-26 06:20 /user/hive/warehouse/orders/_SUCCESS
-rwxr-xr-x   1 cloudera supergroup    2999944 2017-10-26 06:20 /user/hive/warehouse/orders/part-m-00000
```

导入parquet格式表

```
sqoop import --connect jdbc:mysql://quickstart:3306/retail_db  \
--username=retail_dba --password=cloudera   \
--target-dir /usr/hive/warehouse/orders --table orders --hive-import --as-parquetfile
```

HDFS文件
```
[cloudera@quickstart ~]$ hdfs dfs -ls /user/hive/warehouse/orders/
Found 5 items
drwxr-xr-x   - cloudera supergroup          0 2017-10-26 05:53 /user/hive/warehouse/orders/.metadata
drwxr-xr-x   - cloudera supergroup          0 2017-10-26 05:54 /user/hive/warehouse/orders/.signals
-rw-r--r--   1 cloudera supergroup     488257 2017-10-26 05:54 /user/hive/warehouse/orders/5e708d3e-a273-48de-8a3d-37efcdaf8d62.parquet
-rw-r--r--   1 cloudera supergroup          0 2017-10-26 05:50 /user/hive/warehouse/orders/_SUCCESS
-rwxr-xr-x   1 cloudera supergroup    2999944 2017-10-26 05:50 /user/hive/warehouse/orders/part-m-00000
```

加上Snappy 压缩

```
[cloudera@quickstart ~]$ hdfs dfs -ls /user/hive/warehouse/orders/
Found 3 items
drwxr-xr-x   - cloudera supergroup          0 2017-10-26 06:04 /user/hive/warehouse/orders/.metadata
drwxr-xr-x   - cloudera supergroup          0 2017-10-26 06:05 /user/hive/warehouse/orders/.signals
-rw-r--r--   1 cloudera supergroup     488257 2017-10-26 06:05 /user/hive/warehouse/orders/bfa9265e-2711-494e-a924-bd8c6591954e.parquet
```

### Sqoop import-all-tables

导入数据库所有表到hdfs：创建表并将数据导入HDFS

```
sqoop import-all-tables --connect jdbc:mysql://quickstart:3306/retail_db  \
--username=retail_dba --password=cloudera --compress-codec=snappy \ 
--as-parquetfile --warehouse-dir=/user/hive/warehouse --hive-import
```

检查其中一张表到hdfs文件

```
[cloudera@quickstart ~]$ hdfs dfs -ls /user/hive/warehouse/orders
Found 3 items
drwxr-xr-x   - cloudera supergroup          0 2017-10-20 18:57 /user/hive/warehouse/orders/.metadata
drwxr-xr-x   - cloudera supergroup          0 2017-10-20 18:58 /user/hive/warehouse/orders/.signals
-rw-r--r--   1 cloudera supergroup     488257 2017-10-20 18:58 /user/hive/warehouse/orders/a851e80b-1238-465d-93b7-a8ac11c53697.parquet

```

检查hive表结构

```
show create table orders
```

```
 CREATE TABLE `orders`(                             
   `order_id` int,                                  
   `order_date` bigint,                             
   `order_customer_id` int,                         
   `order_status` string)                           
 ROW FORMAT SERDE                                   
   'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe'  
 STORED AS INPUTFORMAT                              
   'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat'  
 OUTPUTFORMAT                                       
   'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat' 
 LOCATION                                           
   'hdfs://quickstart.cloudera:8020/user/hive/warehouse/orders' 
 TBLPROPERTIES (                                    
   'COLUMN_STATS_ACCURATE'='false',                 
   'avro.schema.url'='hdfs://quickstart.cloudera:8020/user/hive/warehouse/orders/.metadata/schemas/1.avsc',  
   'kite.compression.type'='snappy',                
   'numFiles'='0',                                  
   'numRows'='-1',                                  
   'rawDataSize'='-1',                              
   'totalSize'='0',                                 
   'transient_lastDdlTime'='1508551065')   
           
```


### Sqoop list-databases

查看数据库列表

```
sqoop list-databases --connect jdbc:mysql://quickstart:3306  \
--username retail_dba --password cloudera
```
结果

```
information_schema
retail_db
```


### Sqoop list-tables

查看数据库中表到列表

```
sqoop list-tables --connect jdbc:mysql://quickstart:3306/retail_db  \
--username retail_dba --password cloudera

```
结果：

```
categories
customers
departments
order_items
orders
products
```
