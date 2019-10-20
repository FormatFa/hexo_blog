---
title: Hive_Cli
date: 2019-10-11 20:24:40
tags:
- hive
- hive笔记
categories:
- hive笔记
---

Hive 客户端 client

$HIVE_HOME/bin/hive is a shell utility which can be used to run Hive queries in either interactive or batch mode.

$HIVE_HOME/bin/hive 可以用来交互式shell和批处理模式里执行HQL 查询

HiveServer2 (在Hive 0.11里引入) 有自己的CLI叫做 Beeline,是个基于SQLLine的 JDBC客户端

Hive CLi 将来可能会过期,用HiveServer2代替

### Hive命令行参数

使用`hive -H或者hive --help`查看帮助

```
usage: hive
 -d,--define <key=value>          Variable substitution to apply to Hive
                                  commands. e.g. -d A=B or --define A=B
 -e <quoted-query-string>         SQL from command line
 -f <filename>                    SQL from files
 -H,--help                        Print help information
 -h <hostname>                    连接到远程host的Hive ServerConnecting to Hive Server on remote host
 
    --hiveconf <property=value>   Use value for given property
    --hivevar <key=value>         Variable substitution to apply to hive
                                  commands. e.g. --hivevar A=B
 -i <filename>                    Initialization SQL file
 -p <port>                        Connecting to Hive Server on port number
 -S,--silent                      Silent mode in interactive shell
 -v,--verbose                     Verbose mode (echo executed SQL to the
                                  console)
从Hive 0.10.0开始有个额外的命令
--database 	指定使用的数据库
支持
-hiveconf	和 --hiveconf
```

<!-- more -->

### hiverc文件

CLI 不带 -i 选项时会默认加载 $HIVE_HOME/bin/.hiverc和$HOME/.hiverc 作为初始化文件

Logging 日志

Hive 使用log4j来打印日志,默认日志不会输出到标准输出,但会输出到Hive 的log4j 配置文件指定的文件.

默认使用的配置文件是 Hive 安装目录的conf/下的`hive-log4j.default`,日志文件写出到/temp/<userid>/hive.log,日志等级是WARN

如果想输出日志到标准输出,和改变日志等级，可以在命令行里指定

```
$HIVE_HOME/bin/hive --hiveconf hive.root.logger=INFO,console
```

hive.root.logger 指定日志的等级和日志的输出位置,

### Hive 批处理模式命令

用 -e 或者 -f选项时会用批处理模式执行SQL命令

- hive -e '<query-string>'
- hive -f <filepath>

> 从Hive 0.14 开始<filepath>支持时Hadoop 支持的文件系统里的文件(hdfs)

### Hive Resources 

hive 可以管理一些执行查询时用到的文件，比如普通文件,jars,archives压缩包,本地访问的文件都可以添加到Hive的会话里

用法

```
ADD { FILE[S] | JAR[S] | ARCHIVE[S] } <filepath1> [<filepath2>]*
   LIST { FILE[S] | JAR[S] | ARCHIVE[S] } [<filepath1> <filepath2> ..]
   DELETE { FILE[S] | JAR[S] | ARCHIVE[S] } [<filepath1> <filepath2> ..]
```



- FILES 文件只是添加到distrubuted cache,可能是一些用来执行的转换(transform)脚本
- JARS 资源被添加到Java 的classpath,比如用UDF时
- ARCHIVE 压缩包资源会被自动解压,

例子

```
hive> add FILE /tmp/tt.py;
  hive> list FILES;
  /tmp/tt.py
  hive> select from networks a 
               MAP a.networkid 
               USING 'python tt.py' as nn where a.ds = '2009-01-04' limit 10;
```

### HCatalog CLI