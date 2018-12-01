---
title: CCAH-131 Install
date: 2018-05-13 08:33:26
tags: CCAH-131
---

## Install

Demonstrate an understanding of the installation process for Cloudera Manager, CDH, and the ecosystem projects.

### Set up a local CDH repository.  设置CDH 本地Yum源
    
#### 方法1：使用httpd默认的文件路径

检查httpd是否安装 ，如果没有安装结果如下： 

```
[daniel@tiger ~]$ sudo rpm -qa | grep httpd
[daniel@tiger ~]$
```
如果已经安装则是如下结果

```
[daniel@tiger ~]$ sudo rpm -qa | grep httpd
httpd-tools-2.2.15-60.el6.centos.6.x86_64
httpd-2.2.15-60.el6.centos.6.x86_64
[daniel@tiger ~]$
```
<!-- more -->

**安装 httpd**

```
sudo yum install -y httpd
```

启动httpd服务

```
[daniel@tiger ~]$ sudo service httpd start
Starting httpd: httpd: Could not reliably determine the server's fully qualified domain name, using 192.168.25.150 for ServerName
                                                           [  OK  ]
[daniel@tiger ~]$
```

在/var/www/html/ 下创建文件夹 cm

```
[daniel@tiger html]$ sudo mkdir /var/www/html/cm
[daniel@tiger html]$ ll /var/www/html/
total 4
drwxr-xr-x. 2 root root 4096 Jan  5 12:43 cm
[daniel@tiger html]$
```
  
将下载的Cloudera Manager 包放到 /var/www/html/的cm文件下 

sudo mv CDH-5.9.0-1.cdh5.9.0.p0.23-el5.parcel* /var/www/html/cm
#### 修改cm文件夹得权限
```
[daniel@tiger cm]$ sudo yum install -y createrepo
[daniel@tiger cm]$ sudo createrepo /var/www/html/cm
[daniel@tiger cm]$ sudo chmod -R 755 /var/www/html/cm
```
使用浏览器打开 http://hostname/cm/ 其中 hostname是ip或者服务器的hostname。

#### 配置Yum源
在/etc/yum.repo.d/创建文件 cloudera-cm.repo

```
[daniel@tiger yum.repos.d]$ cat cloudera-cm.repo
[cm]
name=Cloudera-Manager
baseurl=http://192.168.25.150/cm
gpgkey=https://archive.cloudera.com/redhat/cdh/RPM-GPG-KEY-cloudera
gpgcheck=1
enabled=1
[daniel@tiger yum.repos.d]$
```
配置完成后，更新本地yum的cache，并查询是否配置成功

```
[daniel@tiger yum.repos.d]$ sudo yum clean all && yum makecache
[daniel@tiger yum.repos.d]$ sudo yum search cloudera-manager
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
 * base: mirrors.shuosc.org
  * extras: mirrors.cn99.com
   * updates: mirrors.shuosc.org
=================================================================== N/S Matched: cloudera-manager ====================================================================
cloudera-manager-agent.x86_64 : The Cloudera Manager Agent
cloudera-manager-daemons.x86_64 : Provides daemons for monitoring Hadoop and related tools.
cloudera-manager-server.x86_64 : The Cloudera Manager Server
cloudera-manager-server-db-2.x86_64 : Embedded database for the Cloudera Manager Server

  Name and summary matches only, use "search all" for everything.
[daniel@tiger yum.repos.d]$
```
Note: 这个方法中需要在Cloudera Manager Server安装完成后将CDH的parcels文件放到 /opt/cloudera/parcels-repo/路径下。
#### 方法2：使用httpd配置Cloudera Manager和CDH本地源
例如： Cloudera Manager的文件在 /home/daniel/software/clouderea-cm5 ,CDH的parcels包文件在/home/daniel/software/cloudera-cdh

使用方法1中安装httpd的方法安装httpd

在配置文件 /etc/httpd/conf/httpd.conf的最后添加

```
<Directory "/home/daniel/software">
Options Indexes FollowSymLinks
  allowOverride None
  </Directory>
  <VirtualHost tiger:8050>
      DocumentRoot "/home/daniel/software/cloudera-cm5"
      ServerName tiger:8050
      </VirtualHost>
      <VirtualHost *:8000>
          DocumentRoot /home/daniel/software/cloudera-cdh
	      ServerName tiger:8000
	      </VirtualHost>
```
并将监听的端口从80修改位8000和8050

```
#Listen 80
Listen 8000
Listen 8050
```
将欢迎页面关闭掉。

```
[daniel@tiger ~]$ sudo vim /etc/httpd/conf.d/welcome.conf
```
注释掉里面的内容

```
#<LocationMatch "^/+$">
#    Options -Indexes
#    ErrorDocument 403 /error/noindex.html
#</LocationMatch>
```


另外需要注意对于文件所在的文件夹对other用户都有r和x的权限。

```
[daniel@tiger ~]$ ll software/
total 8
drwxrwxr-x. 2 daniel daniel 4096 Jan  6 05:19 cloudera-cdh
drwxrwxr-x. 3 daniel daniel 4096 Jan  6 14:47 cloudera-cm5
[daniel@tiger ~]$
```
配置完成后，重启httpd服务，并使用浏览器或者curl检查配置的结果。

```
curl tiger:8000
curl tiger:8050
```

