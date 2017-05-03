---
layout:     post
title:      "Java JCE and Hadoop and Kerberos"
subtitle:   "security is evil"
date:       2017-05-03 00:00:00
author:     "Vincent"
header-img: "img/post-bg-2015.jpg"
header-mask: 0.3
catalog:    true
tags:
    - hadoop
    - security
---



> Kerberos and JCE



# Java JCE and Hadoop and Kerberos

## issue:
执行hadoop命令的时候总是报错如下

```
yarn logs -applicationId application_1493267710027_0414
```

```
17/05/02 03:35:36 WARN ipc.Client: Exception encountered while connecting to the server :
javax.security.sasl.SaslException: GSS initiate failed [Caused by GSSException: No valid credentials provided (Mechanism level: Failed to find any Kerberos tgt)]
	at com.sun.security.sasl.gsskerb.GssKrb5Client.evaluateChallenge(GssKrb5Client.java:212)
	at org.apache.hadoop.security.SaslRpcClient.saslConnect(SaslRpcClient.java:413)
	at org.apache.hadoop.ipc.Client$Connection.setupSaslConnection(Client.java:563)
	at org.apache.hadoop.ipc.Client$Connection.access$1900(Client.java:378)
	at org.apache.hadoop.ipc.Client$Connection$2.run(Client.java:732)
	at org.apache.hadoop.ipc.Client$Connection$2.run(Client.java:728)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1709)
	at org.apache.hadoop.ipc.Client$Connection.setupIOstreams(Client.java:727)
	at org.apache.hadoop.ipc.Client$Connection.access$2900(Client.java:378)
	at org.apache.hadoop.ipc.Client.getConnection(Client.java:1492)
	at org.apache.hadoop.ipc.Client.call(Client.java:1402)
	at org.apache.hadoop.ipc.Client.call(Client.java:1363)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:229)
	at com.sun.proxy.$Proxy27.getFileInfo(Unknown Source)
	at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.getFileInfo(ClientNamenodeProtocolTranslatorPB.java:773)
```

但是kinit明明没有问题。

这个时候需要看一下kerberos的debug log

```
vi /usr/hdp/current/hadoop-yarn-client/bin/yarn
export YARN_OPTS="-Dsun.security.krb5.debug=true ${YARN_OPTS} -Dhdp.version=${HDP_VERSION}"
```

这个时候可以看到一些有用的信息了

```
[03:35]:[yarn@en0003:~]$ yarn logs -applicationId application_1493267710027_0414 -appOwner xxx
/usr/java/latest/bin/java
Java config name: null
Native config name: /etc/krb5.conf
Loaded from native config
>>>KinitOptions cache name is /tmp/krb5cc_204
>>>DEBUG <CCacheInputStream>  client principal is rm/n0003.xyz.com@XYZ.COM
>>>DEBUG <CCacheInputStream> server principal is krbtgt/XYZ.COM@XYZ.COM.COM
>>>DEBUG <CCacheInputStream> key type: 18
>>>DEBUG <CCacheInputStream> auth time: Tue May 02 03:09:56 GMT-07:00 2017
>>>DEBUG <CCacheInputStream> start time: Tue May 02 03:09:56 GMT-07:00 2017
>>>DEBUG <CCacheInputStream> end time: Tue May 02 13:09:56 GMT-07:00 2017
>>>DEBUG <CCacheInputStream> renew_till time: Tue May 09 03:09:56 GMT-07:00 2017
>>> CCacheInputStream: readFlags()  FORWARDABLE; RENEWABLE; INITIAL; PRE_AUTH;
>>>DEBUG <CCacheInputStream>  client principal is rm/en0003.xyz.com@XYZ.COM
>>>DEBUG <CCacheInputStream> server principal is X-CACHECONF:/krb5_ccache_conf_data/pa_type/krbtgt/XYZ.COM@ XYZ.COM
>>>DEBUG <CCacheInputStream> key type: 0
>>>DEBUG <CCacheInputStream> auth time: Wed Dec 31 17:00:00 GMT-07:00 1969
>>>DEBUG <CCacheInputStream> start time: null
>>>DEBUG <CCacheInputStream> end time: Wed Dec 31 17:00:00 GMT-07:00 1969
>>>DEBUG <CCacheInputStream> renew_till time: null
>>> CCacheInputStream: readFlags()
>>> unsupported key type found the default TGT: 18
17/05/02 03:35:34 INFO impl.TimelineClientImpl: Timeline service address: https://en0004.xyz.com:8190/ws/v1/timeline/
17/05/02 03:35:36 WARN ipc.Client: Exception encountered while connecting to the server :
javax.security.sasl.SaslException: GSS initiate failed [Caused by GSSException: No valid credentials provided (Mechanism level: Failed to find any Kerberos tgt)]
	at com.sun.security.sasl.gsskerb.GssKrb5Client.evaluateChallenge(GssKrb5Client.java:212)
	at org.apache.hadoop.security.SaslRpcClient.saslConnect(SaslRpcClient.java:413)
	at org.apache.hadoop.ipc.Client$Connection.setupSaslConnection(Client.java:563)
	at org.apache.hadoop.ipc.Client$Connection.access$1900(Client.java:378)
	at org.apache.hadoop.ipc.Client$Connection$2.run(Client.java:732)
	at org.apache.hadoop.ipc.Client$Connection$2.run(Client.java:728)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1709)
	at org.apache.hadoop.ipc.Client$Connection.setupIOstreams(Client.java:727)
	at org.apache.hadoop.ipc.Client$Connection.access$2900(Client.java:378)
	at org.apache.hadoop.ipc.Client.getConnection(Client.java:1492)
	at org.apache.hadoop.ipc.Client.call(Client.java:1402)
	at org.apache.hadoop.ipc.Client.call(Client.java:1363)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:229)
	at com.sun.proxy.$Proxy27.getFileInfo(Unknown Source)
	at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.getFileInfo(ClientNamenodeProtocolTranslatorPB.java:773)
```


