---
title: Hadoop_Namenode高可用测试
date: 2019-10-21 09:59:38
tags:
categories:
- 大数据
- Hadoop集群搭建
typora-root-url: Hadoop-Namenode高可用测试
---



测试方法:

使用kill 命令杀死namenode,模拟namenode停止

效果:

备用namenode转换为active,重新启动被杀死的namenode,状态变成standby



备用切换成active之前，要确保之前那个namenode凉透了，不然两个一起namenode active,会发生脑裂

切换的方式在hdfs-site.xml里配置

```xml
<property>
<name>dfs.ha.fencing.methods</name>
<value>sshfence
shell(/bin/true)</value>
</property>
```

<!-- more -->

上面配置的两种方式,只要一个成功就可以转换为active,第一个sshfence,需要用到fuser命令,centos mini默认没有安装这个命令,肯定返回错误,第一个肯的返回true,所以上面这个配置namenode一停,另一个就马上转换成active

这里测试sshfence方式,修改配置文件为

```xml
<property>
<name>dfs.ha.fencing.methods</name>
<value>sshfence</value>
</property>
 <property>
      <name>dfs.ha.fencing.ssh.private-key-files</name>
      <value>/root/.ssh/id_rsa</value>
    </property>
```

安装fuser命令

```
yum install psmisc 
```

### 测试

查看哪个namenode是active的

```
[root@master hadoop]# hdfs haadmin -getServiceState nn1
active
[root@master hadoop]# hdfs haadmin -getServiceState nn2
standby

nn1 即master是active,在master上杀死namenode
查看namenode的进程

[root@master hadoop]# jps
14065 JobHistoryServer
19457 Jps
19111 JournalNode
18936 DataNode
18841 NameNode
19273 DFSZKFailoverController
11678 QuorumPeerMain

杀死
[root@master hadoop]# kill -9 18841
不存在了
[root@master hadoop]# jps
14065 JobHistoryServer
19111 JournalNode
18936 DataNode
19273 DFSZKFailoverController
11678 QuorumPeerMain
19470 Jps

查看下namenode状态
[root@master hadoop]# hdfs haadmin -getServiceState nn1
19/10/20 22:37:00 INFO ipc.Client: Retrying connect to server: master/192.168.3.10:9000. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=1, sleepTime=1000 MILLISECONDS)
Operation failed: Call From master/192.168.3.10 to master:9000 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
[root@master hadoop]# hdfs haadmin -getServiceState nn2
active

nn2变成active了，成功

```

查看下nn2的日志

` cat  logs/hadoop-root-zkfc-slave1.log`

