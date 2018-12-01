---
title: CCAH-131 Configure
date: 2018-05-13 08:34:26
tags: CCAH-131
---
# Configure

 Perform basic and advanced configuration needed to effectively administer a Hadoop cluster
##  Configure a service using Cloudera Manager

**使用Cloudera Manager配置YARN服务**

HOME -> CLusters -> YARN(MR2 Included) -> Configuration 到修改配置页面

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Configuration%20Services/configuration%20yarn%20services.png)

可以在搜索框中使用模糊搜索或者在左侧的 Filters中选择相应的属性进行过滤

<!-- more -->

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Configuration%20Services/configuration%20yarn%20services%202.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Configuration%20Services/configuration%20yarn%20services%203.png)

输入更多的条件，过滤到相要修改的属性。修改属性值并保存。

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Configuration%20Services/configuration%20yarn%20services%204.png)

保存后会有重启图标 

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Configuration%20Services/configuration%20yarn%20services%205.png)

或者在HOME页面，也可以看到哪些服务受到影响，并且需求重启。

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Configuration%20Services/configuration%20yarn%20services%206.png)

点击其中一个重启，会给出需要修改的配置文件里面修改的内容，红色背景代表去掉这行，绿色背景代表添加这行。点击 Restart Stale Servers进入下一步

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Configuration%20Services/configuration%20yarn%20services%207.png)

勾选 Re-deploy client configuration 并重启

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Configuration%20Services/configuration%20yarn%20services%208.png)

进入重启 

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Configuration%20Services/configuration%20yarn%20services%209.png)

重启完成

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Configuration%20Services/configuration%20yarn%20services%2010.png)
##  Create an HDFS user's home directory

```
[training@elephant ~]$ sudo -u hdfs hdfs dfs -ls /user  ## 查看hdfs上 /user目录下当前的文件
Found 2 items
drwxrwxrwx   - mapred hadoop          0 2018-01-10 16:29 /user/history
drwxr-x--x   - spark  spark           0 2018-01-10 16:36 /user/spark
[training@elephant ~]$ sudo -u hdfs hdfs dfs -mkdir  /user/training  ## 创建training文件夹
[training@elephant ~]$ sudo -u hdfs hdfs dfs -chown training /user/training ## 修改其权限
[training@elephant ~]$ hdfs dfs -ls /user/training 
[training@elephant ~]$ hdfs dfs -ls /user/  ## 查看新创建的文件夹
Found 3 items
drwxrwxrwx   - mapred   hadoop              0 2018-01-10 16:29 /user/history
drwxr-x--x   - spark    spark               0 2018-01-10 16:36 /user/spark
drwxr-xr-x   - training supergroup          0 2018-01-10 16:57 /user/training
[training@elephant ~]$
```
##  Configure NameNode HA

HOME -> Cluster -> HDFS -> Actions -> Enable High Availability 点击 

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/HDFS%20HA/HDFS%20HA%201.png)

填入namespace的名称

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/HDFS%20HA/HDFS%20HA%202.png)

选择要添加的新的namenode所在的服务器，并添加奇数个Journal Nodes。

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/HDFS%20HA/HDFS%20HA%203.png)

给定Journal Edits所在的文件夹

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/HDFS%20HA/HDFS%20HA%204.png)

进入配置重启阶段

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/HDFS%20HA/HDFS%20HA%205.png)

因为HDFS在之前是初始化过的，此处初始化会失败。正常就是都会失败。

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/HDFS%20HA/HDFS%20HA%206.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/HDFS%20HA/HDFS%20HA%207.png)

配置成功后，会提示如果Hive中在HDFS启用HA之前有数据，则需要对Hive执行 Update Hive Metastore NameNodes

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/HDFS%20HA/HDFS%20HA%208.png)

查看配置成功后的HDFS 服务， SecondryNamenode被删除，2个Namenode一个是Active 一个是Standby，另外有3个JournalNode

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/HDFS%20HA/HDFS%20HA%209.png)
##  Configure ResourceManager HA

HOME -> Cluster -> YARN(MR2 Included) -> Instances -> Add Role Instances

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/Resource%20Manager%20HA/Resource%20Manager%20HA.png)

选择添加一个Resource Manager

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/Resource%20Manager%20HA/Resource%20Manager%20HA%202.png)

会有红色的配置警告，将Zookeepr勾选为 Zookeeper 服务。

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/Resource%20Manager%20HA/Resource%20Manager%20HA%203.png)

会返回到YARN的实例界面，有重启选项

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/Resource%20Manager%20HA/Resource%20Manager%20HA%204.png)

选择重启，会弹出修改的配置文件内容 红色的为删除的内容，绿色的为添加的内容

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/Resource%20Manager%20HA/Resource%20Manager%20HA%205.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/Resource%20Manager%20HA/Resource%20Manager%20HA%206.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/Resource%20Manager%20HA/Resource%20Manager%20HA%207.png)

选择要重启的服务，因为 Spark、Hive、Oozie、Hue都依赖于YARN，如果YARN配置了HA，这些依赖的服务也要重新更新配置。 所以勾选为需要重启的服务。

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/Resource%20Manager%20HA/Resource%20Manager%20HA%208.png)

