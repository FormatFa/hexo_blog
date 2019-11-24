---
title: Hadoop集群搭建（三）配置Hadoop组件等
date: 2019-10-20 22:41:11
tags:
categories:
- 大数据
- Hadoop集群搭建
typora-root-url: 配置Hadoop组件等
---

### JDK配置

解压配置环境变量

```
tar -zxf jdk-8u144-linux-x64.tar.gz -C /opt/
mv /opt/jdk1.8.0_144/ /opt/jdk8
修改~/.bash_profile文件,设置PATH变量

```

.bash_profile文件

```bash
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs
export JAVA_HOME=/opt/jdk8
PATH=$JAVA_HOME/bin:$PATH:$HOME/bin

export PATH

```

<!-- more -->

### Zookeeper配置

解压zookeeper

```
 tar -zxvf zookeeper-3.4.5.tar.gz -C /opt/
 mv /opt/zookeeper-3.4.5/ /opt/zookeeper

```

zoo.cfg文件配置

```
tickTime=2000
initLimit=10
syncLimit=5
clientPort=2181

dataDir=/opt/data/zookeeper_data

server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888
scp /opt/zookeeper/conf/zoo.cfg  root@slave2:/opt/zookeeper/conf/zoo.cfg

```

复制到其他节点

每台机器在`/opt/data/zookeeper_data`设置myid 文件

```
mkdir -p /opt/data/zookeeper_data
master
echo 1 > /opt/data/zookeeper_data/myid
slave1
echo 2 > /opt/data/zookeeper_data/myid
slave2
echo 3 > /opt/data/zookeeper_data/myid

```

添加可执行目录`/opt/zookeeper/bin`到环境变量

启动zookeeper测试

需要先关闭防火墙

```
[root@master zookeeper]# systemctl stop firewalld
[root@master zookeeper]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
[root@master zookeeper]# 

```



```
zkServer.sh start启动zookeeper
zkServer.sh status 查看身份

```

![1571585208025](1571585208025.png)

![1571585217826](1571585217826.png)

![1571585229871](1571585229871.png)

### Hadoop配置

解压配置hadoop

```
 tar -zxvf hadoop-2.6.0.tar.gz -C /opt/^C
mv /opt/hadoop-2.6.0/ /opt/hadoop
添加可执行目录bin , sbin到PATH变量

复制环境变量
scp /root/.bash_profile root@slave1:/root/

```

修改配置文件

- slaves

  ```
  master
  slave1
  slave2
  ```

  

- hadoop-env.sh

  设置jdk路径即可

- core-site.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  
  <configuration>
  
      <!-- cluster1 为集群的 名字-->
  <property>
  <name>dfs.client.failover.proxy.provider.ns</name>
  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>
      
  <property>
  <name>fs.defaultFS</name>
  <value>hdfs://ns</value>
  </property>
  
  <property>
  <name>ha.zookeeper.quorum</name>
  <value>master:2181,slave1:2181,slave2:2181</value>
  </property>
  
  <property>
  <name>hadoop.tmp.dir</name>
  <value>/opt/hadoop_tmp</value>
  </property>
  
  
  </configuration>
  ```

  

- hdfs-site.xml

  ```xml
  <configuration>
  <property>
  <name>dfs.replication</name>
  <value>2</value>
  </property>
  <property>
  <name>dfs.nameservices</name>
  <value>ns</value>
  </property>
  
  <property>
  <name>dfs.ha.namenodes.ns</name>
  <value>nn1,nn2</value>
  </property>
  
  <property>
  <name>dfs.namenode.rpc-address.ns.nn1</name>
  <value>master:9000</value>
  </property>
  
  <property>
  <name>dfs.namenode.rpc-address.ns.nn2</name>
  <value>slave1:9000</value>
  </property>
  
  <property>
  <name>dfs.namenode.http-address.ns.nn1</name>
  <value>master:50070</value>
  </property>
  <property>
  <name>dfs.namenode.http-address.ns.nn2</name>
  <value>slave1:50070</value>
  </property>
  <property>
  <name>dfs.namenode.shared.edits.dir</name>
  <value>qjournal://master:8485;slave1:8485;slave2:8485/ns</value>
  </property>
  <property>
  <name>dfs.ha.fencing.methods</name>
  <value>sshfence
  shell(/bin/true)</value>
  </property>
  
  <property>
  <name>dfs.ha.automatic-failover.enabled</name>
  <value>true</value>
  </property>
  
  
  </configuration>
  
  ```

  

- mapred-site.xml

```xml
<configuration>
        <property>

        <name>mapreduce.framework.name</name>

        <value>yarn</value>

