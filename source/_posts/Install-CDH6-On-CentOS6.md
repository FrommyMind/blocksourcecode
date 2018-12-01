---
title: 在CentOS6上安装CDH6
date: 2018-09-10 09:41:00
tags: 
- CDH6
- CDH
categories: CDH安装
---

## 1.   安装准备
### 1.1.    环境
操作系统: CentOS release 6.9 (Final)
JDK: 1.8.0_121
CM:6.0.0
CDH:6.0.0
MySQL:5.5.59
### 1.2.    下载信息
CM6.0下载地址: https://archive.cloudera.com/cm6/6.0.0/redhat6/yum/RPMS/x86_64/
allkeys.asc文件下载地址: https://archive.cloudera.com/cm6/6.0.0/
CDH6.0下载地址: https://archive.cloudera.com/cdh6/6.0.0/parcels/
MySQL下载地址: https://dev.mysql.com/downloads/mysql/5.5.html#downloads
Python27下载地址: https://www.python.org/ftp/python/2.7.13/
Psycopg下载地址: http://initd.org/psycopg/tarballs/PSYCOPG-2-5/

<!-- more -->
### 1.3.    修改主机名
执行如下命令对其内容进行修改
vim /etc/sysconfig/network
做类型如下的修改

```
NETWORKING=yes
HOSTNAME=node1-c6.example.com
```
并在命令行执行

```
hostname node1-c6.example.com
```
修改/etc/hosts文件,内容如下:

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.11.234 node1-c6.example.com node1-c6
192.168.11.235 node2-c6.example.com node2-c6
192.168.11.236 node3-c6.example.com node3-c6
192.168.11.237 node4-c6.example.com node4-c6
192.168.11.238 node5-c6.example.com node5-c6
```

### 1.4.    关闭防火墙
执行命令:
service iptables stop
并禁止开机启动
chkconfig iptables off
### 1.5.    安装JDK
进入jdk的rpm包所在的目录执行命令:

```
rpm -ivh jdk-8u121-linux-x64.rpm
```
修改/etc/profile文件,在最后添加如下内容:

```
export JAVA_HOME=/usr/java/latest
export PATH=$JAVA_HOME/bin:$PATH
```
### 1.6.    关闭交换内存
在文件/etc/sysctl.conf文件中添加如下内容:
vm.swappiness=0
并执行命令使其生效
sysctl -p

### 1.7.    关闭Selinux
修改文件/etc/selinux/config,将其中的SELINUX=enforcing修改为disabled
注:此操作需要重启生效,可以使用命令setenforce 1 使其临时生效   
### 1.8.    关闭透明大页
在命令行执行如下命令:
echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo never > /sys/kernel/mm/transparent_hugepage/defrag
此命令只是临时生效,如想永久生效,可将其添加到文件/etc/rc.local中去.
### 1.9.    创建本地yum源仓库
* 安装httpd服务  
执行命令 yum install -y httpd 来安装此服务
* 修改配置文件  
在配置文件/etc/httpd/conf/httpd.conf的779行附件将如下内容修改为  

```
AddType application/x-compress .Z  
AddType application/x-gzip .gz .tgz   
```
修改后:

```
AddType application/x-compress .Z
AddType application/x-gzip .gz .tgz .parcel
```
* 启动httpd 服务
service httpd start
* 将下载的cm和cdh文件分别放置在/var/www/html/repos/cm6/和/var/www/html/repos/cdh6/下,格式类型如下:

repos
├── cdh6
│   ├── 6.0.0
│   │   └── parcels
│   │       ├── CDH-6.0.0-1.cdh6.0.0.p0.537114-el6.parcel
│   │       ├── CDH-6.0.0-1.cdh6.0.0.p0.537114-el6.parcel.sha256
│   │       ├── index.html
│   │       └── manifest.json
└── cm6
    ├── 6.0.0
    │   ├── cloudera-manager-agent-6.0.0-530873.el6.x86_64.rpm
    │   ├── cloudera-manager-daemons-6.0.0-530873.el6.x86_64.rpm
    │   ├── cloudera-manager-server-6.0.0-530873.el6.x86_64.rpm
    │   ├── cloudera-manager-server-db-2-6.0.0-530873.el6.x86_64.rpm
    │   ├── index.html
    │   └── oracle-j2sdk1.8-1.8.0+update141-1.x86_64.rpm
    ├── allkeys.asc
        └── repomd.xml

* 安装createrepo软件
yum install -y createrepo
* 在/var/www/html/repos下执行命令
createrepo cm6
createrepo cdh6
* 在浏览器查看
cm6: http://192.168.11.234/repos/cm6/6.0.0/  
cdh6: http://192.168.11.234/repos/cdh6/6.0.0/parcels/
  
### 1.10.   MySQL 安装
* 卸载操作系统上已经存在的mysql-libs文件
先用命令查询rpm -qa | grep mysql-libs,查出rpm文件,再使用如下命令进行卸载

```
rpm -e mysql-libs-5.1.73-8.el6_8.x86_64
```
* 安装MySQL的client,server等rpm包  

```
rpm -ivh MySQL-shared-5.5.59-1.el6.x86_64.rpm  
rpm -ivh MySQL-shared-compat-5.5.59-1.el6.x86_64.rpm  
rpm -ivh MySQL-client-5.5.59-1.el6.x86_64.rpm  
rpm -ivh MySQL-server-5.5.59-1.el6.x86_64.rpm 
```
* 配置
执行初始化脚本/usr/bin/mysql_secure_installation,添加root用户的密码和移除root用户的远程登录权限
* 启动mysqld并设置为开机启动

```
service mysqld start
chkconfig mysqld on
```
* 创建用户和数据库

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
mysql -u root --password='123456' -e "FLUSH PRIVILEGES;"
```
### 1.11.   升级Python
因为CDH6需要python2.7,而centos6默认安装的是python2.6
如果服务器可以联网,则直接执行如下命令:

