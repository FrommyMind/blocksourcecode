---
title: CCAH-131 Required Skills （考试要求）
date: 2018-05-12 12:03:44
tags: 
    - CCAH-131
---
# Required Skills （技能）

## Install （安装）

Demonstrate an understanding of the installation process for Cloudera Manager, CDH, and the ecosystem projects. （熟悉Cloudera Manager、CDH和其他生态系统内的项目的安装过程）
* Set up a local CDH repository （设置一个本地的CDH仓库）
* Perform OS-level configuration for Hadoop installation （操作系统级别的配置准备）
* Install Cloudera Manager server and agents （安装 Cloudera Manager 服务和代理）
* Install CDH using Cloudera Manager （使用Cloudera Manager 安装CDH）
* Add a new node to an existing cluster （在集群上添加一个新的节点）
* Add a service using Cloudera Manager （使用Cloudera Manager添加一个新的服务）
<!-- more -->

## Configure （配置）

Perform basic and advanced configuration needed to effectively administer a Hadoop cluster （为了高效管理集群的基础和高级配置）
* Configure a service using Cloudera Manager（使用Cloudera Manager 配置一个集群）
* Create an HDFS user's home directory （创建用户的HDFS home文件夹）
* Configure NameNode HA （配置Hadoop NameNode的高可用）
* Configure ResourceManager HA （配置Yarn的ResourceManager的高可用）
* Configure proxy for Hiveserver2/Impala（配置HiveServer2或者Impala的代理）

## Manage （管理）

Maintain and modify the cluster to support day-to-day operations in the enterprise (维护或者修改集群已满足企业的日常使用）
* Rebalance the cluster（重新平衡集群）
* Set up alerting for excessive disk fill （设置磁盘使用预警）
* Define and install a rack topology script （定义和安装机架拓扑）
* Install new type of I/O compression library in cluster（在集群中安装新的读写压缩包）
* Revise YARN resource assignment based on user feedback（根据用户的反馈配置YARN的资源）
* Commission/decommission a node（服役/退役一个节点）

## Secure （安全）

Enable relevant services and configure the cluster to meet goals defined by security policy; demonstrate knowledge of basic security practices（可以根据安全要求进行服务安全配置）
* Configure HDFS ACLs （配置HDFS的ACLs）
* Install and configure Sentry（安装配置Sentry）
* Configure Hue user authorization and authentication（配置Hue的用户认证）
* Enable/configure log and query redaction（启用或者配置log和查询的修改）
* Create encrypted zones in HDFS（在HDFS上创建加密区间）

## Test （测试）

Benchmark the cluster operational metrics, test system configuration for operation and efficiency（对集群的性能进行基准测试）
* Execute file system commands via HTTPFS （通过HTTPFS执行文件系统命令）
* Efficiently copy data within a cluster/between clusters （集群间的数据复制）
* Create/restore a snapshot of an HDFS directory（创建或者恢复HDFS文件夹得快照）
* Get/set ACLs for a file or directory structure（获取/设置一个文件或者文件夹的ALCs）
* Benchmark the cluster (I/O, CPU, network)（测试集群的读写、CPU和网络性能）

## Troubleshoot （定位错误）

Demonstrate ability to find the root cause of a problem, optimize inefficient execution, and resolve resource contention scenarios（证明有找到问题根本错误的能力，优化慢的执行任务和解决实际问题的能了）
* Resolve errors/warnings in Cloudera Manager （解决Cloudera Manager上的错误或者警告）
* Resolve performance problems/errors in cluster operation（解决集群操作上性能问题）
* Determine reason for application failure（定位应用失败的原因）
* Configure the Fair Scheduler to resolve application delays（配置公平调度用以解决应用的提交延迟）