</property>

        <property>

        <name>mapreduce.jobhistory.address</name>

        <value>master:10020</value>

        </property>

   <property>

        <name>mapreduce.jobhistory.webapp.address</name>

        <value>master:19888</value>

        <description>job web地址</description>

        </property>
    
</configuration>
```



- yarn-site.xml

  ```xml
  <configuration>
  
  <property>
  
  <name>yarn.log-aggregation-enable</name>
  
  <value>true</value>
  
  </property>
  同一个ZooKeeper集群的不同Hadoop集群</description>
  同一个ZooKeeper集群的不同Hadoop集群</description>
  
  <property>
  
  <name>yarn.nodemanager.aux-services</name>
  
  <value>mapreduce_shuffle</value>
  
  <description>运行在nodemanager上的附属服务</description>
  
  </property>
  
  
  <property>
  
  <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
  
  <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  
  </property>
  
  
  <property>
  
  <name>yarn.resourcemanager.hostname</name>
  
  <value>master</value>
  
  </property>
  
  
  <property>
  
    <name>yarn.resourcemanager.recovery.enabled</name>
  
    <value>true</value>
  
  </property>
  
  <property>
  
  <name>yarn.resourcemanager.zk-address</name>
  
  <value>master:2181,slave1:2181,slave2:2181</value>
  </property>
  <property>
  
    <name>yarn.resourcemanager.ha.enabled</name>
  
    <value>true</value>
  
    <description>是否启用HA，默认false</description>
  
  </property>
  
  <property>
  
    <name>yarn.resourcemanager.ha.rm-ids</name>
  
    <value>rm1,rm2</value>
  
    <description>最少2个</description>
  
  </property>
  
  <property>
  
    <name>yarn.resourcemanager.hostname.rm1</name>
  
    <value>master</value>
  
  </property>
  
  <property>
  
    <name>yarn.resourcemanager.hostname.rm2</name>
  
    <value>slave1</value>
  
  </property>
  <property>
  
  <name>yarn.resourcemanager.cluster-id</name>
  
   <value>yarn-ha</value>
  
  <description>集群HA的id，用于在ZooKeeper上创建节点，区分使用
  
  同一个ZooKeeper集群的不同Hadoop集群</description>
  
   </property>
  
  
  </configuration>
  
  ```

  

初始化hadoop

```
scp -r /opt/hadoop/etc/hadoop/* root@slave1:/opt/hadoop/etc/hadoop/
第一次启动需要按照下面的顺序初始化namenode
1. 三台启动zookeeper
zkServer.sh start
2. 三台启动journal node
hadoop-daemon.sh start journalnode
3. 格式化zookeeper
hdfs zkfc -formatZK
4 master 格式化namenode
hdfs namenode -format
启动master上的namenode
hadoop-daemon.sh start namenode
5 slave1(备用namenode)复制master的namenode

hdfs namenode -bootstrapStandby

```

启动hadoop

```
1. 启动zookeeper
2. 启动hadoop
start-dfs.sh
启动yarn
start-yarn.sh
3. 启动备用resourcemanager
slave1 
yarn-daemon.sh start resourcemanager

4. 启动jobhistory进程
 mr-jobhistory-daemon.sh start historyserver

```

## 测试

### 查看进程

master

![1571620265355](1571620265355.png)



slave1

![1571620283076](1571620283076.png)

slave2

![1571620295864](1571620295864.png)

### 查看web界面

- namenode主备

  master:192.168.3.10:50070

  ![1571620455338](1571620455338.png)

  slave1:192.168.3.11:50070

  ![1571620473764](1571620473764.png)

- resourcemanager主备

master:

http://192.168.3.10:8088/cluster

![1571620537734](1571620537734.png)

slave1上的备用

http://192.168.3.11:8088/cluster

访问会自动重定向到master上的

### 集群测试

1. wordcount测试

```
上传测试数据到hdfs
 hdfs dfs -mkdir word_in
  hdfs dfs -put /opt/hadoop/etc/hadoop/* /word_in
运行hadoop自带的wordcount
 hadoop jar  /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar wordcount /word_in /word_out


```

![1571622851366](1571622851366.png)

jourhistory

![1571622893205](1571622893205.png)

![1571622950115](1571622950115.png)

输出结果

![1571622998104](1571622998104.png)