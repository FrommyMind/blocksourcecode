---
title: Configuration DbVisualize to Connect Hive with Kerberos On CDH
date: 2018-05-13 11:20:26
tags: 
  - CDH
  - Hive
  - Kerberos
---
# Configuration DbVisualize to Connect Hive with Kerberos On CDH

## What you need
1. DbVisualize  Download From *[Here](http://www.dbvis.com/download/)*
2. Kerbers Windows Tool. Download From *[Here(64)](http://web.mit.edu/kerberos/dist/kfw/4.0/kfw-4.0.1amd64.msi)*  *[Here(32)](http://web.mit.edu/kerberos/dist/kfw/4.0/kfw-4.0.1amd64.msi)*
3. Jre Download From *[Here](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)*
4. krb5.conf / krb5.ini *This should copy from your kerberos server.*
5. jaas.conf
6. Cloudera Hive JDBC Driver Download from *[Here](https://www.cloudera.com/downloads/connectors/hive/jdbc/2-5-4.html)*
7. Make sure your laptop have the **UTP** access from port 88 to the kdc server.

<!-- more -->

## Install *dbvis_windows-x64_10_0_jre.exe* 

## Install *kfw-4.0.1-amd64.msi* 

## Config jre
cp local_policy.jar US_export_policy.jar from jre folder to %DbViisualize_HOME%\jre\lib\security

![](https://i.imgur.com/b7xoKOb.png)

## Configuration Envrionment Parameters

KRB5_CONFIG=C:\Windows\krb5.ini
KRB5CCNAME=C:\Users\Username\krb5cache
![](https://i.imgur.com/vbaCQCS.png)
![](https://i.imgur.com/qMqzOhE.png)

## Kinit
Use Administrator run cmd

    cd C:\"Program Files"\MIT\Kerberos\bin
    kinit -kt hive.keytab -c *cachefilename* hive/h2_server.com@REALM
    klist -c *cachefilename*

 cachefilename is KRB5CCNAME

## jass.conf 

HiveClient {
  com.sun.security.auth.module.Krb5LoginModule required
  doNotPrompt=true
  useTicketCache=true
  principal="hive/example@HTSEC.COM"
  useKeyTab=true
  keyTab="C:/Users/Administrator/hive.keytab"
  client=true;
};


## Start DbVisualize
Use cmd to start DbVisualize 

![](https://i.imgur.com/fGFxAZv.png)

set jvm on DbVisualize
![](https://i.imgur.com/lljTOzx.png)

	-Dsun.security.jgss.debug=true
	-Dsun.security.krb5.debug=true
	-Djava.security.krb5.realm=REALM.COM
	-Djava.security.krb5.kdc=example.com
	-Djava.security.auth.login.config=C:\Users\Users\jaas.conf
	-Djava.security.krb5.conf=C:\Windows\krb5.ini

restart DbVisualize

## Load Driver
![](https://i.imgur.com/nvHuG8q.png)
![](https://i.imgur.com/Mfb3ZlN.png)
![](https://i.imgur.com/aI4q02J.png)

## Connect Hive2

    jdbc:hive2://example.com:10000/default;AuthMech=1;KrbRealm=REALM.COM;KrbHostFQDN=h2_server.com;KrbServiceName=hive;
