---
title: Hadoop HA(高可用)搭建
date: 2019-10-03 10:31:21
tags:
- Docker
- hadoop
- 大数据
categories:
- 大数据
- Docker搭建Hadoop环境集群
typora-root-url: Hadoop-HA-高可用-搭建
---

# Hadoop HA(高可用)搭建

## 环境准备和集群规划

> Hadoop 是安装其他组件的基础,必须先配置好Hadoop环境

### 1. 环境

需要准备三台centos 7虚拟机，配置好下面条件

1.  设置好hostname主机名(master,slave1,slave2)
2. 网络可以互相ping通
3. 关闭防火墙

如果是用Docker 搭建,使用里面的centos:7镜像的话,还需要用yum安装下面的东西(因为默认没有安装)

1. 安装好网络工具ss 和ifconfig
2. 安装sshd服务

宿主机使用ssh工具连接到三台主机,和对应的上传文件工具(上传安装到到集群)

这里宿主机是Ubuntu 系统的,直接使用内置的终端ssh连接到集群(Docker的可以直接exec进入bash也行)

### 2. 集群规划

| 软件   | 版本     |      |
| ------ | -------- | ---- |
| hadoop | 2.7.7    |      |
| jdk    | 1.8      |      |
| 系统   | CentOS 7 |      |

一台master+两台slave

| 主机名 | 角色     | 其他 |
| ------ | -------- | ---- |
| master | namenode |      |
| slave1 | datanode |      |
| slave2 | datanode |      |



集群软件目录:

| 项目            | 安装目录   |      |
| --------------- | ---------- | ---- |
| hadoop组件等    | /opt/      |      |
| hadoop 数据目录 | /opt/data/ |      |
|                 |            |      |

### 3. 下载连接

hadoop等:apache 镜像站:https://mirrors.tuna.tsinghua.edu.cn/apache/

## 搭建过程

#### 1. 配置 ssh 免密登陆

>设置主机之间可以免密码登录

两种方式设置免密登陆,一种是手动将公钥追加到要登录的主机的~/.ssh/authorized_keys文件下，一种是使用ssh-copy-id命令(推荐)

<!-- more -->

​	a. 手动设置免密登陆

​	手动设置免密登录,这里以设置master免密登录两个slave(和自己)为例

- 先在master 生成公钥和私钥

  输入`ssh-keygen` 命令,一直回车,生成的密钥保存在/root/.ssh目录

- 添加公钥到slave1和slave2节点

master,slave1和slave2节点默认没有`~/.ssh`文件夹,在两个节点执行`ssh localhost`就会生成`~/.ssh`文件夹(或者手动建立)

