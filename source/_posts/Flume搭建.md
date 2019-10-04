---
title: Flume搭建
date: 2019-10-04 13:50:04
tags:
categories:
- 大数据
- Docker搭建Hadoop环境集群
typora-root-url: Flume搭建
---

### 解压

```
tar -zxvf apache-flume-1.8.0-bin.tar.gz -C /opt/
mv /opt/apache-flume-1.8.0-bin/ /opt/flume  
修改配置文件 flume-env.sh 设置java_home路径

```

添加可执行目录bin到环境变量PATH

<!-- more -->

拷贝到其他节点

```
scp -r /opt/flume/ root@slave1:/opt/
scp -r /opt/flume/ root@slave2:/opt/


```

## 测试

查看版本

```

[root@master flume]# flume-ng version
Flume 1.8.0
Source code repository: https://git-wip-us.apache.org/repos/asf/flume.git
Revision: 99f591994468633fc6f8701c5fc53e0214b6da4f
Compiled by denes on Fri Sep 15 14:58:00 CEST 2017
From source with checksum fbb44c8c8fb63a49be0a59e27316833d
[root@master flume]# 

```