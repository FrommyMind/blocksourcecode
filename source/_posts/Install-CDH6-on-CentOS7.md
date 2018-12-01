---
title: Install CDH6 On CentOS7
date: 2018-09-13 17:58:15
tags:
- CDH6
- CDH
---
## 1.   安装准备
### 1.1.    环境
操作系统: CentOS Linux release 7.3.1611 (Core)
JDK: 1.8.0_181
CM:6.0.0
CDH:6.0.0
mariadb:5.5
### 1.2.    下载信息
CM6.0下载地址: https://archive.cloudera.com/cm6/6.0.0/redhat7/yum/RPMS/x86_64/
allkeys.asc文件下载地址: https://archive.cloudera.com/cm6/6.0.0/
CDH6.0下载地址: https://archive.cloudera.com/cdh6/6.0.0/parcels/
mariadb下载地址: https://downloads.mariadb.org
Psycopg下载地址: http://initd.org/psycopg/tarballs/PSYCOPG-2-5/

<!-- more -->

### 1.3. 关闭IPv6
在文件/etc/sysctl.conf文件中如下内容

```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
```
并在命令行执行

```
sysctl -p
```
### 1.4. 关闭交换内存
在文件/etc/sysctl.conf文件中如下内容

```
vm.swappiness=0
```
并在命令行执行

```
sysctl -p
```
### 1.5. 修改主机名
在CentOS7上可以直接执行如下命令修改主机名称
```
[root@localhost ~]# hostnamectl set-hostname node1.example.com
[root@localhost ~]# hostname
node1.example.com
[root@localhost ~]#
```
修改/etc/hosts 文件

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.11.224 node1-c7.example.com node1
192.168.11.225 node2-c7.example.com node2
192.168.11.226 node3-c7.example.com node3
192.168.11.227 node4-c7.example.com node4
192.168.11.228 node5-c7.example.com node5
```
### 1.6. 关闭防火墙
在CentOS7上使用如下命令关闭防火墙并禁止开机启动。

```
[root@localhost java]# systemctl stop firewalld
[root@localhost java]# systemctl disable firewalld
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
Removed symlink /etc/systemd/system/basic.target.wants/firewalld.service.
[root@localhost java]#
```

### 1.7. 关闭SELINUX
修改文件/etc/selinux/config 将其中的SELINUX=enforcing修改为disabled
```
SELINUX=disabled
```

### 1.8. 关闭透明大页
在命令行执行

```
[root@localhost ~]# echo never > /sys/kernel/mm/transparent_hugepage/enabled
[root@localhost ~]# echo never > /sys/kernel/mm/transparent_hugepage/defrag
[root@localhost ~]#
```
为使其永久生效，将其添加到文件/ect/rc.local

```
touch /var/lock/subsys/local
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
```

### 1.9. ntp服务
参考其他网上对ntp的配置

### 1.10. 创建本地repository
参考：https://www.cloudera.com/documentation/enterprise/upgrade/topics/cm_ig_create_local_package_repo.html

#### 1.10.1. 安装Httpd和createrepo
执行如下命令安装http和createrepo

```
[root@localhost ~]# yum install -y httpd createrepo
```
修改配置文件/etc/httpd/conf/httpd.conf，修改284行附件将

```
AddType application/x-compress .Z
AddType application/x-gzip .gz .tgz
```
修改为

```
AddType application/x-compress .Z
AddType application/x-gzip .gz .tgz .parcel
```
启动Http

```
systemctl start httpd
```
在/var/www/html/下创建文件夹repos，在repos下创建文件夹cm6和cdh6

```
mkdir -p /var/www/html/repos/cm6
mkdir -p /var/www/html/repos/cdh6
```
将下载的文件分别上传到对应的文件夹

```
/var/www/html/repos/
|-- cdh6
|   |-- CDH-6.0.0-1.cdh6.0.0.p0.537114-el7.parcel
|   |-- CDH-6.0.0-1.cdh6.0.0.p0.537114-el7.parcel.sha256
|    `--manifest.json
`-- cm6
    |-- allkeys.asc
    |-- cloudera-manager-agent-6.0.0-530873.el7.x86_64.rpm
    |-- cloudera-manager-daemons-6.0.0-530873.el7.x86_64.rpm
    |-- cloudera-manager-server-6.0.0-530873.el7.x86_64.rpm
    |-- cloudera-manager-server-db-2-6.0.0-530873.el7.x86_64.rpm
    |-- index.html
    `--  oracle-j2sdk1.8-1.8.0+update141-1.x86_64.rpm
```
在repos文件夹下执行命令

```
createrepo cm6
createrepo cdh6
```

### 1.11. 安装数据库
使用命令行安装mariadb，因为当前版本的Linux镜像中自带的就是mariadb5.5

```
yum install -y mariadb-server
```
启动mariadb

```
systemctl start mariadb
systemctl enable mariadb
```
创建数据库和用户