重启服务

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/Resource%20Manager%20HA/Resource%20Manager%20HA%209.png)

重启完成

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/Resource%20Manager%20HA/Resource%20Manager%20HA%2010.png)

查看服务实例 有2个Resource Manager 一个是Active状态一个是Standby状态。

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HA/Resource%20Manager%20HA/Resource%20Manager%20HA%2011.png)
##  Configure proxy for Hiveserver2/Impala

HOME -> Cluster -> Hive -> Configuration 搜索load balance

填写load balance的服务器的域名

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HiveAndImpalaLoadBalance/HiveLoadBalance1.png)

添加 高级配置 

```
Name: hive.server2.support.dynamic.service.discovery
Value: true
```

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HiveAndImpalaLoadBalance/HiveLoadBalance2.png)

停掉一个 hiveserver2 

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HiveAndImpalaLoadBalance/HiveLoadBalance3.png)

使用如下代码，测试连接

```
beeline -u "jdbc:hive2://elephant:2181,horse:2181,tiger:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2"
```

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HiveAndImpalaLoadBalance/HiveLoadBalance4.png)






### Impala Proxy

**安装Haproxy**

[Example of Configuring HAProxy Load Balancer for Impala](https://www.cloudera.com/documentation/enterprise/latest/topics/impala_proxy.html#tut_proxy)

参考：http://blog.csdn.net/lsb2002/article/details/53843340
	  http://blog.csdn.net/aa168b/article/details/50372649
	  
下载：haproxy：http://www.haproxy.org/download/1.7/src/haproxy-1.7.1.tar.gz

解压 

```
tar -xvf haproxy-1.7.1.tar
```

编译和安装

```
  sudo make TARGET=linux31 PREFIX=/usr/local/haproxy
  sudo make install PREFIX=/usr/local/haproxy
  cd /usr/local/haproxy/
  sudo mkdir -p /usr/haproxy/
  sudo vim /etc/haproxy/haproxy.cfg
```
将下面的内容添加到 /etc/haproxy/haproxy.cfg

```
global
    # To have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local0
    log         127.0.0.1 local1 notice
    #chroot      /usr/local/haproxy
    pidfile     /usr/local/haproxy/logs/haproxy.pid
    maxconn     4000
    #uid      501
    #gid 501
    daemon

    # turn on stats unix socket
    #stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#
# You might need to adjust timing values to prevent timeouts.
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    #option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    maxconn                 3000
    timeout connect 5000
    timeout check 20000
    timeout client 50000
    timeout server 50000

#
# This sets up the admin page for HA Proxy at port 25002.
#
listen stats
    bind 0.0.0.0:25002
    balance
    mode http
    stats enable
    stats auth admin:admin

# This is the setup for Impala. Impala client connect to load_balancer_host:25003.
# HAProxy will balance connections among the list of servers listed below.
# The list of Impalad is listening at port 21000 for beeswax (impala-shell) or original ODBC driver.
# For JDBC or ODBC version 2.x driver, use port 21050 instead of 21000.
listen impala
    bind 0.0.0.0:25002
    mode tcp
    option tcplog
    balance leastconn

    server symbolic_name_1 elephant:21000
    server symbolic_name_2 tiger:21000
    server symbolic_name_3 monkey:21000
    server symbolic_name_4 horse:21000

# Setup for Hue or other JDBC-enabled applications.
# In particular, Hue requires sticky sessions.
# The application connects to load_balancer_host:21051, and HAProxy balances
# connections to the associated hosts, where Impala listens for JDBC
# requests on port 21050.
listen impalajdbc
    bind 0.0.0.0:21051
    mode tcp
    option tcplog
    balance source
    server symbolic_name_5 elephant:21050
    server symbolic_name_6 tiger:21050
    server symbolic_name_7 monkey:21050
    server symbolic_name_8 horse:21050
```

启动 haproxy

```
sudo /usr/local/haproxy/sbin/haproxy -f /etc/haproxy/haproxy.cfg
```


impala 访问

```
[training@elephant ~]$ beeline -d "com.cloudera.impala.jdbc41.Driver" -u "jdbc:impala://elephant:21051"
2018-01-12 14:35:57,602 WARN  [main] mapreduce.TableMapReduceUtil: The hbase-prefix-tree module jar containing PrefixTreeCodec is not present.  Continuing without it.
Connecting to jdbc:impala://elephant:21051
com.cloudera.impala.jdbc41.Driver
Beeline version 1.1.0-cdh5.9.0 by Apache Hive
0: jdbc:impala://elephant:21051 (closed)>
```

停掉 elephant上的impala daemon，再次测试

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HiveAndImpalaLoadBalance/ImpalaLoadBalance1.png)

```
[training@elephant ~]$ beeline -d "com.cloudera.impala.jdbc41.Driver" -u "jdbc:impala://elephant:21051"
2018-01-12 14:35:57,602 WARN  [main] mapreduce.TableMapReduceUtil: The hbase-prefix-tree module jar containing PrefixTreeCodec is not present.  Continuing without it.
Connecting to jdbc:impala://elephant:21051
com.cloudera.impala.jdbc41.Driver
Beeline version 1.1.0-cdh5.9.0 by Apache Hive
0: jdbc:impala://elephant:21051 (closed)>
```
