---
title: Hadoop集群搭建（五）Hive部署
date: 2019-10-23 18:25:01
tags:
categories:
- 大数据
- Hadoop集群搭建
---

解压hive 安装包

```
tar -zxvf apache-hive-1.1.0-bin.tar.gz -C /opt/
 mv /opt/apache-hive-1.1.0-bin/ /opt/hive
添加到环境变量
```

修改配置文件

- hive-env.sh 

```

```

- hive-site.xml

  ```
  修改hive-default.xml为hive-site.xml
  1. 设置jdbc路径
  修改
  
  
  ```

  1. 修改下面的配置项

  ```xml
    <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>hive</value>
      <description>password to use against metastore database</description>
    </property>
  
    <property>
      <name>javax.jdo.option.ConnectionURL</name>
       <value>jdbc:mysql://192.168.3.11/hivemeta?useSSL=false&amp;createDatabaseIfNotExist=true</value>
      <description>JDBC connect string for a JDBC metastore</description>
    </property>
  <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
      <description>Driver class name for a JDBC metastore</description>
    </property>
  <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>hive</value>
      <description>Username to use against metastore database</description>
    </property>
  
  
  
  ```

  2.  修改java.io.tmp
     把hive-site.xml里的
     ${system:java.io.tmpdir%}全部修改为/tmp,不然启动会报错

  上传mysql驱动到hive的lib目录中

  ```
  scp /root/mysql-connector-java-5.1.47.jar  /opt/hive/lib/
  
  ```

  启动后会有这个错误

  ```
  
  Exception in thread "main" java.lang.IncompatibleClassChangeError: Found class jline.Terminal, but interface was expected
  	at jline.console.ConsoleReader.<init>(ConsoleReader.java:230)
  	at jline.console.ConsoleReader.<init>(ConsoleReader.java:221)
  	at jline.console.ConsoleReader.<init>(ConsoleReader.java:209)
  	at org.apache.hadoop.hive.cli.CliDriver.getConsoleReader(CliDriver.java:773)
  	at org.apache.hadoop.hive.cli.CliDriver.executeDriver(CliDriver.java:715)
  	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:675)
  	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:615)
  	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
  	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  	at java.lang.reflect.Method.invoke(Method.java:498)
  	at org.apache.hadoop.util.RunJar.run(RunJar.java:221)
  	at org.apache.hadoop.util.RunJar.main(RunJar.java:136)
  
  ```

  

  类冲突,网上解决方法

  https://blog.csdn.net/qq_39507276/article/details/85367305

  ```
   mv    /opt/hadoop/share/hadoop/yarn/lib/jline-0.9.94.jar   /opt/hadoop/share/hadoop/yarn/lib/jline-0.9.94.jar.bak
   cp /opt/hive/lib/jline-2.12.jar  /opt/hadoop/share/hadoop/yarn/lib/
  hive里的jline版本比hadoop里的高
  ```

  

- 初始化元数据

  ```
  暂时不知道有什么用，没初始化直接使用hive命令也行
  schematool -dbType mysql -initSchema 
  ```

- 复制目录到其他节点

```

```