###Perform OS-level configuration for Hadoop installation

**关闭防火墙**

```
sudo service iptables stop
sudo chkconfig iptables off
```

**关闭 SELinux**

```
sudo vim /etc/sysconfig/selinux
```
将其中的 SELINUX设置为disabled, SELINUXTYPE 设置为targeted
  
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

**关闭透明大页**

因为透明大页（Transparent HugePages ） 存在一些问题：

a.在RAC环境下 透明大页（Transparent HugePages ）会导致异常节点重启，和性能问题；   
b.在单机环境中，透明大页（Transparent HugePages ） 也会导致一些异常的性能问题；
        
将下面2个命令执行，并写入 /etc/rc.local 文件里

```
sudo echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag
sudo echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag
```

```
[training@lion ~]$ cat /etc/rc.local
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local
echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag
echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled
[training@lion ~]$
```

**修改使用SWAP分区的优先级**
在文件 /etc/sysctl.conf 里添加 vm.swappiness = 1

```
[training@lion ~]$ vim /etc/sysctl.conf
```

```
vm.swappiness = 1
```
linux 会使用硬盘的一部分做为SWAP分区，用来进行进程调度--进程是正在运行的程序--把当前不用的进程调成‘等待（standby）‘，甚至‘睡眠（sleep）’，一旦要用，再调成‘活动（active）’，睡眠的进程就躺到SWAP分区睡大觉，把内存空出来让给‘活动’的进程。
　　如果内存够大，应当告诉 linux 不必太多的使用 SWAP 分区， 可以通过修改 swappiness 的数值。swappiness=0的时候表示最大限度使用物理内存，然后才是 swap空间，swappiness＝100的时候表示积极的使用swap分区，并且把内存上的数据及时的搬运到swap空间里面。
													    　　

###Install Cloudera Manager server and agents

#### 安装Cloudera Manager Server
```
[daniel@tiger ~]$ sudo yum install -y cloudera-manager-daemons cloudera-manager-server
```
#### 配置使用数据库
执行如下命令：使Cloudera Manager使用配置好的MySQL数据库

```
[daniel@tiger lib]$ sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql cmserver cmserveruser password
JAVA_HOME=/usr/lib/java
Verifying that we can write to /etc/cloudera-scm-server
Creating SCM configuration file in /etc/cloudera-scm-server
Executing:  /usr/lib/java/bin/java -cp /usr/share/java/mysql-connector-java.jar:/usr/share/java/oracle-connector-java.jar:/usr/share/cmf/schema/../lib/* com.cloudera.enterprise.dbutil.DbCommandExecutor /etc/cloudera-scm-server/db.properties com.cloudera.cmf.db.
[                          main] DbCommandExecutor              INFO  Successfully connected to database.
All done, your SCM database is configured correctly!
```
等待1-2分钟，使用浏览器打开 http://localhost:7180

###Install CDH using Cloudera Manager
安装Cloudear Manager Server

```
[training@lion training]$ sudo yum install -y cloudera-manager-daemons  cloudera-manager-server
```
配置Cloudear Manager所使用的数据库

```
[training@lion training]$ sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql cmserver cmserveruser password
```
启动Cloudera Manager， 并使其开机启动,并坚持其状态（TODO：解释1-6）

```
[training@lion training]$ sudo service cloudera-scm-server start
Starting cloudera-scm-server:                              [  OK  ]
[training@lion training]$ sudo chkconfig cloudera-scm-server on
[training@lion training]$ sudo chkconfig --list | grep cloudera-scm-server
cloudera-scm-server  0:off  1:off  2:on  3:on  4:on  5:on  6:off
[training@lion training]$
```

###Add a new node to an existing cluster

HOME -> Hosts -> All Hosts -> Add New Hosts to Cluster 当前只有4个节点 

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode1.png)


![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode2.png)

搜索节点 

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode3.png)

选择节点

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode4.png)

设置本地Cloudera Manager所在的http服务

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode5.png)

选择用户并填写密码

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode6.png)

添加Host到集群

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode7.png)

成功添加Host到集群

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode8.png)

分配并激活Parcels

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode9.png)

成功激活Parcels

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode10.png)

检测服务器

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode11.png)

使用模板或者使用默认设置，使用模板可以在添加节点的时候添加和模板一样的服务到新的节点上

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode12.png)

添加成功

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode13.png)

查看新节点状态

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/AddNewNode/AddNewNode14.png)

###Add a service using Cloudera Manager

**添加Hive 服务**

在集群的Home页，选择下拉框，Add Service
选择要添加的服务
![第一步](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Hive/Add%20New%20Hive%20Service%201.png)

选择服务的依赖

![第二步](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Hive/Add%20New%20Hive%20Service%202.png)

选择需要添加的角色

![第三步](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Hive/Add%20New%20Hive%20Service%203.png)

选择要使用的数据库

![第四步](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Hive/Add%20New%20Hive%20Service%204.png)

查看也可修改相关配置

![第五步](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Hive/Add%20New%20Hive%20Service%205.png)

初始化和启动服务

![第六步](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Hive/Add%20New%20Hive%20Service%206.png)

服务启动完成

![第七步](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Hive/Add%20New%20Hive%20Service%207.png)

添加成功

![第八步](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Hive/Add%20New%20Hive%20Service%208.png)

