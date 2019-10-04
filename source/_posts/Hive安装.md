---
title: Hive安装
date: 2019-10-03 18:07:42
tags:
categories:
- 大数据
- Docker搭建Hadoop环境集群
typora-root-url: Hive安装
---

hive版本 1.2 

下载地址https://mirrors.tuna.tsinghua.edu.cn/apache/hive/

环境准备:

1. 已经安装了Hadoop 

2. 一台安装有mysql的主机 (用于保存hive的元数据)

---

docker 创建mysql 5.7的 镜像bigdata_mysql，用来保存hive元数据

- 指定root密码为root

- 创建容器时指定创建一个数据库hivedata

- 创建一个用户hive,密码为hive

- 添加到网络hadoopCluster

  <!-- more -->

```
docker create --network hadoopCluster --name "bigdata_mysql"   -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=hivedata -e MYSQL_USER=hive -e MYSQL_PASSWORD=hive  mysql:5.7 

//连接测试 创建一个临时mysql客户端容器
docker  run  -it --network hadoopCluster --rm mysql mysql -hbigdata_mysql  -uroot  -p
js@ljh-X441UVK:~$ docker  run  -it --network hadoopCluster --rm mysql mysql -hbigdata_mysql  -uhive  -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.27 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
打开后发现，命令里指定的数据库成功创建了
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hivedata           |
+--------------------+
2 rows in set (0.01 sec)

mysql> 

```

---

解压

```
[root@master ~]# tar -zxf apache-hive-1.2.2-bin.tar.gz -C  /opt/
[root@master ~]# mv /opt/apache-hive-1.2.2-bin/ /opt/hive

```

修改~/.bashrc 文件 添加bin目录到PATH变量

在安装目录下的conf下新建hive-site.xml文件 

jdbc url里直接用bigdata_mysql,因为时在docker里配置的，都在统一网络里

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<configuration>

<property>
<name>javax.jdo.option.ConnectionURL</name>
<value>jdbc:mysql://bigdata_mysql:3306/hivedata?useSSL=false&amp;createDatabaseIfNotExist=true</value>
</property>


<property>
<name>javax.jdo.option.ConnectionDriverName</name>
<value>com.mysql.jdbc.Driver</value>
</property>

<property>
<name>javax.jdo.option.ConnectionUserName</name>
<value>hive</value>
</property>
<property>
<name>javax.jdo.option.ConnectionPassWord</name>
<value>hive</value>
</property>


</configuration>
```

上传mysql驱动到hive安装目录的lib文件夹

```
 cp /root/mysql-connector-java-5.1.47.jar  /opt/hive/lib/
```

hive 调试模式,在某个命令卡住时,开启调试可以看到真正的错误信息

 hive -hiveconf hive.root.logger=DEBUG,console

测试


