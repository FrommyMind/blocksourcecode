---
title: PARQUET FILE FORMAT
date: 2018-05-13 11:59:26
tags: 
  - Parquet
  - CDH
---
# PARQUET FILE FORMAT

来源 https://www.cloudera.com/documentation/enterprise/latest/topics/impala_parquet.html#parquet_ddl

Parquet是一个列式存储文件格式，可用于Hadoop生态系统的各个组件之上。
- 列式存储：对单独一个列的查询可以只查询数据的一部分数据。
- 灵活的压缩方式：数据可以使用多种压缩方式进行压缩
- 创新的编码方案：相同的、相似的或相关的数据值的序列可以以节省磁盘空间和内存的方式表示。
- 大文件存储:Parquet的文件存储格式就是为大文件而设计的，文件范围从M-G。

## Parquet文件的压缩
对于大多数CDH的组件，Parquet文件默认是没有压缩的，CDH建议启用压缩以减少硬盘的使用和提高文件的读写效率
在读取压缩格式的Parquet文件时，你不需要指定压缩类型，但是当你写一个压缩的Parquet文件时，你必须指定压缩格式。

<!-- more -->
## HBase使用Parquet

TBD

## Hive中使用Parquet表
使用如下的命令创建一个Parquet格式的Hive表。

```
CREATE TABLE parquet_table_name (x INT, y STRING) STORED AS PARQUET;
```
当你创建完成Parquet格式的hive表， 你可以使用Impala或者Spark去访问它。

如果数据文件是由外部导入的，你也可以创建外部表
```
create external table parquet_table_name (x INT, y STRING)
ROW FORMAT SERDE 'parquet.hive.serde.ParquetHiveSerDe'
STORED AS
INPUTFORMAT "parquet.hive.DeprecatedParquetInputFormat"
OUTPUTFORMAT "parquet.hive.DeprecatedParquetOutputFormat"
LOCATION '/test-warehouse/tinytable';
```
写入数据时，设置压缩方式。
```
set parquet.compression=GZIP;
INSERT OVERWRITE TABLE tinytable SELECT * FROM texttable;
```
支持的压缩方式有UNCOMPRESSED、GZIP和SNAPPY。

## Impala中使用Parquet表
Impala只要在创建表的时候使用STORED AS PARQUET关键字指定存储格式，就能在插入和读写时使用它了。

    create table parquet_table (x int, y string) stored as parquet;
    insert into parquet_table select x, y from some_other_table;
    Inserted 50000000 rows in 33.52s
    select y from parquet_table where x between 70 and 100;

使用这种方法创建的表，可以使用Impala或者Hive进行查询。
在Impala2.0 默认Parquet文件的大小是256M，低版本是1G。避免使用INSERT ... VALUES这样的语法或者使用颗粒度比较大的字段进行分区。那样会降低Parquet的性能。
Impala的数据插入是内存密集型操作，因为每个数据文件都需要一些内存区域来保存数据。有时这样的操作可能会超过HDFS同时打开文件的限制，在这种情况下，可以考虑将插入操作拆分为每个分区一insert语句。
如果使用Sqoop将RDBMS的数据转换为Parquet文件，请注意 DATE, DATETIME, or TIMESTAMP类型的列，Parquet底层使用INT64表示这些值，Parquet使用毫秒表示这些值，但是Impala编译BIGINT作为秒的为单位的时间，因此如果你有BIGINT类型的Time类型，在转换为Parquet文件时需要除以1000 。

## Parquet文件结构
使用parquet-tools 查看Parquet文件的内容，在CDH中包含这个命令。
cat: 使用标准输出Parquet格式的文件
head: 输出文件的前几行
schema: 输出文件的schema
meta: 输出文件的元数据信息，包括key-value属性，压缩率，编码，压缩方式和行组信息。
dump: 输出所有的数据和元数据信息。

## Parquet表查询优化
Parquet表的查询依赖需要查询的列的个数和where条件。Parquet将数据根据固定块的大小切分成大的文件。
例如：下面的命令就是一个高效的查询

	select avg(income) from census_data where state = 'CA';
这个查询语句只处理了2列数据，如果表是根据state进行分区的，那将会更加高效，因为它只需要读取分区‘CA’文件夹下文件的部分数据。
下面的命令就比较低效
	
	select * from census_data;

如果对表进行大量数据的插入后，执行 COMPUTE STATS 更新meta信息。
https://www.cloudera.com/documentation/enterprise/latest/topics/impala_compute_stats.html#compute_stats

```	
COMPUTE STATS [db_name.]table_name
COMPUTE INCREMENTAL STATS [db_name.]table_name [PARTITION (partition_spec)]

partition_spec ::= partition_col=constant_value

partition_spec ::= simple_partition_spec | complex_partition_spec

simple_partition_spec ::= partition_col=constant_value

complex_partition_spec ::= comparison_expression_on_partition_col
```

注：对于一个表，不要将 COMPUTE STATS 和COMPUTE INCREMENTAL STATS 混合使用，如果想要切换，需要先使用 DROP STATS 和 DROP INCREMENTAL STATS。
对于一个比较大的表需要 400 bytes的metedata信息。如果所有表的metedata超过了2G，可能会失败几次。

