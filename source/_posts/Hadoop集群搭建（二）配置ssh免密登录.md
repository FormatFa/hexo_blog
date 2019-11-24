---
title: Hadoop集群搭建（二）配置ssh免密登录
date: 2019-10-20 22:24:46
tags:
categories:
- 大数据
- Hadoop集群搭建
typora-root-url: 配置ssh免密登录
---



前提:

配置好网络,虚拟机可以互相ping



每台设置免密码自己和其他两台机器

![1571581639806](/1571581639806.png)

<!-- more -->

1. 使用ssh-keygen命令生成公钥和私钥

   输入ssh-keygen 全部选默认回车

   ![1571581726603](/1571581726603.png)

2. 使用ssh-copy-id 命令发送公钥到其他两台机器

   ```
   master 为例
   ssh-copy-id master
   ssh-copy-id slave1
   ssh-copy-id slave2
   ```

   

   ![1571581788086](/1571581788086.png)

3. 测试免密登录

> 直接输入ssh  slave1,不用输入密码即可登录就是成功

![1571581825834](/1571581825834.png)

依次设置三台机器免密登录三台机器