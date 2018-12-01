---
title: CCAH-131 Manage
date: 2018-05-13 08:35:26
tags: CCAH-131
---
# Manage

Maintain and modify the cluster to support day-to-day operations in the enterprise
##  Rebalance the cluster

[Configuring and Running the HDFS Balancer Using Cloudera Manager](https://www.cloudera.com/documentation/enterprise/5-12-x/topics/admin_hdfs_balancer.html#cmug_topic_5_14)

HOME -> Clusters -> HDFS -> Configuration 在 SCOPE 选择 Balancer，在 CATEGORY里选择main。设置Rebalancing Threshold

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Rebalance/Rebalance%201.png)

修改DataNode Advanced Configuration Snippet (Safety Valve) 的值。
<!-- more -->

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Rebalance/Rebalance%202.png)

输入内容

```
<property>
  <name>dfs.datanode.balance.max.concurrent.moves</name>
  <value>50</value>
</property>
```

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Rebalance/Rebalance%202.1.png)

执行 Rebalance

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Rebalance/Rebalance%203.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Rebalance/Rebalance%204.png)

Rebalance log 

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Rebalance/Rebalance%205.png)

##  Set up alerting for excessive disk fill

### 配置邮箱
Administrator -> Alerts 

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Alerts/Add%20Email%201.png)

点击 Edit 

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Alerts/Add%20Email%202.png)

修改 Alerts: Mail Message Recipients

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Alerts/Add%20Email%203.png)

或者搜索email进行相关设置修改

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Alerts/Add%20Email%204.png)


### 修改警告项配置

选择一个服务进入configuration 在左侧选择 monitoring ，然后可以修改监控选项的 warning和critial值。

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Alerts/Add%20Email%205.png)



##  Define and install a rack topology script

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Rack/Rack%201.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Rack/Rack%202.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Rack/Rack%203.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Rack/Rack%204.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Rack/Rack%205.png)

##  Install new type of I/O compression library in cluster
http://archive.cloudera.com/gplextras5/parcels/5.9.0/

[Configuring Services to Use the GPL Extras Parcel](https://www.cloudera.com/documentation/enterprise/5-12-x/topics/cm_mc_gpl_extras.html#xd_583c10bfdbd326ba--6eed2fb8-14349d04bee--7c3e)

HOME -> Clusters -> HDFS -> Configuration 搜索 compression 
点击 + 添加新的压缩类。（Note: 操作系统已经支持此压缩）

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20Compression%20Codec/AddCompressionCodec1.png)
 
##  Revise YARN resource assignment based on user feedback

##  Commission/decommission a node

**Decommission**

HOME -> Hosts -> Roles  查看要退役节点上的服务

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode1.png)

选择节点，停止上面的所有服务

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode2.png)

停止上面所有的服务
	
![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode3.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode4.png)

查看节点上的服务是否已经都停止

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode5.png)

将节点从集群里移除

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode6.png)

确认 

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode7.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode8.png)

查看节点，已经没有角色了

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode9.png)

退役节点

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode10.png)

确认

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode11.png)

成功退役

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode12.png)

查看节点的服役状态，已经为 Decommission


![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/decommissionNode13.png)

**Commission**

选择退役的节点，点击重启服役

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/recommissionNode1.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/recommissionNode2.png)

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/recommissionNode3.png)

查看节点的服役状态，已经为 Commission

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/DecommisionNode/recommissionNode4.png)