优化查询的第一步就是对所有的表执行COMPUTE STATS。或者在out-of-memory错误的时候
- 准确的统计数据有助于Impala构建一个高效的查询计划，用于查询、提高性能和减少内存使用。
- 准确的统计数据帮助Impala将工作有效地分配到Parquet表中，提高性能和减少内存使用。
- 准确的统计数据有助于Impala估计每个查询所需的内存，当您使用资源管理特性时，例如权限控制和YARN资源管理框架，这一点非常重要。统计数据帮助Impala实现高并发性，充分利用可用内存，并避免与其他Hadoop组件的工作负载争用。
- 在CDH 5.10 / Impala 2.8 或者更高的版本, 当您运行计算统计数据或在Parquet表上计算增量统计语句时，Impala会自动应用查询选项设置MT_DOP = 4，以增加在此cpu密集型操作期间内节点并行度的增加。

在CDH 5.10 / Impala 2.8 或者更高的版本,可以使用如下的命令计算多个分区的信息
	
	compute incremental stats int_partitions partition (x < 100);

查询表的状态

	show table stats t1;

查询列的状态
	
	show column stats t1;

## Impala支持的文件格式
https://www.cloudera.com/documentation/enterprise/latest/topics/impala_file_formats.html#file_formats

| 文件类型 | 结构 |压缩方式|Impala能否创建|Impala能否插入|
| ---| ---|
|Parquet|有结构的|Snappy, gzip; 默认 Snappy|能|是: 创建表, 插入, Load数据和查询|
|Text|无结构的|LZO, gzip, bzip2, Snappy|是. 在创建表的时候不带STORED AS 语法, 默认文件格式是没有压缩的text, 使用ASCII 0x01 做分隔符 (通常点表Ctrl-A).|是: 创建表, 插入, Load数据和查询. 如果使用LZO压缩, 你必须在hive里面创建表和load数据. 如果使用了其他的压缩翻身,你必须load数据通过LOAD DATA, Hive,或者手动在 HDFS上操作.|
|Avro|有结构的|	Snappy, gzip, deflate, bzip2|是, 在Impala 1.4.0和更高的版本. 之前的版本使用hive创建表|不能, 导入数据时，使用正确格式的数据文件加载数据，或者在Hive中使用INSERT，然后在Impala中使用REFRESH table_name。|
|RCFile|有结构的| Snappy, gzip, deflate, bzip2|是|不能, 导入数据时，使用正确格式的数据文件加载数据，或者在Hive中使用INSERT，然后在Impala中使用REFRESH table_name。|
|SequenceFile|有结构的|Snappy, gzip, deflate, bzip2|是|不能,导入数据时，使用正确格式的数据文件加载数据，或者在Hive中使用INSERT，然后在Impala中使用REFRESH table_name。|

Impala 只支持以上的文件格式，尤其要指出，Impala不支持ORC格式的文件。

Impala 支持以下的压缩格式
- Snappy. 因为其压缩率和解压速度是优先被推荐的格式 Snappy 压缩速度非常快,但是gzip更节省空间. 在 Impala 2.0 和更高的版本支持text使用这种压缩方式。
- Gzip. 当需要更高level的压缩时，Gzip是被推荐的 (以为其更节省空间). 在 Impala 2.0 和更高的版本支持text使用这种压缩方式
- Deflate. text 文件不支持使用此种压缩方式.
- Bzip2. 在 Impala 2.0 和更高的版本支持text使用这种压缩方式
- LZO, 只能为text文件使用. Impala 只能查询 LZO压缩的 text表,但是还不能创建和插入; 在Hive中去创建表和插入数据。

## 为一个表选择文件格式


- 如果您正在处理的文件格式是已经被支持的，请使用与实际的Impala表相同的格式。如果原始格式不产生可接受的查询性能或资源使用情况，可以考虑使用不同的文件格式或压缩特性创建一个新的Impala表，并使用INSERT语句将数据复制到新表进行一次性转换。根据文件格式，您可以在impala - shell中或在Hive中运行INSERT语句。
- 许多不同的工具都可以很方便的生产文本文件，而且是便于验证和调试的。这些特性就是为什么文本是Impala创建表语句的默认格式。当性能和资源使用是主要考虑因素时，使用另一种文件格式，并考虑使用压缩。一个典型的工作流可能包括将数据复制到一个Impala表中，将CSV或TSV文件复制到适当的数据目录中，然后使用insert...select语法转换到适当的表中
- 如果您的体系结构需要在内存中存储数据，那么不要压缩数据。由于数据不需要从磁盘移动，所以没有I/O成本，但是要解压数据，需要消耗CPU成本。

## Impala数据类型
https://www.cloudera.com/documentation/enterprise/latest/topics/impala_datatypes.html
Note: 当前Impala只支持标量的数据类型，不支持复合和嵌套类型。

ARRAY类型：
语法

	column_name ARRAY < type >

	type ::= primitive_type | complex_type
