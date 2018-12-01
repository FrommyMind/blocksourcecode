---
title: CCAH-131 Test
date: 2018-05-13 08:38:26
tags: CCAH-131
---
# Test

Benchmark the cluster operational metrics, test system configuration for operation and efficiency


## Execute file system commands via HTTPFS

[Using the HttpFS Server with curl](https://www.cloudera.com/documentation/enterprise/5-12-x/topics/cdh_ig_httpfs_server_curl.html#topic_25_8)

例如查看 用户training的Home目录

```
[training@elephant ~]$ curl "http://elephant:14000/webhdfs/v1?op=gethomedirectory&user.name=training"
{"Path":"\/user\/training"}
[training@elephant ~]$
```
<!-- more --> 

[WebHDFS REST API](https://archive.cloudera.com/cdh5/cdh/5/hadoop/hadoop-project-dist/hadoop-hdfs/WebHDFS.html)

```
curl -i -X PUT "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=CREATE
                    [&overwrite=<true|false>][&blocksize=<LONG>][&replication=<SHORT>]
                    [&permission=<OCTAL>][&buffersize=<INT>]"
```

查询一个目录 

```
[training@elephant ~]$ curl -i  "http://elephant:14000/webhdfs/v1/user?op=LISTSTATUS&user.name=training"
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Set-Cookie: hadoop.auth="u=training&p=training&t=simple-dt&e=1515685732115&s=9cv4wBc8rJe83+6WUg7B5xaeafE="; Path=/; HttpOnly
Content-Type: application/json
Transfer-Encoding: chunked
Date: Thu, 11 Jan 2018 05:48:52 GMT

{"FileStatuses":{"FileStatus":[{"pathSuffix":"history","type":"DIRECTORY","length":0,"owner":"mapred","group":"hadoop","permission":"777","accessTime":0,"modificationTime":1515572964146,"blockSize":0,"replication":0},{"pathSuffix":"hive","type":"DIRECTORY","length":0,"owner":"hive","group":"hive","permission":"1775","accessTime":0,"modificationTime":1515575604845,"blockSize":0,"replication":0},{"pathSuffix":"hue","type":"DIRECTORY","length":0,"owner":"hue","group":"hue","permission":"775","accessTime":0,"modificationTime":1515575637482,"blockSize":0,"replication":0},{"pathSuffix":"impala","type":"DIRECTORY","length":0,"owner":"impala","group":"impala","permission":"775","accessTime":0,"modificationTime":1515575938788,"blockSize":0,"replication":0},{"pathSuffix":"oozie","type":"DIRECTORY","length":0,"owner":"oozie","group":"oozie","permission":"775","accessTime":0,"modificationTime":1515645000550,"blockSize":0,"replication":0},{"pathSuffix":"spark","type":"DIRECTORY","length":0,"owner":"spark","group":"spark","permission":"751","accessTime":0,"modificationTime":1515573417601,"blockSize":0,"replication":0},{"pathSuffix":"training","type":"DIRECTORY","length":0,"owner":"training","group":"supergroup","permission":"755","accessTime":0,"modificationTime":1515586402387,"blockSize":0,"replication":0}]}}
[training@elephant ~]$
```

## Efficiently copy data within a cluster/between clusters

**迁移hdfs数据至新集群**

```
hadoop distcp -update hdfs://hostname:port/path hdfs://ip:8020/path
```

**迁移Hive数据仓库**

可以每次迁移一个表或者一个database。
注意： 表的内容小文件不应过多，否则会非常慢。
源集群metastore数据备份导出(mysql导出)

```
mysqldump -u root -p’密码’--skip-lock-tables -h xxx.xxx.xxx.xxx hive > mysql_hive.sql
```

新的集群导入metastore数据

```
mysql -u root -proot --default-character-set=utf8 hvie < mysql_hive.sql
```

升级hive内容库(如果hive版本需要升级操作，同版本不需要操作)

```
mysql -uroot -proot risk -hxxx.xxx.xxx.xxx < mysqlupgrade-0.13.0-to-0.14.0.mysql.sql
mysql -uroot -proot risk -hxxx.xxx.xxx.xxx < mysqlupgrade-0.14.0-to-1.1.0.mysql.sql
```

版本要依据版本序列升序升级,不可跨越版本，如当前是hive0.12打算升级到0.14，需要先升级到0.13再升级到0.14

修改 metastore 内容库的集群信息

因为夸集群，hdfs访问的名字可能变化了，所以需要修改下hive库中的表DBS和SDS内容，除非你的集群名字或者HA的名字跟之前的一致这个就不用修改了
登录mysql数据库，查看：

```
mysql> use hive;
mysql> select * from DBS;
mysql> select * from SDS;
```

## Create/restore a snapshot of an HDFS directory

创建一个测试文件夹和文件

```
[training@elephant ~]$ hdfs dfs -mkdir /user/training/snapshottest
[training@elephant ~]$ hdfs dfs -put /etc/hosts /user/training/snapshottest/
[training@elephant ~]$ hdfs dfs -ls /user/training/snapshottest/
Found 1 items
-rw-r--r--   3 training supergroup        251 2018-01-11 14:03 /user/training/snapshottest/hosts
[training@elephant ~]$
```
HOME -> Cluster -> HDFS -> File Browser -> /user/training/snapshottest/ 点击 Enable Snapshot

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Snapshot/Snapshot1.png)

启用

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Snapshot/Snapshot3.png)