```
yum install centos-release-scl -y 
yum install scl-utils -y 
yum install python27 -y
```
并执行如下语句,且将其加入环境变量中去

```
source /opt/rh/python27/enable
```
如果不能联网,则需要手动进行升级
下载Python文件并解压

```
tar -zxvf Python-2.7.13.tgz
```
将解压的文件夹移动到/usr/local下,并重命名为python27

```
mv Python-2.7.13 /usr/local/python27
```
编译安装

```
./configure --prefix=/usr/local/python27
make
make install
```
将系统默认的python,从2.6的指向2.7的版本

```
mv /usr/bin/python /usr/bin/python_old
ln -s /usr/local/python27/python2.7 /usr/bin/python
```
测试是否修改正确

```
python -v
```

![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture1.png?raw=true)

### 1.12.   安装psycopg
安装依赖包

```
yum install -y gcc postgresql postgresql-server postgresql-devel
```
将之前下载的psycopg文件解压并进入解压的psycopg文件夹执行命令

```
python setup.py build
python setup.py install
```

### 1.13.   配置ntp
参考其他网上对ntp的配置
## 2.  Cloudera Manager安装
### 2.1.    配置CM的yum源
在要安装Cloudera Manager的主机上的/etc/yum.repos.d/下,创建文件cloudera-manager.local.repo,并在其中添加如下内容:

```
[cm-local]
name = cm-local
baseurl = http://192.168.11.234/repos/cm6/
gpgcheck = 0
```
其中的IP需要做对应的修改
### 2.2.    安装
执行命令:

```
yum install -y cloudera-manager-server
```
### 2.3.    配置数据库
在命令行执行:

```
/opt/cloudera/cm/schema/scm_database_functions.sh mysql cm cm 123456
```
其中,mysql是数据库类型,第一个cm是数据库名称,第二个cm是用户名,123456是密码.
### 2.4.    启动cloudera manager server
执行命令

```
service cloudera-scm-server start
```
并设置为开机启动

```
chkconfig cloudera-scm-server on
```
### 2.5.    检查
执行命令:

```
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```
监控log,当log出现如下内容时,表示启动成功,可以进入下一步操作.
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture2.png?raw=true)

## 3.  CDH安装
登录http://cloudera manager server:7180,用户名和密码都是admin
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture3.png?raw=true)
同意License协议
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture4.png?raw=true)
选择安装的版本,在CDH6中同CDH5一样,安装的时候可以进行3个版本的选择,Cloudera Express-免费版;Cloudera Enterprise Trial-企业60天试用版; Cloudera Enterprise-企业版.此处选择60天试用版本.
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture5.png?raw=true)
进入添加集群的安装界面
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture6.png?raw=true)
在CDH6的版本中,建议用户启用TLS,在安装的过程中会出现如下界面,给出启用TLS的步骤,如果需要启用可以按照其步骤进行操作,如果不需要则直接跳过.
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture7.png?raw=true)
在主机搜索界面,选择要添加的主机,可以是ip也可以是域名.
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture8.png?raw=true)
在下一页面中,在CDH and other software选项中,点击more options,在Remote Parcel Repository URLs的配置项中,将默认的配置都删除,添加前面配置的本地源,如下图所示:
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture9.png?raw=true)
在Cloudera Manager Agent的 Repository Location选项中选择Custom Repository,并将之前配置的本地cm的源地址填入其中
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture10.png?raw=true)
如果自己手动安装过JDK,此处可以不用勾选.
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture11.png?raw=true)
填入用户名和密码或者使用公钥文件
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture12.png?raw=true)

进入安装步骤
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture13.png?raw=true)
如果出现类型如下错误,说明在cm的yum源下少了allkeys.asc文件
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture14.png?raw=true)
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture15.png?raw=true)
进入分发CDH和激活界面
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture16.png?raw=true)
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture17.png?raw=true)
在主机检查界面,尽量将所有的红色或者黄色的问题都修复掉,修复后再重新运行主机检查,所有的检查都通过后,再进入下一步
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture18.png?raw=true)

选择安装的方式,不同的安装方式默认安装的CDH组件不同.
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture19.png?raw=true)
配置所有组件需要的数据库连接信息
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture20.png?raw=true)

启动
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture21.png?raw=true)
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture22.png?raw=true)
安装完成
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture23.png?raw=true)
点击完成,会自动跳转到集群的home界面.如下图
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture24.png?raw=true)
## 4.  验证
### 4.1.    HDFS验证
执行如下命令,进行验证

```
export HADOOP_USER_NAME=hdfs
hdfs dfs -ls /
```
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture25.png?raw=true)
### 4.2.    Hive验证

```
export HADOOP_USER_NAME=hdfs
beeline -u "jdbc:hive2://node1-c6.example.com:10000/default"
show databases;
```
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture26.png?raw=true)

### 4.3.    Impala验证
```
impala-shell  -i node3-c6.example.com
show databases;
```
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture27.png?raw=true)
### 4.4.    HBase验证
```
export HADOOP_USER_NAME=hbase
hbase shell
list_namespace
```
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/clouderainstall/cdh6/centos6/Picture28.png?raw=true)

