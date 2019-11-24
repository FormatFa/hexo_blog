---
title: Hadoop集群搭建（一）Box搭建Hadoop集群网络配置
date: 2019-10-20 11:33:33
tag:
categories:
- 大数据
- Hadoop集群搭建
typora-root-url: VBox搭建Hadoop集群网络配置
---

> 之前集群用的是仅主机模式,不能连接外网,要用外网时切换成桥接，十分麻烦,现在可以设置成可以虚拟机互ping,同时也可以连接外网的配置了

## 集群规划
>虚拟机系统:CentOS 7
宿主机:Ubuntu 或者 win

机器	Ip	网络	主机名(hostname)	
宿主机	172.18.17.241			
Master虚拟机	192.168.3.10	Nat + 仅主机	master	
Slave1 虚拟机	192.168.3.11	Nat + 仅主机	slave1	
Slave2 虚拟机	192.168.3.12	Nat + 仅主机	slave2	

网络要求
1. 虚拟机设置静态ip地址,网段和宿主机的不一样 (用来设置host映射),

2. 三台虚拟机互相ping(集群通讯)

3. 主机ping三台虚拟机(传输文件)

4. 三台虚拟机连接外网(用yum安装一些软件)

5. 三台虚拟机ping主机

   <!-- more -->

其他不可行方案:

网络方式	

桥接模式	如果虚拟机设置的静态ip和net1的网段不一样，就无法连接外网
修改宿主机的网段为虚拟机需要的?没测试	
Nat 模式	只能连接外网,无法虚拟机互ping	
仅主机	虚拟机可以互ping,主机可以ping虚拟机,虚拟机应该可以ping主机	
Nat + 仅主机 (虚拟机配置双网卡)	满足	

![1571542933047](1571542933047.png)

## VBox 配置

### VBox 仅主机网络设置

创建一个仅主机网络(设置ip地址和集群的在统一网段)

![1571542957232](1571542957232.png)

## 虚拟机配置

### 三台虚拟机设置

以master为例

使用两张网卡(网络适配器),第一张为NAT模式,第二张为仅主机(使用上面创建的)

![1571542972609](1571542972609.png)

![1571542981892](1571542981892.png)

### 修改三台机器网卡配置文件 

master为例

这里网卡名为enp0s3和enp0s8,分别对应nat模式的和仅主机模式的

 

**/etc/sysconfig/network-scripts/ifcfg-enp0s3**


```
TYPE=Ethernet

PROXY_METHOD=none

BROWSER_ONLY=no

BOOTPROTO=dhcp

DEFROUTE=yes

IPV4_FAILURE_FATAL=no

IPV6INIT=yes

IPV6_AUTOCONF=yes

IPV6_DEFROUTE=yes

IPV6_FAILURE_FATAL=no

IPV6_ADDR_GEN_MODE=stable-privacy

NAME=enp0s3

UUID=af6d5285-33f4-4151-9687-246645365916

DEVICE=enp0s3

ONBOOT=yes
```


**/etc/sysconfig/network-scripts/ifcfg-enp0s8**

这个的网关设置为nat模式那的网关


```
TYPE=Ethernet

PROXY_METHOD=none

BROWSER_ONLY=no

BOOTPROTO=static

DEFROUTE=yes

IPV4_FAILURE_FATAL=no

IPV6INIT=yes

IPV6_AUTOCONF=yes

IPV6_DEFROUTE=yes

IPV6_FAILURE_FATAL=no

IPV6_ADDR_GEN_MODE=stable-privacy

NAME=enp0s8

UUID=af6d5285-33f4-4151-9687-246645365916

DEVICE=enp0s8

ONBOOT=yes

IPADDR=192.168.3.11


GATEWAY=10.0.2.2
```

## 



## 测试

`service network restart 重启网络`

> 下面是个人理解,某些词语用的不是很准确,表达的可能不够专业

### 1．查看路由

### 2. 查看ip 地址

###  3．效果

1. 虚拟机互Ping

   ![1571543341398](1571543341398.png)

2.  虚拟机ping外网

   ![1571543350682](1571543350682.png)

3. 虚拟机Ping主机

   (主机ip为172.18.17.241)

   ![1571543361943](1571543361943.png)

4.  主机ping虚拟机

   (这里宿主机时ubuntu系统的)

![1571543374188](1571543374188.png)