- 将master的公钥(id_rsa.pub)添加到 三台主机的~/.ssh/authorized_keys 文件

  ```markdown
  在master执行复制命令
  
  //复制公钥到slave1节点的/root目录
  [root@3de59086e794 ~]# scp ~/.ssh/id_rsa.pub  root@slave1:/root/
  The authenticity of host 'slave1 (172.19.0.3)' can't be established.
  ECDSA key fingerprint is SHA256:JKinnsafdpyJ4e9NB3nvdNz5sHASU0whnVLWL9nyGlg.
  ECDSA key fingerprint is MD5:07:b5:34:4e:14:63:2c:3a:8a:52:88:4c:bf:07:58:71.
  Are you sure you want to continue connecting (yes/no)? yes
  Warning: Permanently added 'slave1,172.19.0.3' (ECDSA) to the list of known hosts.
  root@slave1's password: 
  id_rsa.pub                                                                                                  100%  399     1.8MB/s   00:00    
  //复制公钥到slave2节点的/root目录
  [root@3de59086e794 ~]# scp ~/.ssh/id_rsa.pub  root@slave2:/root/
  root@slave2's password: 
  id_rsa.pub                                                                                                  100%  399     1.2MB/s   00:00    
  [root@3de59086e794 ~]# 
  
  ```

  ```
  在master,slave1 和 slave2 节点执行(这里为读取id_ras.pub文件的内容追加到authorized_keys文件)
  
   cat id_rsa.pub >> ~/.ssh/authorized_keys
   //查看下添加后的authorized_keys文件
  [root@8569789b1bc0 ~]# cat ~/.ssh/authorized_keys 
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCkG4760Jb3jf8ARJ6knvVkXpixJ0lEhyu2GfFDEGF9SYtdtK6/rZ33noGCoK50PmZCIOrVcvYrVit5NyiyWtcQljkwowLPP6K2SRLkh7nCTEHZjfOhyW1qjyteHazKpF36iDli37zcKFzsIQz2IfoaNoyghE3y6eqXsaByO3kooPJlhjvRsb/AbuPDIMgZPeQ++gfEsBALBhfqo4xnQDvCuo8Ibi5Pw9gSs2Kik8JXmxju1R6IHmzQYnhXQy2pVfrvIgyznfWrWRxAPd9xNfTfnZ51hSc0t1Gfnpgi1ptknTfXqm0DYa+TFWzgle98SjZ58plf1Ca9vzWmfc5qJfld root@3de59086e794
  
  authorized_keys文件的权限,权限需要设置为600 
  [root@8569789b1bc0 ~]# ll ~/.ssh/authorized_keys 
  -rw-r--r-- 1 root root 399 Oct  2 03:17 /root/.ssh/authorized_keys
  [root@8569789b1bc0 ~]# 
  可以看到上面的权限是 644,需要修改为 600(测试 644也能登录,但777不行)
  修改为 600的命令为
  chmod 600 ~/.ssh/authorized_keys 
  
  
  
  
  ```

  

  b. 使用ssh-copyid命令

  用这个命令就不用复制公钥再追加到文件，直接一个命令就行，以slave1免密登录三台主机为例

  - 生成公钥和私钥

    `ssh-keygen`

  - 发送公钥到其他节点

    ```
    ssh-copy-id master
    ssh-copy-id slave1
    ssh-copy-id slave2
    
    ```

    ```
    测试免密登录
    ssh master(第一次可能要验证主机,输入yes就行)
    直接登录，不用输入密码
    ```

    ![1569986981320](1569986981320.png)

#### 2. 安装jdk和hadoop

> 用scp 或者 第三方工具将jdk和hadoop的安装包上传到主机里
>
> 先在master 解压和修改好配置 文件,然后直接发送到其他两个slave主机

- 解压

jdk解压

```
 tar -zxvf jdk-8u191-linux-x64.tar.gz  -C /opt/
 mv /opt/jdk1.8.0_191/ /opt/jdk1.8

```

hadoop解压

```
tar -zxvf hadoop-2.7.7.tar.gz -C  /opt/

mv /opt/hadoop-2.7.7/ /opt/hadoop

```

- 修改配置文件

配置文件位于hadoop安装目录下的`etc/hadoop/`文件夹下

1. hadoop-evn.sh

   这个文件只要修改JAVA_HOME变量

```
...

# The java implementation to use.
export JAVA_HOME=/opt/jdk1.8


...
```



2. core-site.xml

   配置文件里出现的cluster1 为fs.default.FS里指定的namenode的地址,如果修改了`fs.default.FS`,其他的地方也要对应修改

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>


<configuration>

    <!-- cluster1 为集群的 名字-->
<property>
<name>dfs.client.failover.proxy.provider.cluster1</name>
<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>
    
<property>
<name>fs.defaultFS</name>
<value>hdfs://cluster1</value>
</property>

<property>
<name>ha.zookeeper.quorum</name>
<value>master:2181,slave1:2181,slave2:2181</value>
</property>

    临时目录,如果没有指定namenode或者datanode的数据目录，默认会在${hadoop.tmp.dir}目录下,这个目录默认在???,重启可能会被清除，这里换成指定的hadoop.tmp.dir
<property>
<name>hadoop.tmp.dir</name>
<value>/opt/hadoop_tmp</value>
</property>


</configuration>
```



3. hdfs-site.xml

```xml
<configuration>
    

<property>
<name>dfs.replication</name>
<value>2</value>
</property>
<!--   -->
<property>
<name>dfs.nameservices</name>
<value>cluster1</value>
</property>

<property>
<name>dfs.ha.namenodes.cluster1</name>
<value>nn1,nn2</value>
</property>

<property>
<name>dfs.namenode.rpc-address.cluster1.nn1</name>
<value>master:9000</value>
</property>

