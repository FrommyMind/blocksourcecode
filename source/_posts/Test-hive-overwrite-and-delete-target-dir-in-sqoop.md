---
title: Test hive-overwrite and delete-target-dir in sqoop
date: 2018-05-16 14:15:39
tags: sqoop
---

# 测试Sqoop 中hive-overwrite和delete-target-dir是否冲突

- 指定 –hive-import –hive-table –target-dir –delete-target-dir ,并且target-dir和hive表所在的路径一致。

```
sqoop import --hive-import --connect "jdbc:mysql://lion:3306/retail_db" --username retaildba --password retaildba --table orders --hive-table orders --target-dir /user/hive/warehouse/orders/ --delete-target-dir --hive-overwrite
```

![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite1.png?raw=true)

<!-- more -->
- 指定 –hive-import –hive-table –target-dir –delete-target-dir ,并且target-dir和hive表所在的路径不一致。

```
sqoop import --hive-import --connect "jdbc:mysql://lion:3306/retail_db" --username retaildba --password retaildba --table orders --hive-table orders  --target-dir /user/hive/warehouse/order-1/ --delete-target-dir --hive-overwrite
```

![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite2.png?raw=true)

从结果来看，数据没有导入到—target-dir 指定的HDFS路径上，而是导入到表中去了。

- 如果不指定target-dir和—delete-target-dir

```
sqoop import --hive-import --connect "jdbc:mysql://lion:3306/retail_db" --username retaildba --password retaildba --table orders --hive-table orders --hive-overwrite
```
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite3.png?raw=true)

- 调整下 –hive-overwrite的位置

```
sqoop import --hive-import --hive-overwrite --connect "jdbc:mysql://lion:3306/retail_db" --username retaildba --password retaildba --table orders --hive-table orders
```

![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite4.png?raw=true)
原因是默认在当前用户的hdfs home目录下创建一个和表名一致的文件夹，先手动删除这个文件夹然后再执行命令，在执行的Log中有如下的内容，因为不是使用hive进行数据导入的。
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite5.png?raw=true)

检查结果
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite6.png?raw=true)

再次执行，即使没有delete-target-dir 仍能执行成功，并且在执行的过程中，发现确实创建了 /user/training/orders文件夹，然后在任务执行完成后将文件移动到表的路径下，如下面第二张图所示。
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite7.png?raw=true)
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite8.png?raw=true)



- 测试-query的情况

```
sqoop import --hive-import --hive-overwrite --connect "jdbc:mysql://lion:3306/retail_db" --username retaildba --password retaildba --query "select * from orders WHERE \$CONDITIONS" --hive-table orders
```
没有指定—target-dir  报如下错误
<img src="https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite9.png?raw=true" alt="drawing" style="width: 500px;"/>

```
sqoop import --hive-import --hive-overwrite --connect "jdbc:mysql://lion:3306/retail_db" --username retaildba --password retaildba --query "select * from orders WHERE \$CONDITIONS" --hive-table orders –target-dir /user/training/orders/
```

![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite10.png?raw=true)
必须指定split-by

```
sqoop import --hive-import --hive-overwrite --connect "jdbc:mysql://lion:3306/retail_db" --username retaildba --password retaildba --query "select * from orders WHERE \$CONDITIONS" --hive-table orders  --target-dir /user/training/orders-1/ --split-by order_id
```

![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite11.png?raw=true)


在导入的过程中在target-dir生成临时文件，完成后将文件移动到表的路径下。所以此时不需要—delete-target-dir
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite12.png?raw=true)

- 如果即使用--hive-table  --hive-overwrite 、--target-dir、 --delete-target-dir 并且 target-dir和表的路径一致。

```
sqoop import --hive-import --hive-overwrite --connect "jdbc:mysql://lion:3306/retail_db" --username retaildba --password retaildba --query "select * from orders WHERE \$CONDITIONS" --hive-table orders  --target-dir /user/hive/warehouse/orders/ --split-by order_id --delete-target-dir
```

报如下错误

![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite13.png?raw=true)

```
sqoop import --hive-import --hive-overwrite --connect "jdbc:mysql://lion:3306/retail_db" --username retaildba --password retaildba --query "select * from orders WHERE \$CONDITIONS" --hive-table test.orders  --target-dir /user/hive/warehouse/test.db/orders/ --split-by order_id --delete-target-dir
```
即使是加上db name也是一样的错误。







使用如下格式的代码导入也是一样的错误，但是如果是Oracle数据库，则可以执行成功但是没有数据。（暂时无法解释）

```
sqoop import -D \
mapreduce.job.queuename=hdfs \
 --hive-import \
 --hive-overwrite \
--connect "jdbc:mysql://lion:3306/retail_db" \
--username retaildba --password retaildba \
 --fetch-size 5000 \
--query "select * from orders WHERE \$CONDITIONS" \
 --target-dir '/user/hive/warehouse/test.db/orders/' \
 --delete-target-dir  \
 --hive-table test.orders \
 --hive-delims-replacement ' ' \
 --null-string '\\N' --null-non-string '\\N'\
 --split-by order_id \
```

结论： --target-dir 在有--hive-import的情况下只是一个临时的存储文件夹，而且不要和表的路径设置一样。


--hive-overwrite

首先创建一个表test.order
<img src="https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite14.png?raw=true"  style="width: 300px; display"/>
执行下面的sqoop

```
sqoop import --hive-import --hive-overwrite --connect "jdbc:mysql://lion:3306/retail_db" --username retaildba --password retaildba --query "select * from orders WHERE \$CONDITIONS" --hive-table test.orders  --target-dir /user/hive/orders/ --split-by order_id --delete-target-dir
```
再次查询表结构
<p align = "center">
<img src="https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Sqoop/testhiveoverwrite/testhiveoverwrite15.png?raw=true" alt="drawing" style="width: 600px; "/>
</p>
结论：不要轻易使用 –hive-overwrite