Take Sanpshot 并命名 

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Snapshot/Snapshot4.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Snapshot/Snapshot5.png)

在 Snapshots 下可以看到已有的Sanpshots

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Snapshot/Snapshot6.png)

或者到Hadoop 50070上查看

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Snapshot/Snapshot7.png)

删除测试文件 

```
[training@elephant ~]$ hdfs dfs -rm /user/training/snapshottest/hosts
18/01/11 14:09:37 INFO fs.TrashPolicyDefault: Moved: 'hdfs://mycluster/user/training/snapshottest/hosts' to trash at: hdfs://mycluster/user/training/.Trash/Current/user/training/snapshottest/hosts
[training@elephant ~]$
```

从snapshot恢复

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Snapshot/Snapshot8.png)

选择使用得分snapshot，并恢复

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Snapshot/Snapshot10.png)

查看文件是否已经恢复

```
[training@elephant ~]$ hdfs dfs -ls /user/training/snapshottest/
Found 1 items
-rw-r--r--   3 hdfs supergroup        251 2018-01-11 14:12 /user/training/snapshottest/hosts
[training@elephant ~]$
```

## Get/set ACLs for a file or directory structure

```
[training@elephant ~]$ hdfs dfs -ls /user/training/ ## 查看文件夹内内容
Found 1 items
drwx------   - training supergroup          0 2018-01-11 08:00 /user/training/.Trash
[training@elephant ~]$ hdfs dfs -put /etc/hosts /user/training/  ## 上传一个文件
[training@elephant ~]$ hdfs dfs -ls /user/training/ ## 查看上传的文件的权限
Found 2 items
drwx------   - training supergroup          0 2018-01-11 08:00 /user/training/.Trash
-rw-r--r--   3 training supergroup        251 2018-01-11 13:51 /user/training/hosts
[training@elephant ~]$ hdfs dfs -getfacl /user/training/hosts  ## 使用命令查询文件的ACL 
# file: /user/training/hosts
# owner: training
# group: supergroup
user::rw-
group::r--
other::r--

[training@elephant ~]$ hdfs dfs -getfacl /user/training/  ## 查看文件夹的ACL
# file: /user/training
# owner: training
# group: supergroup
user::rwx
group::r-x
other::r-x

[training@elephant ~]$
```
## Benchmark the cluster (I/O, CPU, network)
