---
title: CDH 安装环境需求
date: 2018-05-13 10:10:26
tags: CDH
---
# CDH 安装环境需求

https://www.cloudera.com/documentation/enterprise/release-notes/topics/rn_consolidated_pcm.html#concept_mh3_sht_kbb
## 操作系统
**Red Hat/CentOS** 7.3, 7.2, 7.1,6.9, 6.8, 6.7, 6.5

**SUSE** 12 SP1, 11 SP4, 11 SP3, 11 SP2

**Ubuntu** 14.04,12.04
## 文件系统
挂载时不带atime，可以加速读取

<!-- more -->
```
/dev/sdb1 /data1 ext4 defaults,noatime 0
mount -o remount /data1
```
ext3,ext4，XFS

## 机器
内存： > 4G 

IPv6：不支持，而且必须关闭 

端口： 7180，8088,8888,19888(根据实际需求增加）
## 硬盘空间
 /var > 20G 

 /usr > 500M

 /opt > 10G 
## yum源
挂载硬盘镜像并配置本地yum源
## Others

所有的数据库使用UTF-8编码。

**MySQL** 5.5,5.6,5.7

**MariaDB** 5.5, 10.0

**PostgreSQL** 8.1,8.3,8.4,9.1,9.2,9.3,9.4

**Oracle** 11g R2,12c R1

**JDK** 1.7u80,1.7u75,1.7u67,1.7u55,1.8u121,1.8u111，1.8u102，1.8u91，1.8u74，1.8u31 （如果需要安装Spark2.x 则需要JDK 1.8）

**浏览器** Chrome，Firefox，Internet Explorer，Safari 

## 注意项
* Cloudera Manager对应的CDH版本，el6对应于el6的版本。
	CDH的大版本不能比CM的大版本大。
* 所有的CDH节点必须允许统一版本的操作系统。
* Kudu 只可以允许在ext4 和 XFS的文件系统上。
* 必须修改/etc/sysconfig/network里面的host名和/etc/hosts一致**
* /etc/hosts 里不允许有大写的域名

