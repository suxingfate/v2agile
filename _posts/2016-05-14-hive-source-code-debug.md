---
layout:     post
title:      "hive源码调试 IntelliJ"
subtitle:   ""
date:       2017-05-01 01:00:00
author:     "Vincent"
header-img: "img/post-bg-2015.jpg"
header-mask: 0.3
catalog:    true
tags:
    - hive
---

### 序
***
最近在阅读hive源码。所谓工欲善其事，必先利其器。能够对源码进行调试会对阅读源码有很大帮助。那么就开始调试hive吧。

### 目录
***
- hadoop pseudo distributed mode
- Hive 安装
- Hive 源码调试



### hadoop pseudo distributed mode

在本地编译好的hadoop-dist/target目录下启动hadoop

```
/Users/suxingfate/code/hadoop-release-2.7.1/hadoop-dist/target/hadoop-2.7.1
```

```
bin/hdfs namenode -format
sbin/start-dfs.sh
sbin/start-yarn.sh 
```
错误分析：

```shell
$HADOOP_HOME/bin/hadoop fs -ls 16/05/14 08:28:46 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
ls: Call From lm-shc-16501189.lan/192.168.199.105 to localhost:9000 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
```

这里是因为没有format namenode，导致namenode根本没有启动

[http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation)


### Hive 安装

1. 这里直接下载来**hive 1.2.1**的源码包和安装包

根据官方文档的要求设置相应的环境变量并且生成对应目录即可

	export HADOOP_HOME=<hadoop-install-dir>
	$HADOOP_HOME/bin/hadoop fs -mkdir /tmp
	$HADOOP_HOME/bin/hadoop fs -mkdir /user/hive/warehouse
	$HADOOP_HOME/bin/hadoop fs -chmod g+w /tmp
	$HADOOP_HOME/bin/hadoop fs -chmod g+w /user/hive/warehouse

然后可以启动hive了

	$HIVE_HOME/bin/hive


**参考文献:**

[https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-InstallingHivefromaStableRelease](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-InstallingHivefromaStableRelease)


### hive 源码调试

**实验环境：**

* mac
* hadoop-2.7.1伪分布安装，这里使用的是本地编译的版本，也可以直接使用下载版本
* hive 1.2.1. 直接从hive官网下载的编译版本和代码源jar包
intellj

**实验过程**

- **Java Project**

在Intellj中创建了一个新的java project，然后右键项目设置Modules相关设置，添加hive conf和hadoop conf，以及hive与hadoop相关的jar包

> 这里指定到目录级别就可以了，IntelliJ可以自动识别目录下的jar

![hive dep](/img/2016/hive-dep.png)

并创建如下的代码调用HIVE

```java
import org.apache.hadoop.hive.cli.CliDriver;
public class HiveCli {
    public static void main(String[] args){
      	 try {
      	     CliDriver.main(args);
      	 } catch (Exception e) {
            e.printStackTrace();
    	 }
    }
}
```

调试成功。

![hive dep](/img/2016/hive-debug.png)




- **metastore**

打开项目所在目录，可以看到metastore_db, derby.log


```shell
hive-src# ls
derby.log    hive-src.iml metastore_db pom.xml      src          target
```

参考文章：

[在IDE上调试Hive](http://eclipse-cc.iteye.com/blog/1410012)

[hive1.2.1源码导入eclipse阅读以及调试](http://blog.csdn.net/zhoudetiankong/article/details/50484086)