<property>
<name>dfs.namenode.rpc-address.cluster1.nn2</name>
<value>slave1:9000</value>
</property>

<property>
<name>dfs.namenode.http-address.cluster1.nn1</name>
<value>master:50070</value>
</property>
<property>
<name>dfs.namenode.http-address.cluster1.nn2</name>
<value>slave1:50070</value>
</property>
<property>
<name>dfs.namenode.shared.edits.dir</name>
<value>qjournal://master:8485;slave1:8485;slave2:8485/cluster1</value>
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


    
<property>
<name>dfs.namenode.name.dir</name>
<value>/opt/data/hadoop_data/namenode</value>
</property>

<property>
<name>dfs.datanode.data.dir</name>
<value>/opt/data/hadoop_data/datanode</value>
</property>

</configuration>


```

3. yarn-site.xml

```xml
<configuration>

<property>

<name>yarn.log-aggregation-enable</name>

<value>true</value>

</property>

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



 4. **mapred-site.xml**

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



- slaves 文件

  ```
  slave1
  slave2
  ```

  

### zookeeper 安装

> 高可用需要用到zookeeper

解压

```
tar -zxvf zookeeper-3.4.5.tar.gz  -C /opt/
mv /opt/zookeeper-3.4.5/ /opt/zookeeper

```



修改zoo.cfg文件

```
 mv conf/zoo_sample.cfg  conf/zoo.cfg
添加修改conf.cfg文件

dataDir=/opt/data/zookeeper_data

server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888
```

### 配置环境变量

修改`/root/.bashrc`文件

```shell
后面添加下面的变量
export JAVA_HOME=/opt/jdk1.8
export HADOOP_HOME=/opt/hadoop
export ZOOKEEPER_HOME=/opt/zookeeper
export PATH=${JAVA_HOME}/bin:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:${ZOOKEEPER_HOME}/bin:$PATH


```





在三台主机 建立要手动建立的文件夹(namenode数据目录,datanode数据目录,zookeeper的数据目录)

```

mkdir -p /opt/data/hadoop_data/datanode
mkdir -p /opt/data/hadoop_data/namenode
zookeeper 数据目录
mkdir -p /opt/data/zookeeper_data
```



复制下面的数据到其他节点,第一次可以 直接将整个/opt/下的复制过去，因为组件和上面建的文件夹都在/opt下面

- 复制环境变量

  ```
  scp /root/.bashrc root@slave1:/root/
  scp /root/.bashrc root@slave2:/root/
  
  ```

  

- 复制组件

  ```
  scp -r  /opt/* root@slave1:/opt/
  scp -r  /opt/* root@slave2:/opt/
  ```

  

测试环境变量

```
[root@d7998eb32bfb ~]# source /root/.bashrc
[root@d7998eb32bfb ~]# java -version
java version "1.8.0_191"
Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
[root@d7998eb32bfb ~]# 



```

### 修改需要单独配置的文件

- zookeeper 的 myid文件

```
master,slave1,slave2分别执行
echo 1 > /opt/data/zookeeper_data/myid
echo 2 > /opt/data/zookeeper_data/myid
echo 3 > /opt/data/zookeeper_data/myid
```

### 初始化hadoop

1. 三个主机都启动zookeeper

   ```
   zkServer.sh start
   ```

   使用`zkServer.sh status`查看身份，如果有一个leader和两个follower就成功

   ![1569990314156](1569990314156.png)

   ![1569990323756](1569990323756.png)

![1569990305940](1569990305940.png)

2. 启动journal node 进程

```
三台机器
hadoop-daemon.sh start journalnode

```

3. 格式化zookeeper节点

```
格式化前查看(通过zkCli.sh进入)
[zk: localhost:2181(CONNECTED) 1] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 2] 

```

`hdfs zkfc -formatZK`