```
mysql -u root --password='123456' -e 'create database metastore default character set utf8;'
mysql -u root --password='123456' -e "CREATE USER 'hive'@'%' IDENTIFIED BY '123456';"
mysql -u root --password='123456' -e "GRANT ALL PRIVILEGES ON metastore. * TO 'hive'@'%';"
mysql -u root --password='123456' -e "create user 'amon'@'%' identified by '123456'"
mysql -u root --password='123456' -e 'create database amon default character set utf8'
mysql -u root --password='123456' -e "grant all privileges on amon .* to 'amon'@'%'"
mysql -u root --password='123456' -e "create user 'rman'@'%' identified by '123456'"
mysql -u root --password='123456' -e 'create database rman default character set utf8'
mysql -u root --password='123456' -e "grant all privileges on rman .* to 'rman'@'%'"
mysql -u root --password='123456' -e "create user 'sentry'@'%' identified by '123456'"
mysql -u root --password='123456' -e 'create database sentry default character set utf8'
mysql -u root --password='123456' -e "grant all privileges on sentry.* to 'sentry'@'%'"
mysql -u root --password='123456' -e "create user 'nav'@'%'identified by '123456'"
mysql -u root --password='123456' -e 'create database nav default character set utf8'
mysql -u root --password='123456' -e "grant all privileges on nav.* to 'nav'@'%'"
mysql -u root --password='123456' -e "create user 'navms'@'%' identified by '123456'"
mysql -u root --password='123456' -e 'create database navms default character set utf8'
mysql -u root --password='123456' -e "grant all privileges on navms.* to 'navms'@'%'"
mysql -u root --password='123456' -e "create user 'cm'@'%' identified by '123456'"
mysql -u root --password='123456' -e 'create database cm default character set utf8'
mysql -u root --password='123456' -e "grant all privileges on cm.* to 'cm'@'%'"
mysql -u root --password='123456' -e "create user 'oozie'@'%'identified by '123456'"
mysql -u root --password='123456' -e 'create database oozie default character set utf8'
mysql -u root --password='123456' -e "grant all privileges on oozie.* to 'oozie'@'%'"
mysql -u root --password='123456' -e "create user 'hue'@'%' identified by '123456'"
mysql -u root --password='123456' -e 'create database hue default character set utf8'
mysql -u root --password='123456' -e "grant all privileges on hue.* to 'hue'@'%'"
mysql -u root --password='123456' -e "FLUSH PRIVILEGES;
```

### 1.12.    安装JDK
进入jdk的rpm包所在的目录执行命令:

```
rpm -ivh jdk-8u181-linux-x64.rpm
```
修改/etc/profile文件,在最后添加如下内容:

```
export JAVA_HOME=/usr/java/latest
export PATH=$JAVA_HOME/bin:$PATH
```
### 1.13.   安装psycopg
安装依赖包

```
yum install -y gcc postgresql postgresql-server postgresql-devel
```
将之前下载的psycopg文件解压并进入解压的psycopg文件夹执行命令

```
python setup.py build
python setup.py install
```
## 2. 安装Cloudera Manager
使用yum 命令安装cloudera manager

```
yum install -y cloudera-manager-server
```
安装完成后，配置cm数据库。

```
[root@node1 java]# /opt/cloudera/cm/schema/scm_prepare_database.sh  mysql cm cm 123456
JAVA_HOME=/usr/java/latest
Verifying that we can write to /etc/cloudera-scm-server
Creating SCM configuration file in /etc/cloudera-scm-server
Executing:  /usr/java/latest/bin/java -cp /usr/share/java/mysql-connector-java.jar:/usr/share/java/oracle-connector-java.jar:/usr/share/java/postgresql-connector-java.jar:/opt/cloudera/cm/schema/../lib/* com.cloudera.enterprise.dbutil.DbCommandExecutor /etc/cloudera-scm-server/db.properties com.cloudera.cmf.db.
[                          main] DbCommandExecutor              INFO  Successfully connected to database.
All done, your SCM database is configured correctly!
[root@node1 java]#
```

## 3.页面安装CDH
打开网页http://cloudera manager host:7180,用户名和密码都是admin
![初始界面](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/1.png?raw=true)
同意License
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/2.png?raw=true)
选择安装的版本,在CDH6中同CDH5一样,安装的时候可以进行3个版本的选择,Cloudera Express-免费版;Cloudera Enterprise Trial-企业60天试用版; Cloudera Enterprise-企业版.此处选择60天试用版本.
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/3.png?raw=true)
进入集群安装界面
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/4.png?raw=true)
在CDH6的版本中,建议用户启用TLS,在安装的过程中会出现如下界面,给出启用TLS的步骤,如果需要启用可以按照其步骤进行操作,如果不需要则直接跳过.
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/5.png?raw=true)
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/6.png?raw=true)
填入需要安装的主机IP地址
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/7.png?raw=true)
在Custom Repository的输入框中填入之前创建的本地CM的yum源地址
在CDH and other software的More Options的选项中填入CDH的yum源的地址，当CDH Version自动选择的CDH-6.0才继续下一步
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/8.png?raw=true)
如果安装过了JDK，此处不勾选。
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/9.png?raw=true)
填入root用户的密码
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/10.png?raw=true)
进入安装cloudera agent的步骤
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/11.png?raw=true)
进入分发parcels的步骤
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/12.png?raw=true)
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/13.png?raw=true)
主机检查，尽量修复所有的警告问题。
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/14.png?raw=true)
选择安装的方式,不同的安装方式默认安装的CDH组件不同.
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/15.png?raw=true)
配置所有组件需要的数据库连接信息
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/16.png?raw=true)
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/17.png?raw=true)
启动
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/18.png?raw=true)
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/19.png?raw=true)
安装完成
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/20.png?raw=true)
