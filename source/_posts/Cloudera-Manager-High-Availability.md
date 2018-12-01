---
title: Cloudera Manager High Availability
date: 2018-10-22 11:28:39
tags: 
- CDH安装
---

## 1. 环境准备
### 1.1 环境
操作系统：CentOS Linux release 7.3.1611 (Core)
JDK：jdk1.8.0_111
服务器：53-58 60-61 共8台

### 1.2 软件下载
HAProxy: http://www.haproxy.org/download/1.8/src/haproxy-1.8.13.tar.gz

<!-- more --> 

### 1.3 架构
![enter image description here](https://www.cloudera.com/documentation/enterprise/5-4-x/images/cm_ha_lb_setup.png)

### 1.4 修改主机名称
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.17.1.53 cms1.test.com cms1
172.17.1.54 cms2.test.com cms2
172.17.1.55 mgmt1.test.com mgmt1
172.17.1.56 mgmt2.test.com mgmt2
172.17.1.57 nfs.test.com nfs
172.17.1.58 cms.test.com cms
172.17.1.60 mgmt.test.com mgmt
172.17.1.61 dn1.test.com dn1
```


## 2 安装前环境准备
### 2.1 创建主和副主机
Cloudera Manager Server and Cloudera Management Service Primary host cms1.test.com
Cloudera Manager Server and Cloudera Management Service Secondary host cms2.test.com

此外，Cloudera建议：
    不要在安装CDH的节点安装Cloudera Manager或者Cloudera Management Service，因为这样会使failover的配置变的复杂。并且覆盖失败的域名可能会造成容错和错误检查的问题。
    对主副主机都使用相同的主机配置。用来保证故障转移后性能不会降低。
    对主副主机使用分开的主机和网络组件。

主机分配

| __IP__ | __DomainName__ | __功能__ | __角色__|
|-----------|-----------|-----------|-----------|
|172.17.1.53|cms1.test.com| cloudera manager 主节点|cms,cma|
|172.17.1.54|cms2.test.com| cloudera manager 备份节点|cms,cma|
|172.17.1.55|mgmt1.test.com|cloudera management service 主节点|cmmg,cma|
|172.17.1.56|mgmt2.test.com|cloudera managerment service 备份节点|cmmg,cma|
|172.17.1.57|nfs.test.com|挂载存储服务器|nfs|
|172.17.1.58|cms.test.com|cm代理服务器|haproxy|
|172.17.1.60|mgmt.test.com|mgmt代理服务器|haproxy|
|172.17.1.61|dn1.test.com|数据节点|cma,数据库|


### 2.2 安装配置Load Balancer
在`cms`节点上安装HAProxy
```
yum install -y haproxy
```
配置haproxy开机启动
```
systemctl enable haproxy
```
配置HAProxy
在`cms`上，编辑*/etc/haproxy/haproxy.cfg*文件，添加需要代理的端口

```
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
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
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
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
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
#frontend  main *:5000
#    acl url_static       path_beg       -i /static /images /javascript /stylesheets
#    acl url_static       path_end       -i .jpg .gif .png .css .js
#
#    use_backend static          if url_static
#    default_backend             app

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
#backend static
#    balance     roundrobin
#    server      static 127.0.0.1:4331 check

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
#backend app
#    balance     roundrobin
#    server  app1 127.0.0.1:5001 check
#    server  app2 127.0.0.1:5002 check
#    server  app3 127.0.0.1:5003 check
#    server  app4 127.0.0.1:5004 check

#---------------------------------------------------------------------
# stats port
#---------------------------------------------------------------------
listen stats
    bind 0.0.0.0:1080
    mode http
    option httplog
    maxconn 5000
    stats refresh 30s
    stats  uri /stats

#---------------------------------------------------------------------
# Cloudera Manager
#---------------------------------------------------------------------

listen cmf
    mode tcp
    option tcplog
    bind 0.0.0.0:7180
    server cmfhttp1 cms1.test.com:7180 check
    server cmfhttp2 cms2.test.com:7180 check

listen cmfavro :7182
    mode tcp
    option tcplog
    server cmfavro1 cms1.test.com:7182 check
    server cmfavro2 cms2.test.com:7182 check

#ssl pass-through, without termination
listen cmfhttps :7183
    mode tcp
    option tcplog
    server cmfhttps1 cms1.test.com:7183 check
    server cmfhttps2 cms2.test.com:7183 check

listen mgmt1 :5678
    mode tcp
    option tcplog
    server mgmt1a cms1.test.com check
    server mgmt1b cms2.test.com check

listen mgmt2 :7184
    mode tcp
    option tcplog
    server mgmt2a cms1.test.com check
    server mgmt2b cms2.test.com check

listen mgmt3 :7185
    mode tcp
    option tcplog
    server mgmt3a cms1.test.com check
    server mgmt3b cms2.test.com check
listen mgmt4 :7186
    mode tcp
    option tcplog
    server mgmt4a cms1.test.com check
    server mgmt4b cms2.test.com check
listen mgmt5 :7187
    mode tcp
    option tcplog
    server mgmt5a cms1.test.com check
    server mgmt5b cms2.test.com check

listen mgmt6 :8083
    mode tcp
    option tcplog
    server mgmt6a cms1.test.com check
    server mgmt6b cms2.test.com check
listen mgmt7 :8084
    mode tcp
    option tcplog
    server mgmt7a cms1.test.com check
    server mgmt7b cms2.test.com check
listen mgmt8 :8086
    mode tcp
    option tcplog
    server mgmt8a cms1.test.com check
    server mgmt8b cms2.test.com check
listen mgmt9 :8087
    mode tcp
    option tcplog
    server mgmt9a cms1.test.com check
    server mgmt9b cms2.test.com check
listen mgmt10 :8091
    mode tcp
    option tcplog
    server mgmt10a cms1.test.com check
    server mgmt10b cms2.test.com check
listen mgmt-agent :9000
    mode tcp
    option tcplog
    server mgmt-agenta cms1.test.com check
    server mgmt-agentb cms2.test.com check
listen mgmt11 :9994
    mode tcp
    option tcplog
    server mgmt11a cms1.test.com check
    server mgmt11b cms2.test.com check
listen mgmt12 :9995
    mode tcp
    option tcplog
    server mgmt12a cms1.test.com check
    server mgmt12b cms2.test.com check
listen mgmt13 :9996
    mode tcp
    option tcplog
    server mgmt13a cms1.test.com check
    server mgmt13b cms2.test.com check
listen mgmt14 :9997
    mode tcp
    option tcplog
    server mgmt14a cms1.test.com check
    server mgmt14b cms2.test.com check
listen mgmt15 :9998
    mode tcp
    option tcplog
    server mgmt15a cms1.test.com check
    server mgmt15b cms2.test.com check
listen mgmt16 :9999
    mode tcp
    option tcplog
    server mgmt16a cms1.test.com check
    server mgmt16b cms2.test.com check
listen mgmt17 :10101
    mode tcp
    option tcplog
    server mgmt17a cms1.test.com check
    server mgmt17b cms2.test.com check
```
重启haproxy

```
systemctl restart  haproxy
```
### 2.3 安装配置数据库
在dn1节点上安装数据库
```
yum install -y mariadb mariadb-server
```
启动数据库
```
systemctl restart mariadb
```

初始化
```
/usr/bin/mysql_secure_installation
```

配置mariadb的主从 [参考](https://mariadb.com/kb/en/library/setting-up-replication/)
在主节点上的/etc/my.cnf添加如下配置并重启
```
[mariadb]
log-bin
server_id=1
log-basename=master1
```
创建replication用户
```
CREATE USER 'replication_user'@'%' IDENTIFIED BY 'bigs3cret';
GRANT REPLICATION SLAVE ON *.* TO 'replication_user'@'%';
```
在从节点上修改/etc/my.cnf文件,并重启
```
[mariadb]
log-bin
server_id=2
log-basename=slave1
```
获取主节点的Binary Log
在主节点上执行命令锁住所有的表
```
FLUSH TABLES WITH READ LOCK
```
执行命令获取主节点的状态，并记住Position的值
```
MariaDB [(none)]> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000004 |      495 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)