```
19/10/02 04:34:50 INFO zookeeper.ZooKeeper: Client environment:java.library.path=/opt/hadoop/lib/native
19/10/02 04:34:50 INFO zookeeper.ZooKeeper: Client environment:java.io.tmpdir=/tmp
19/10/02 04:34:50 INFO zookeeper.ZooKeeper: Client environment:java.compiler=<NA>
19/10/02 04:34:50 INFO zookeeper.ZooKeeper: Client environment:os.name=Linux
19/10/02 04:34:50 INFO zookeeper.ZooKeeper: Client environment:os.arch=amd64
19/10/02 04:34:50 INFO zookeeper.ZooKeeper: Client environment:os.version=5.0.0-29-generic
19/10/02 04:34:50 INFO zookeeper.ZooKeeper: Client environment:user.name=root
19/10/02 04:34:50 INFO zookeeper.ZooKeeper: Client environment:user.home=/root
19/10/02 04:34:50 INFO zookeeper.ZooKeeper: Client environment:user.dir=/opt
19/10/02 04:34:50 INFO zookeeper.ZooKeeper: Initiating client connection, connectString=master:2181,slave1:2181,slave2:2181 sessionTimeout=5000 watcher=org.apache.hadoop.ha.ActiveStandbyElector$WatcherWithClientRef@37afeb11
19/10/02 04:34:50 INFO zookeeper.ClientCnxn: Opening socket connection to server 3de59086e794/172.19.0.4:2181. Will not attempt to authenticate using SASL (unknown error)
19/10/02 04:34:50 INFO zookeeper.ClientCnxn: Socket connection established to 3de59086e794/172.19.0.4:2181, initiating session
19/10/02 04:34:50 INFO zookeeper.ClientCnxn: Session establishment complete on server 3de59086e794/172.19.0.4:2181, sessionid = 0x16d8ab6613c0001, negotiated timeout = 5000
19/10/02 04:34:50 INFO ha.ActiveStandbyElector: Session connected.
19/10/02 04:34:50 INFO ha.ActiveStandbyElector: Successfully created /hadoop-ha/cluster1 in ZK.
19/10/02 04:34:50 INFO zookeeper.ZooKeeper: Session: 0x16d8ab6613c0001 closed
19/10/02 04:34:50 INFO zookeeper.ClientCnxn: EventThread shut down
[root@3de59086e794 opt]# 

```

4.   格式化namenode节点

```
master上执行
hdfs namenode -format
格式化完后有文件了namenode的目录
[root@3de59086e794 opt]# ll data/hadoop_data/namenode/current/
total 16
-rw-r--r-- 1 root root 201 Oct  2 04:38 VERSION
-rw-r--r-- 1 root root 321 Oct  2 04:38 fsimage_0000000000000000000
-rw-r--r-- 1 root root  62 Oct  2 04:38 fsimage_0000000000000000000.md5
-rw-r--r-- 1 root root   2 Oct  2 04:38 seen_txid
[root@3de59086e794 opt]# 

```

master上的namenode格式化后，先启动master上的namenode节点(下面同步需要，不然会报错说连接不到master:9000)

`hadoop-daemon.sh  start namenode`

接下来要在第二个namenode上把master上格式化完的数据同步到第二个namenode上,在slave1(备用namenode节点)上执行

`hdfs namenode -bootstrapStandby`

---

### 启动集群

1. 启动zookeeper(三台主机)

   `zkServer.sh start`

2. 启动hdfs

   `start-dfs.sh`

3. 启动yarn

   `start-yarn.sh`

4.手动启动yarn的备用节点

`yarn-daemon.sh start resourcemanager`



5. 在master启动mr 历史进程

` mr-jobhistory-daemon.sh start historyserver`

开启jobhistory ,可以在任务结束后查看任务运行情况

jobhistory 的web端口在mapred-site.xml里设置,为19888

### 查看进程

master

![1569992007816](1569992007816.png)

slave1

![1569992018604](1569992018604.png)

slave2

![1569992027208](1569992027208.png)

查看开放端口

![1570063825179](1570063825179.png)

### 停止集群

```
stop-dfs.sh
stop-yarn.sh
```



## 集群测试

### 1. web界面查看

namenode

![1569992108322](1569992108322.png)

备用节点slave1

![1569992161841](1569992161841.png)

​	ResouceManager 主备节点

![1570064382082](1570064382082.png)

进入备用的会重定向到master的,由于宿主机没有设置master对应的ip,所以访问不了

![1570064476894](1570064476894.png)

### 2. namenode 高可用测试

> 1. 杀掉master上的namenode,查看slave1上的namenode的状态
> 2. 启动杀死的namenode,查看它的状态

集群刚启动的情况, 现在master是standby,slave1是active 

