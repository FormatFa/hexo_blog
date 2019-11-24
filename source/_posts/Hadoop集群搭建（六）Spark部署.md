---
title: Hadoop集群搭建（六）Spark部署
date: 2019-10-23 20:01:11
tags:
categories:
- 大数据
- Hadoop集群搭建
typora-root-url: Hadoop集群搭建（五）Spark部署
---

HA ，主从master

Spark版本:2.0.0



解压到指定目录

```
tar -zxvf spark-2.0.0-bin-hadoop2.6.tgz  -C /opt/
mv /opt/spark-2.0.0-bin-hadoop2.6/ /opt/spark
添加到环境变量

```

修改配置文件

- slaves

```
slave1
slave2
```

<!-- more -->

- spark-env.sh

  ```bash
  
  export JAVA_HOME=/opt/jdk8
  #在yarn模式运行时读取
  export HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop
  #指定master到其他主机，这里指定时本地,因为可能有俩个机器master
  export  SPARK_MASTER_HOST=master
  
  #修改master的端口，这里还是7077,默认也是7077
  export SPARK_MASTER_PORT=7077
  #这台机器上使用的核心数
  export SPARK_WORKER_CORES=1
  #
  export SPARK_WORKER_MEMORY=1G
  #master高可用,使用zookeeper
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=master:2181,slave1:2181,slave2:2181 -Dspark.deploy.zookeeper.dir=/spark/ha"
  ```

  发送环境变量
  
  ```

  ```

  

  

  配置好的spark文件夹复制到其他节点
  
  ```
  scp -r /opt/spark/ root@slave1:/opt/
  scp -r /opt/spark/ root@slave2:/opt/
  
  
  修改配置文件就直接发送配置文件
scp -r /opt/spark/conf/* root@slave1:/opt/spark/conf
  ```
  
  

启动

master启动

`start-all.sh`

在slave1启动备用的master

```
start-master.sh 
```



效果

主master

![1571837573533](/../Hadoop集群搭建（六）Spark部署/1571837573533.png)

备用master

![1571835801466](/../Hadoop集群搭建（六）Spark部署/1571835801466.png)

spark-shell



### 错误

- 启动备用master失败

![1571834581434](/../Hadoop集群搭建（六）Spark部署/1571834581434.png)

绑定不了端口，之前SPARK_MASTER_HOST是设置为master,可能是根据这个值来绑定的，所以slave1启动master绑定不了

在spark-env.sh配置

```bash
单独配置slave1机器的spark-env.sh的这个变量，为这个机器的ip或者主机名
export  SPARK_MASTER_HOST=slave1

```

和hive连接

默认的是spark默认的，改成自己的hive

```
复制hive-site.xml到spark的配置目录
cp /opt/hive/conf/hive-site.xml  /opt/spark/conf/

 scp /opt/hive/conf/hive-site.xml  root@slave1:/opt/spark/conf/
  scp /opt/hive/conf/hive-site.xml  root@slave2:/opt/spark/conf/
复制hmysql驱动到spark的jars目录
 scp /root/mysql-connector-java-5.1.47.jar  root@slave1:/opt/spark/jars

```

测试保存数据到hive

```
启动spark集群
start-all.sh

进入spark-shell
spark-shell --master spark://master:7077
创建一个dataframe
val df = spark.createDataFrame(Seq(("mt","bin"),("adk","mxk"))).toDF("app","author")
//保存到hive表里，没有指定数据库，默认使用default数据库
 df.write.saveAsTable("fromsparktable")
 
 进入hive查看结果
hive> show tables;
OK
fromsparktable
Time taken: 0.668 seconds, Fetched: 1 row(s)
hive> select * from fromsparktable;
OK
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
mt	bin
adk	mxk

```

![1571839502090](/../Hadoop集群搭建（六）Spark部署/1571839502090.png)

参考

- spark ha https://developpaper.com/spark-ha-cluster-construction/

