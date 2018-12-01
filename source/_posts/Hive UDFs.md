---
title: Hive UDFs
date: 2018-05-13 11:50:26
tags: 
  - Hive
  - Udf
---
# Hive UDFs

## 创建自定义UDF

首先需要一个类继承hive.udf类。

```
package com.example.hive.udf;
 
import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.io.Text;
 
public final class Lower extends UDF {
  public Text evaluate(final Text s) {
    if (s == null) { return null; }
    return new Text(s.toString().toLowerCase());
  }
}
```

编译之后，将jar包放到hive的classpath路径下。

```
## 默认读取当前路径，hiveserver2上的路径。
hive> add jar my_jar.jar;
Added my_jar.jar to class path

## 也可以指定绝对路径
hive> add jar /tmp/my_jar.jar;
Added /tmp/my_jar.jar to class path

## 查看所有的jars
hive> list jars;
my_jar.jar
```
创建临时函数并使用

```
## 创建函数
create temporary function my_lower as 'com.example.hive.udf.Lower';

## 使用 
 select my_lower(title), sum(freq) from titles group by my_lower(title);
 
## 或者使用如下方法创建。
CREATE FUNCTION myfunc AS 'myclass' USING JAR 'hdfs:///path/to/jar';
```

## Hive Auxiliary JARs Directory
在CDH上可以将开发的jar放到 *HIVE\_AUX\_JARS_PATH* 配置的路径下（这个是本地路径，并非hdfs路径),然后重启，这样所有的用户都能够读取并使用这些jar包。


## UDF 开发