注意其中的`>>> unsupported key type found the default TGT: 18`


## Reason

[https://bugzilla.redhat.com/show_bug.cgi?id=1243337](https://bugzilla.redhat.com/show_bug.cgi?id=1243337)

根据解释，这里的type 18， 表示的是AES256的一种加密算法。而当前的JDK并不支持这种方式。所以导致无法解密。


## Solution

### 方案1

修改kerberos client上的默认的加密方式

```
[libdefaults]
  renew_lifetime = 7d
  forwardable = true
  default_realm = XYZ.COM
  ticket_lifetime = 24h
  dns_lookup_realm = false
  dns_lookup_kdc = false
  #default_tgs_enctypes = aes des3-cbc-sha1 rc4 des-cbc-md5
  #default_tkt_enctypes = aes des3-cbc-sha1 rc4 des-cbc-md5
 default_tgs_enctypes = aes256-cts aes128-cts arcfour-hmac-md5 des-cbc-md5 des-cbc-crc
 default_tkt_enctypes = arcfour-hmac-md5 aes256-cts aes128-cts des-cbc-md5 des-cbc-crc
 permitted_enctypes = aes256-cts aes128-cts arcfour-hmac-md5 des-cbc-md5 des-cbc-crc
```
 
 
修改之前的加密类型
 
```
 [05:12]:[yarn@node2.xyz.com:~]$ klist -e
Ticket cache: FILE:/tmp/krb5cc_204
Default principal: nm/node2.xyz.com@XYZ.COM

Valid starting     Expires            Service principal
05/02/17 05:12:17  05/02/17 15:12:17  krbtgt/XYX.COM@XYZ.COM
	renew until 05/09/17 05:12:17, Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96
```


修改之后的加密类型

```
[05:27]:[yarn@node2.xyz.com:~]$ klist -e
Ticket cache: FILE:/tmp/krb5cc_204
Default principal: rm/node2.xyz.com@XYZ.COM

Valid starting       Expires              Service principal
05/02/2017 05:27:20  05/02/2017 15:27:20  krbtgt/XYZ.COM@XYZ.COM
	renew until 05/09/2017 05:27:20, Etype (skey, tkt): arcfour-hmac, aes256-cts-hmac-sha1-96

```


### 方案2 

下载JCE也可以,安装到jre/lib/security/也可以

[http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html](http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html)
 
 
 
### 方案3 

使用OpenJDK， 因为OpenJDK 本身就是自带Unlimited Encrytion功能的
