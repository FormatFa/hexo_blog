---
title: centos7安装mysql
date: 2019-11-22 17:12:21
tags:
typora-root-url: centos7安装mysql
---

下载mysql 社区版 安装包

下载地址

https://dev.mysql.com/downloads/

选择对应的mysql版本，和系统

https://dev.mysql.com/downloads/mysql/

这里安装到Cent OS 7里，选择如下图

![image-20191120204355796](/image-20191120204355796.png)



下载的文件为` mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar`

### 安装

发送安装包到centos里



解压后，yum install *.rpm

期间可能需要下载perl，yum会自动提示下载

### 设置mysql服务

启动mysql服务

` service mysqld start`



查看临时密码`cat /var/log/mysqld.log`

![image-20191120210756164](/image-20191120210756164.png)

使用初始密码进入，修改密码

![image-20191120210902692](/image-20191120210902692.png)

修改密码:

`ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';`

密码可能有强度要求,想修改成1234等简单密码，需要修改密码策略

```
 set global validate_password_policy=0;
 set global validate_password_length =0;
```

开启远程访问权限

 update user set host = '%' where user = 'root'; 

![image-20191120212448626](/image-20191120212448626.png)

host变成%号，就能任意访问了