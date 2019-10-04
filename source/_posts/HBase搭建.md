---
title: HBase搭建
date: 2019-10-04 11:19:14
tags:
categories:
- 大数据
- Docker搭建Hadoop环境集群
typora-root-url: HBase搭建
---

## 环境

- Hadoop HA

HBase 版本 : 1.2.2

## 安装

1. 解压
```
tar -zxvf hbase-1.2.12-bin.tar.gz  -C /opt/
mv /opt/hbase-1.2.12/ /opt/hbase
```


2.  设置环境变量

3. 修改配置文件
- hbase-env.sh
  设置JAVA_HOME
  export HBASE_MANAGES_ZK=false
  export JAVA_HOME=/opt/jdk_1.8

  <!-- more -->

- hbase-site.xml
```xml
<configuration>
<property>
   <name>hbase.rootdir</name>
   <value>hdfs://cluster1/hbase</value>
</property>

<property>
   <name>hbase.cluster.distributed</name>
   <value>true</value>
</property>

<property>
   <name>hbase.zookeeper.quorum</name>
   <value>master,slave1,slave2</value>
</property>

</configuration>

```
-  regionservers
```
slave1
slave2
```
- backup-masters
输入备用的master主机名

4. 配置hdfs
复制hadoop的配置文件 到 hbase安装目录的conf目录下
```
 cp /opt/hadoop/etc/hadoop/core-site.xml  /opt/hbase/conf/     
 cp /opt/hadoop/etc/hadoop/hdfs-site.xml  /opt/hbase/conf/
5. 拷贝到其他节点
```
拷贝到hbase目录到其他两个节点:
``
scp -r /opt/hbase/ root@slave1:/opt/          
scp -r /opt/hbase/ root@slave2:/opt/       


6. 启动hbase 
 `start-hbase.sh`






## 测试

### 1. web界面截图

![1570160408051](1570160408051.png)

### 2. hbase shell

```
hbase-shell 进入shell

```

