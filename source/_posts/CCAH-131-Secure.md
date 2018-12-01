---
title: CCAH-131 Secure
date: 2018-05-13 08:36:26
tags: CCAH-131
---
# Secure

Enable relevant services and configure the cluster to meet goals defined by security policy; demonstrate knowledge of basic security practices

## Configure HDFS ACLs
[HDFS Extended ACLs](https://www.cloudera.com/documentation/enterprise/5-12-x/topics/cdh_sg_hdfs_ext_acls.html)

HOME -> Cluster -> HDFS -Configuration 搜索ACL，勾选启用 Access Control Lists

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HDFSAcl1.png)

重启依赖的服务 

<!-- more -->
![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/HDFSAcl2.png)

## Install and configure Sentry
HOME -> Cluster -> Add Services 

选择Sentry

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Sentry/Add%20Sentry%20Service%201.png)

选择服务安装的服务器

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Sentry/Add%20Sentry%20Service%202.png)

选择数据库

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Sentry/Add%20Sentry%20Service%203.png)

安装配置并重启

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Sentry/Add%20Sentry%20Service%204.png)

重启完成

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Sentry/Add%20Sentry%20Service%205.png)

安装成功

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Sentry/Add%20Sentry%20Service%206.png)

Hive 启用 Sentry ，在Configuration配置选择中，勾选Sentry Services

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Sentry/Add%20Sentry%20Service%207.png)

Impala 启用 Sentry，在Configuration配置选择中，勾选Sentry Services

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Add%20New%20Service/Add%20Sentry/Add%20Sentry%20Service%208.png)
## Configure Hue user authorization and authentication


## Enable/configure log and query redaction
[Log and Query Redaction](https://www.cloudera.com/documentation/enterprise/5-12-x/topics/sg_redaction.html#concept_i2b_zt2_5y)

[Using Cloudera Navigator Data Management for Data Redaction](https://www.cloudera.com/documentation/enterprise/5-12-x/topics/sg_redaction.html#concept_aym_vw3_fr)

HOME -> Cluster -> HDFS -> Configuration 搜索redaction

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Log%20and%20Query%20Redaction%201.png)

选择要添加的规则并测试

![](https://github.com/FrommyMind/MarkDownPhotos/raw/master/screenshot/Log%20and%20Query%20Redaction%202.png)

## Create encrypted zones in HDFS

Create an encryption key for your zone as the application user that will be using the key. For example, if you are creating an encryption zone for HBase, create the key as the hbase user as follows:

```
$ sudo -u hbase hadoop key create <key_name>
```
Create a new empty directory and make it an encryption zone using the key created above.

```
$ sudo -u hdfs hadoop fs -mkdir /encryption_zone
$ sudo -u hdfs hdfs crypto -createZone -keyName <key_name> -path /encryption_zone
```

You can verify creation of the new encryption zone by running the -listZones command. You should see the encryption zone along with its key listed as follows:

```
$ sudo -u hdfs hdfs crypto -listZones
/encryption_zone    <key_name>
```
Warning: Do not delete an encryption key as long as it is still in use for an encryption zone. This results in loss of access to data in that zone.



###Validate Data Encryption
Login or su to these users on one of the hosts in your cluster. These directions will help to verify KMS is setup to encrypt files.

Create a key and directory.

```
su <KEY_ADMIN_USER>
hadoop key create mykey1
hadoop fs -mkdir /tmp/zone1
```
```
[training@elephant ~]$ hadoop key create mykey1
mykey1 has been successfully created with options Options{cipher='AES/CTR/NoPadding', bitLength=128, description='null', attributes=null}.
KMSClientProvider[http://horse:16000/kms/v1/] has been updated.
[training@elephant ~]$ hadoop fs -mkdir /tmp/zone1
```


Create a zone and link to the key.

```
su hdfs
hdfs crypto -createZone -keyName mykey1 -path /tmp/zone1
```

```
[training@elephant ~]$ sudo su - hdfs
-bash-4.1$ hdfs crypto -createZone -keyName mykey1 -path /tmp/zone1
Added encryption zone /tmp/zone1
-bash-4.1$
```

Create a file, put it in your zone and ensure the file can be decrypted.

```
su <KEY_ADMIN_USER>
echo "Hello World" > /tmp/helloWorld.txt
hadoop fs -put /tmp/helloWorld.txt /tmp/zone1
hadoop fs -cat /tmp/zone1/helloWorld.txt
rm /tmp/helloWorld.txt
```

```
[training@elephant ~]$ echo "Hello World" > /tmp/helloWorld.txt
[training@elephant ~]$ hadoop fs -put /tmp/helloWorld.txt /tmp/zone1
[training@elephant ~]$ hadoop fs -cat /tmp/zone1/helloWorld.txt
Hello World
[training@elephant ~]$ rm /tmp/helloWorld.txt
[training@elephant ~]$
```

Ensure the file is stored as encrypted.

```
su hdfs
hadoop fs -cat /.reserved/raw/tmp/zone1/helloWorld.txt
hadoop fs -rm -R /tmp/zone1
```
```
[training@elephant ~]$ hadoop fs -cat /.reserved/raw/tmp/zone1/helloWorld.txt
cat: Access denied for user training. Superuser privilege is required
[training@elephant ~]$
[training@elephant ~]$ sudo -u hdfs hadoop fs -cat /.reserved/raw/tmp/zone1/helloWorld.txt
�@*�@��"��[training@elephant ~]$ hadoop fs -rm -R /tmp/zone1
18/01/10 20:13:22 INFO fs.TrashPolicyDefault: Moved: 'hdfs://mycluster/tmp/zone1' to trash at: hdfs://mycluster/user/training/.Trash/Current/tmp/zone1
[training@elephant ~]$

```

By default, non-admin users cannot access any encrypted data. You must create appropriate ACLs before users can access encrypted data. See the Cloudera documentation for more information on managing KMS ACLs.

Modify the Advanced Configuration Snippet for ACL file: kms-acls.xml


[Integrating Key HSM with Key Trustee Server](https://www.cloudera.com/documentation/enterprise/5-12-x/topics/key_hsm_key_trustee.html#concept_key_hsm_key_trustee)

[Installing Cloudera Navigator Key HSM](https://www.cloudera.com/documentation/enterprise/5-12-x/topics/key_hsm_install.html#xd_583c10bfdbd326ba-590cb1d1-149e9ca9886--7a1c)

[Configuring CDH Services for HDFS Encryption](https://www.cloudera.com/documentation/enterprise/5-12-x/topics/cdh_sg_component_kms.html#xd_583c10bfdbd326ba-7dae4aa6-147c30d0933--7ba5)