```
2019-10-20 22:36:25,154 INFO org.apache.hadoop.ha.ZKFailoverController: Should fence: NameNode at master/192.168.3.10:9000
2019-10-20 22:36:26,163 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: master/192.168.3.10:9000. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=1, sleepTime=1000 MILLISECONDS)
这里开始连接不到master了
2019-10-20 22:36:26,179 WARN org.apache.hadoop.ha.FailoverController: Unable to gracefully make NameNode at master/192.168.3.10:9000 standby (unable to connect)
java.net.ConnectException: Call From slave1/192.168.3.11 to master:9000 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at org.apache.hadoop.net.NetUtils.wrapWithMessage(NetUtils.java:791)
	at org.apache.hadoop.net.NetUtils.wrapException(NetUtils.java:731)
	at org.apache.hadoop.ipc.Client.call(Client.java:1472)
	at org.apache.hadoop.ipc.Client.call(Client.java:1399)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:232)
	at com.sun.proxy.$Proxy9.transitionToStandby(Unknown Source)
	at org.apache.hadoop.ha.protocolPB.HAServiceProtocolClientSideTranslatorPB.transitionToStandby(HAServiceProtocolClientSideTranslatorPB.java:112)
	at org.apache.hadoop.ha.FailoverController.tryGracefulFence(FailoverController.java:172)
	at org.apache.hadoop.ha.ZKFailoverController.doFence(ZKFailoverController.java:512)
	at org.apache.hadoop.ha.ZKFailoverController.fenceOldActive(ZKFailoverController.java:503)
	at org.apache.hadoop.ha.ZKFailoverController.access$1100(ZKFailoverController.java:61)
	at org.apache.hadoop.ha.ZKFailoverController$ElectorCallbacks.fenceOldActive(ZKFailoverController.java:890)
	at org.apache.hadoop.ha.ActiveStandbyElector.fenceOldActive(ActiveStandbyElector.java:902)
	at org.apache.hadoop.ha.ActiveStandbyElector.becomeActive(ActiveStandbyElector.java:801)
	at org.apache.hadoop.ha.ActiveStandbyElector.processResult(ActiveStandbyElector.java:416)
	at org.apache.zookeeper.ClientCnxn$EventThread.processEvent(ClientCnxn.java:599)
	at org.apache.zookeeper.ClientCnxn$EventThread.run(ClientCnxn.java:498)
Caused by: java.net.ConnectException: Connection refused
	at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
	at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:717)
	at org.apache.hadoop.net.SocketIOWithTimeout.connect(SocketIOWithTimeout.java:206)
	at org.apache.hadoop.net.NetUtils.connect(NetUtils.java:530)
	at org.apache.hadoop.net.NetUtils.connect(NetUtils.java:494)
	at org.apache.hadoop.ipc.Client$Connection.setupConnection(Client.java:607)
	at org.apache.hadoop.ipc.Client$Connection.setupIOstreams(Client.java:705)
	at org.apache.hadoop.ipc.Client$Connection.access$2800(Client.java:368)
	at org.apache.hadoop.ipc.Client.getConnection(Client.java:1521)
	at org.apache.hadoop.ipc.Client.call(Client.java:1438)
	... 14 more
	开始隔离
2019-10-20 22:36:26,193 INFO org.apache.hadoop.ha.NodeFencer: ====== Beginning Service Fencing Process... ======
尝试第一个sshfence方式
2019-10-20 22:36:26,195 INFO org.apache.hadoop.ha.NodeFencer: Trying method 1/1: org.apache.hadoop.ha.SshFenceByTcpPort(null)
2019-10-20 22:36:26,279 INFO org.apache.hadoop.ha.SshFenceByTcpPort: Connecting to master...
2019-10-20 22:36:26,280 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Connecting to master port 22
2019-10-20 22:36:26,288 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Connection established
2019-10-20 22:36:26,300 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Remote version string: SSH-2.0-OpenSSH_7.4
2019-10-20 22:36:26,300 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Local version string: SSH-2.0-JSCH-0.1.42
2019-10-20 22:36:26,300 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: CheckCiphers: aes256-ctr,aes192-ctr,aes128-ctr,aes256-cbc,aes192-cbc,aes128-cbc,3des-ctr,arcfour,arcfour128,arcfour256
2019-10-20 22:36:26,699 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: aes256-ctr is not available.
2019-10-20 22:36:26,699 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: aes192-ctr is not available.
2019-10-20 22:36:26,699 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: aes256-cbc is not available.
2019-10-20 22:36:26,699 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: aes192-cbc is not available.
2019-10-20 22:36:26,699 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: arcfour256 is not available.
2019-10-20 22:36:26,700 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: SSH_MSG_KEXINIT sent
2019-10-20 22:36:26,700 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: SSH_MSG_KEXINIT received
2019-10-20 22:36:26,700 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: kex: server->client aes128-ctr hmac-sha1 none
2019-10-20 22:36:26,700 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: kex: client->server aes128-ctr hmac-sha1 none
2019-10-20 22:36:26,717 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: SSH_MSG_KEXDH_INIT sent
2019-10-20 22:36:26,717 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: expecting SSH_MSG_KEXDH_REPLY
rsa 验证成功
2019-10-20 22:36:26,741 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: ssh_rsa_verify: signature true
2019-10-20 22:36:26,743 WARN org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Permanently added 'master' (RSA) to the list of known hosts.
2019-10-20 22:36:26,743 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: SSH_MSG_NEWKEYS sent
2019-10-20 22:36:26,743 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: SSH_MSG_NEWKEYS received
2019-10-20 22:36:26,766 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: SSH_MSG_SERVICE_REQUEST sent
2019-10-20 22:36:26,768 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: SSH_MSG_SERVICE_ACCEPT received
2019-10-20 22:36:26,770 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Authentications that can continue: gssapi-with-mic,publickey,keyboard-interactive,password
2019-10-20 22:36:26,770 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Next authentication method: gssapi-with-mic
2019-10-20 22:36:26,773 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Authentications that can continue: publickey,keyboard-interactive,password
2019-10-20 22:36:26,773 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Next authentication method: publickey
2019-10-20 22:36:26,896 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Authentication succeeded (publickey).
2019-10-20 22:36:26,898 INFO org.apache.hadoop.ha.SshFenceByTcpPort: Connected to master
在master查找进程,
2019-10-20 22:36:26,898 INFO org.apache.hadoop.ha.SshFenceByTcpPort: Looking for process running on port 9000
2019-10-20 22:36:27,291 INFO org.apache.hadoop.ha.SshFenceByTcpPort: Indeterminate response from trying to kill service. Verifying whether it is running using nc...
用nc命令判断是不是在运行
2019-10-20 22:36:27,366 WARN org.apache.hadoop.ha.SshFenceByTcpPort: nc -z master 9000 via ssh: bash: nc: command not found
找不到nc命令?
2019-10-20 22:36:27,370 INFO org.apache.hadoop.ha.SshFenceByTcpPort: Verified that the service is down.
2019-10-20 22:36:27,370 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Disconnecting from master port 22
2019-10-20 22:36:27,372 INFO org.apache.hadoop.ha.NodeFencer: ====== Fencing successful by method org.apache.hadoop.ha.SshFenceByTcpPort(null) ======
2019-10-20 22:36:27,372 INFO org.apache.hadoop.ha.ActiveStandbyElector: Writing znode /hadoop-ha/ns/ActiveBreadCrumb to indicate that the local node is the most recent active...
2019-10-20 22:36:27,374 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Caught an exception, leaving main loop due to Socket closed
将nn2转换为active
2019-10-20 22:36:27,599 INFO org.apache.hadoop.ha.ZKFailoverController: Trying to make NameNode at slave1/192.168.3.11:9000 active...
2019-10-20 22:36:30,408 INFO org.apache.hadoop.ha.ZKFailoverController: Successfully transitioned NameNode at slave1/192.168.3.11:9000 to active state

```

