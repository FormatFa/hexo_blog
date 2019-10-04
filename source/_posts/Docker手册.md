---
title: Docker手册
tags:
 - 手册
categories:
- 手册
date: 2019-09-21 16:50:50
---


### 常用操作
#### 进入容器命令行

`docker exec -it dockertest  bash`

####  获取容器IP地址
```
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'  dockertest 
```

本机上传文件到docker 容器里

`docker cp 本地目录 容器ID:容器内目录`

### Image 镜像

#### 查看所有的镜像

docker images

#### 搜索镜像
docker search 关键字
<!-- more -->
### Network 网络
#### 查看所有网络
`docker network ls`
#### 创建网络
`docker network create -d bridge net_mysql`
#### 创建容器时指定网络
`--network net_mysql`
#### 手动添加容器到网络
`docker network connect 网络 容器`

### 容器
#### 创建容器

`docker create [选项] 镜像 [命令][参数]`

- --name 镜像名字
- 

#### 导出容器文件系统到本地
` docker export master > master.tar`