MariaDB [(none)]>
```

在从节点执行如下命令：
```
CHANGE MASTER TO
  MASTER_HOST='cms1.test.com',
  MASTER_USER='replication_user',
  MASTER_PASSWORD='bigs3cret',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='mysql-bin.000004',
  MASTER_LOG_POS=495,
  MASTER_CONNECT_RETRY=10;
```
 
 在主节点执行命令
```
UNLOCK TABLES;
```

在从节点执行命令启动slave
```
START SLAVE;
```
使用命令查询是否配置争取
```
MariaDB [(none)]> show slave status \G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: cms1.test.com
                  Master_User: replication_user
                  Master_Port: 3306
                Connect_Retry: 10
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 495
               Relay_Log_File: slave1-relay-bin.000005
                Relay_Log_Pos: 529
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 495
              Relay_Log_Space: 824
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
1 row in set (0.00 sec)
```
Slave_IO_Running 和 Slave_SQL_Running应该是yes

在主节点上执行如下命令来创建数据库和用户
```
mysql -u root --password='123456' -e 'create database metastore default character set utf8;'
mysql -u root --password='123456' -e "CREATE USER 'hive'@'%' IDENTIFIED BY '123456';"
mysql -u root --password='123456' -e "GRANT ALL PRIVILEGES ON metastore. * TO 'hive'@'%';"
mysql -u root --password='123456' -e "create user 'amon'@'%' identified by '123456'" 
mysql -u root --password='123456' -e 'create database amon default character set utf8' 
mysql -u root --password='123456' -e "grant all privileges on amon.* to 'amon'@'%'"
mysql -u root --password='123456' -e "create user 'rman'@'%' identified by '123456'" 
mysql -u root --password='123456' -e 'create database rman default character set utf8' 
mysql -u root --password='123456' -e "grant all privileges on rman.* to 'rman'@'%'"
mysql -u root --password='123456' -e "create user 'sentry'@'%' identified by '123456'" 
mysql -u root --password='123456' -e 'create database sentry default character set utf8' 
mysql -u root --password='123456' -e "grant all privileges on sentry.* to 'sentry'@'%'"
mysql -u root --password='123456' -e "create user 'nav'@'%' identified by '123456'" 
mysql -u root --password='123456' -e 'create database nav default character set utf8' 
mysql -u root --password='123456' -e "grant all privileges on nav.* to 'nav'@'%'"
mysql -u root --password='123456' -e "create user 'navms'@'%' identified by '123456'" 
mysql -u root --password='123456' -e 'create database navms default character set utf8'
mysql -u root --password='123456' -e "grant all privileges on navms.* to 'navms'@'%'"
mysql -u root --password='123456' -e "create user 'cm'@'%' identified by '123456'" 
mysql -u root --password='123456' -e 'create database cm default character set utf8' 
mysql -u root --password='123456' -e "grant all privileges on cm.* to 'cm'@'%'"
mysql -u root --password='123456' -e "create user 'oozie'@'%' identified by '123456'" 
mysql -u root --password='123456' -e 'create database oozie default character set utf8' 
mysql -u root --password='123456' -e "grant all privileges on oozie.* to 'oozie'@'%'"
mysql -u root --password='123456' -e "create user 'hue'@'%' identified by '123456'" 
mysql -u root --password='123456' -e 'create database hue default character set utf8' 
mysql -u root --password='123456' -e "grant all privileges on hue.* to 'hue'@'%'"
mysql -u root --password='123456' -e "FLUSH PRIVILEGES;"
```


### 2.4 安装配置NFS Server

在nfs.test.com节点上安装NFS
```
yum install nfs-utils nfs-utils-lib
```
启动nfs和rpcbind
```
systemctl enable nfs
systemctl start rpcbind
systemctl start nfs
```

创建nfs文件目录
```
mkdir -p /media/cloudera-scm-server
```
在文件/etc/exports中添加如下的内容
```
/media/cloudera-scm-server cms1(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-scm-server cms2(rw,sync,no_root_squash,no_subtree_check)
```
执行mounts
```
exportfs -a
```
在cms1和cms2上创建挂载点
```
rm -rf /var/lib/cloudera-scm-server
mkdir -p /var/lib/cloudera-scm-server
```
安装nfs工具并执行挂载命令
其中nfs-utils是在centos7上有效
```
yum install nfs-utils-lib 
yum install nfs-utils
mount -t nfs nfs.test.com:/media/cloudera-scm-server /var/lib/cloudera-scm-server
```
重启rpcbind
```
systemctl restart  rpcbind
```
修改/etc/fstab文件，使用其永久有效
```
nfs.test.com:/media/cloudera-scm-server /var/lib/cloudera-scm-server nfs auto,noatime,nolock,intr,tcp,actimeo=1800 0 0
```

查看
```
cms1:~>df -h
文件系统                                 容量  已用  可用 已用% 挂载点
/dev/vda2                                 90G  6.2G   84G    7% /
devtmpfs                                 7.5G     0  7.5G    0% /dev
tmpfs                                    7.5G     0  7.5G    0% /dev/shm
tmpfs                                    7.5G  8.4M  7.5G    1% /run
tmpfs                                    7.5G     0  7.5G    0% /sys/fs/cgroup
tmpfs                                    1.5G     0  1.5G    0% /run/user/0
/dev/loop0                               4.1G  4.1G     0  100% /mnt
nfs.test.com:/media/cloudera-scm-server   90G  2.1G   88G    3% /var/lib/cloudera-scm-server
```

```
cms2:~>df -h
文件系统                                 容量  已用  可用 已用% 挂载点
/dev/vda2                                 90G  2.1G   88G    3% /
devtmpfs                                 7.5G     0  7.5G    0% /dev
tmpfs                                    7.5G     0  7.5G    0% /dev/shm
tmpfs                                    7.5G  8.4M  7.5G    1% /run
tmpfs                                    7.5G     0  7.5G    0% /sys/fs/cgroup
tmpfs                                    1.5G     0  1.5G    0% /run/user/0
nfs.test.com:/media/cloudera-scm-server   90G  2.1G   88G    3% /var/lib/cloudera-scm-server
```

## 3. Cloudera Manager高可用的安装和配置

### 在主节点上安装（cms1）
安装cloudera manager server 服务
```
yum install -y cloudera-manager-server
```
配置cloudera manager server所使用的数据库
```
/usr/share/cmf/schema/scm_prepare_database.sh mysql -h dn1.test.com cm cm 123456
```
启动cloudera manager server
```
 service cloudera-scm-server start
```
验证：
http://CMS1:7180


在代理服务器上检查代理的状态
http://cms.test.com:1080/stats
并查看http://cms.test.com:7180/cmf/login 是否可用（通过代理的ip访问）

HTTP Referer 配置
Cloudera推荐禁用HTTP Referer检查，因为它可能会造成一些代理或者load balancer出错。通过如下步骤手动禁用。
![Alt text](/img/1539920764009.png)

### 在副节点上安装


```
yum install -y cloudera-manager-server
```
复制数据库配置文件
```
scp root@cms1:/etc/cloudera-scm-server/db.properties  /etc/cloudera-scm-server/
```
关闭开机启动
```
systemctl disable cloudera-scm-server
```
如果配置自动故障转移，需要在主节点也禁用自动开机启动

测试故障转移
停止cms1.test.com上的cloudera-scm-server
```
systemctl stop cloudera-scm-server
```

**等待1分钟左右**在cms2.test.com上启动cloudera-scm-server
```
systemctl start cloudera-scm-server
```
访问http://cm.test.com:1080/stats
和http://cm.test.com:7180/cmf/login 

更新cloudera manager agens使其使用load balancer (***除了cms1，cms2，mgmt1，mgmt2这几个节点***)
更新配置文件/etc/cloudera-scm-agent/config.ini，修改server_host
```
server_host = cms.test.com
```
重启agent
```
service cloudera-scm-agent restart
```

## 4. Cloudera Management Service的高可用安装和配置

### 4.1 为Cloudera Management Service 设置NFS挂载点
在NFS服务器 dn3.test.com上执行命令
```
mkdir -p /media/cloudera-host-monitor
mkdir -p /media/cloudera-scm-agent
mkdir -p /media/cloudera-scm-eventserver
mkdir -p /media/cloudera-scm-headlamp
mkdir -p /media/cloudera-service-monitor
mkdir -p /media/cloudera-scm-navigator
mkdir -p /media/etc-cloudera-scm-agent
```
在NFS服务器的/etc/exports 文件中添加如下内容
```
/media/cloudera-host-monitor mgmt1(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-scm-agent mgmt1(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-scm-eventserver mgmt1(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-scm-headlamp mgmt1(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-service-monitor mgmt1(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-scm-navigator mgmt1(rw,sync,no_root_squash,no_subtree_check)
/media/etc-cloudera-scm-agent mgmt1(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-host-monitor mgmt2(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-scm-agent mgmt2(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-scm-eventserver mgmt2(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-scm-headlamp mgmt2(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-service-monitor mgmt2(rw,sync,no_root_squash,no_subtree_check)
/media/cloudera-scm-navigator mgmt2(rw,sync,no_root_squash,no_subtree_check)
/media/etc-cloudera-scm-agent mgmt2(rw,sync,no_root_squash,no_subtree_check)
```

在NFS上执行如下命令导出挂载点
```
exportfs -a
```
在MGMT1和MGMT2节点上配置，此处仍旧使用cms1.test.com 和cms2.test.com
如果是新的节点，则需要安装nfs-utils
```
yum install nfs-utils
```
### 4.2 在`MGMT1`和`MGMT2`上创建挂载点
```
mkdir -p /var/lib/cloudera-host-monitor
mkdir -p /var/lib/cloudera-scm-agent
mkdir -p /var/lib/cloudera-scm-eventserver
mkdir -p /var/lib/cloudera-scm-headlamp
mkdir -p /var/lib/cloudera-service-monitor
mkdir -p /var/lib/cloudera-scm-navigator
mkdir -p /etc/cloudera-scm-agent
```
挂载
```
mount -t nfs nfs.test.com:/media/cloudera-host-monitor /var/lib/cloudera-host-monitor
mount -t nfs nfs.test.com:/media/cloudera-scm-agent /var/lib/cloudera-scm-agent
mount -t nfs nfs.test.com:/media/cloudera-scm-eventserver /var/lib/cloudera-scm-eventserver
mount -t nfs nfs.test.com:/media/cloudera-scm-headlamp /var/lib/cloudera-scm-headlamp
mount -t nfs nfs.test.com:/media/cloudera-service-monitor /var/lib/cloudera-service-monitor
mount -t nfs nfs.test.com:/media/cloudera-scm-navigator /var/lib/cloudera-scm-navigator
mount -t nfs nfs.test.com:/media/etc-cloudera-scm-agent /etc/cloudera-scm-agent
```
设置fstab，添加如下内容在/etc/fstab文件中
```
nfs.test.com:/media/cloudera-host-monitor /var/lib/cloudera-host-monitor nfs auto,noatime,nolock,intr,tcp,actimeo=1800 0 0
nfs.test.com:/media/cloudera-scm-agent /var/lib/cloudera-scm-agent nfs auto,noatime,nolock,intr,tcp,actimeo=1800 0 0
nfs.test.com:/media/cloudera-scm-eventserver /var/lib/cloudera-scm-eventserver nfs auto,noatime,nolock,intr,tcp,actimeo=1800 0 0
nfs.test.com:/media/cloudera-scm-headlamp /var/lib/cloudera-scm-headlamp nfs auto,noatime,nolock,intr,tcp,actimeo=1800 0 0
nfs.test.com:/media/cloudera-service-monitor /var/lib/cloudera-service-monitor nfs auto,noatime,nolock,intr,tcp,actimeo=1800 0 0
nfs.test.com:/media/cloudera-scm-navigator /var/lib/cloudera-scm-navigator nfs auto,noatime,nolock,intr,tcp,actimeo=1800 0 0
nfs.test.com:/media/etc-cloudera-scm-agent /etc/cloudera-scm-agent nfs auto,noatime,nolock,intr,tcp,actimeo=1800 0 0
```

### 4.3 在主节点上安装 agent
登陆MGMT1，安装cloudera-manager-daemons 和cloudera-manager-agent
```
yum install -y cloudera-manager-daemons cloudera-manager-agent
```
配置agent的/etc/cloudera-scm-agent/config.ini
```
server_host = cms.test.com
listening_hostname = mgmt.test.com
```
编辑/etc/hosts 文件，添加如下内容
```
172.17.1.60 dn1.test.com
```
使用ping检查
```
mgmt1:~>ping dn1.test.com
PING dn1.test.com (172.17.1.60) 56(84) bytes of data.
64 bytes from mgmt1.test.com (172.17.1.60): icmp_seq=1 ttl=64 time=0.092 ms
64 bytes from mgmt1.test.com (172.17.1.60): icmp_seq=2 ttl=64 time=0.059 ms
```

修改文件夹的权限
```
chown -R cloudera-scm:cloudera-scm /var/lib/cloudera-scm-eventserver
chown -R cloudera-scm:cloudera-scm /var/lib/cloudera-scm-navigator
chown -R cloudera-scm:cloudera-scm /var/lib/cloudera-service-monitor
chown -R cloudera-scm:cloudera-scm /var/lib/cloudera-host-monitor
chown -R cloudera-scm:cloudera-scm /var/lib/cloudera-scm-agent
chown -R cloudera-scm:cloudera-scm /var/lib/cloudera-scm-headlamp
```

重启 agent
```
service cloudera-scm-agent restart
```

![Alt text](/img/1539939416172.png)
![Alt text](/img/1539939421998.png)



### 4.4 在副节点上安装agent
在cm上停掉所有的Cloudera Management Service服务

停掉主节点的`cloudera-scm-agent` 服务
```
mgmt1:~>systemctl stop cloudera-scm-agent
mgmt1:~>
```
在副节点上安装`cloudera-manager-agent`
```
yum install -y cloudera-manager-agent
```

在cm上启动所有的`cloudera management service`

Note: *确保cloudera-scm用户UID和GID在Cloudera Management Service的主副节点上是一致的。*
![Alt text](/img/1539939380054.png)
启动Cloudera Management Service
![Alt text](/img/1539939371522.png)


## 5. 问题：

![Alt text](/img/1539939461605.png)
节点的SELINUX没有设施为disabled


----------

