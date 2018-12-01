---
title: Cloudera Manager API的简单实用
date: 2018-10-23 18:08:57
tags:
- CM
---

参考： https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cm_intro_api.html
## Alter 告警

Alter是从event中获取的。
event的元数据

|property|  type|   描述|
|:-----|:------|:-------|
|id|    id (string)|    事件的唯一ID.
|content    |content (string)   |事件内容描述
|timeOccurred   |timeOccurred (dateTime)    |事件发生事件
|timeReceived   |timeReceived (dateTime)    |Cloudera Manager 获取事件的时间. 事件并不是顺时到达的. 
|category   |category (apiEventCategory)    |事件的分类 -- 健康的事件，审计事件或者操作时间等 UNKNOWN：未知分类；HEALTH_EVENT：健康事件；LOG_EVENT：日志事件；AUDIT_EVENT：审计事件；ACTIVITY_EVENT：活动事件；HBASE：HBase事件；SYSTEM：系统事件；
|severity   |severity (apiEventSeverity)    |事件的严重性 UNKNOWN：未知；INFORMATIONAL：状态修改；IMPORTANT：需要注意的事件；CRITICAL：严重，需要立即解决
|alert  |alert (boolean)    |事件是否需要晋级
|attributes |array of attributes/attributes (apiEventAttribute) |属性列表

<!-- more -->

查看所有事件
http://< *cloudera manager host* >:7180/api/v19/events
查看所有警告事件 
http://< *cloudera manager host* >:7180/api/v19/events/?query=alert==true
查看严重的警告事件
http://< *cloudera manager host* >:7180/api/v19/events/?query=alert==true;severity==CRITICAL
查看一个时间区间范围内的严重警告事件
http://< *cloudera manager host* >:7180/api/v19/events/?query=alert==true;severity==critical;timeReceived=ge=2018-10-16T00:00;timeReceived=lt=2018-10-18T00:10
查看指定eventid的事件内容
http://< *cloudera manager host* >:7180/api/v19/events/272baa75-0fe9-4fd9-ab27-f4e333fd524

## 监控报表
[tsquery Language](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_dg_tsquery.html)

http://< *cloudera manager host* >:7180/api/v19/timeseries/?query=select%20*%20where%20roleType=DATANODE

![Alt text](/img/1540285620938.png)


http://< *cloudera manager host* >:7180/api/v19/timeseries/?query=select%20cpu_user_rate%20where%20roleType=DATANODE
![Alt text](/img/1540285662271.png)


http://< *cloudera manager host* >:7180/api/v19/timeseries/?query=select%20await_time,%20await_read_time,%20await_write_time,%20250%20where%20category=disk
![Alt text](/img/1540285688689.png)

##  如何获取Cloudera Manager已有报表的数据
1. 打开对应报表所在的页面
![Alt text](/img/1540285723630.png)
2. 点击在报表的右上角上的工具选项
![Alt text](/img/1540286887142.png)
3. 选择弹出框的Open in Chart Builder
![Alt text](/img/1540286907954.png)
4. 复制查询语句
![Alt text](/img/1540286924463.png)
5. 在浏览器或其他restful api工具上执行查询。
http://< *cloudera manager host* >:7180/api/v19/timeseries/?query=select%20cpu_percent_across_hosts%20where%20category%20=%20CLUSTER

![Alt text](/img/1540287176183.png)