![1570063926217](1570063926217.png)

![1570063935599](1570063935599.png)

杀死active的namenode,在slave1上执行

```
通过jps命令查看到namenode的pid,使用kill -9 pid命令杀死
[root@slave1 /]# jps
51 QuorumPeerMain
372 DFSZKFailoverController
117 NameNode
262 JournalNode
536 NodeManager
185 DataNode
697 Jps
namenode的pid是117
[root@slave1 /]# kill -9 117
杀死后再看jps已经没有了
[root@slave1 /]# jps
51 QuorumPeerMain
372 DFSZKFailoverController
262 JournalNode
536 NodeManager
712 Jps
185 DataNode
[root@slave1 /]# 

```

这时查看web 界面,master由standby变成active

![1570064080387](1570064080387.png)

因为namenode进程被杀死,web服务也没了，slave1的打不开

![1570064129505](1570064129505.png)



然后重新启动被杀死的namenode

```
启动namenode进程
[root@slave1 /]# hadoop-daemon.sh start namenode
starting namenode, logging to /opt/hadoop/logs/hadoop--namenode-slave1.out
[root@slave1 /]# jps
51 QuorumPeerMain
836 Jps
372 DFSZKFailoverController
262 JournalNode
742 NameNode
536 NodeManager
185 DataNode
[root@slave1 /]# 

```

web界面恢复，由active变成现在的standby

![1570064227489](1570064227489.png)

### 3. mapreduce 运行测试

```
[root@master ~]# hadoop jar wordcount.jar WordCount
19/10/03 01:21:40 INFO input.FileInputFormat: Total input paths to process : 1
19/10/03 01:21:41 INFO mapreduce.JobSubmitter: number of splits:1
19/10/03 01:21:41 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1570063799017_0001
19/10/03 01:21:42 INFO impl.YarnClientImpl: Submitted application application_1570063799017_0001
19/10/03 01:21:42 INFO mapreduce.Job: The url to track the job: http://master:8088/proxy/application_1570063799017_0001/
19/10/03 01:21:42 INFO mapreduce.Job: Running job: job_1570063799017_0001
19/10/03 01:21:51 INFO mapreduce.Job: Job job_1570063799017_0001 running in uber mode : false
19/10/03 01:21:51 INFO mapreduce.Job:  map 0% reduce 0%
19/10/03 01:21:59 INFO mapreduce.Job:  map 100% reduce 0%
19/10/03 01:22:05 INFO mapreduce.Job:  map 100% reduce 100%
19/10/03 01:22:07 INFO mapreduce.Job: Job job_1570063799017_0001 completed successfully
19/10/03 01:22:07 INFO mapreduce.Job: Counters: 49
	File System Counters
		FILE: Number of bytes read=1731
		FILE: Number of bytes written=253541
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=1259
		HDFS: Number of bytes written=1063
		HDFS: Number of read operations=6
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
	Job Counters 
		Launched map tasks=1
		Launched reduce tasks=1
		Data-local map tasks=1
		Total time spent by all maps in occupied slots (ms)=4626
		Total time spent by all reduces in occupied slots (ms)=3956
		Total time spent by all map tasks (ms)=4626
		Total time spent by all reduce tasks (ms)=3956
		Total vcore-milliseconds taken by all map tasks=4626
		Total vcore-milliseconds taken by all reduce tasks=3956
		Total megabyte-milliseconds taken by all map tasks=4737024
		Total megabyte-milliseconds taken by all reduce tasks=4050944
	Map-Reduce Framework
		Map input records=45
		Map output records=114
		Map output bytes=1497
		Map output materialized bytes=1731
		Input split bytes=87
		Combine input records=0
		Combine output records=0
		Reduce input groups=89
		Reduce shuffle bytes=1731
		Reduce input records=114
		Reduce output records=89
		Spilled Records=228
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=116
		CPU time spent (ms)=1830
		Physical memory (bytes) snapshot=432754688
		Virtual memory (bytes) snapshot=3911442432
		Total committed heap usage (bytes)=348127232
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=1172
	File Output Format Counters 
		Bytes Written=1063
[root@master ~]# 

```



查看job history 

![1570065773875](1570065773875.png)

![1570065796139](1570065796139.png)