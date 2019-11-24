---
title: 连接Hive的方式
date: 2019-11-21 22:32:39
tags:
- hive
---

 连接Hive 的方式

- 直接输入hive启动
- 使用beeline

> beeline 会取代直接hive的方式

### 直接输入hive的方式

缺点:多个窗口输入hive时,每个都启动了一个进程

hive命令 启动流程

1. 判断是不是cygwin

```shell

cygwin=false
case "`uname`" in
   CYGWIN*) cygwin=true;;
esac
```

2. 获取当前hive命令的目录，配置到bin变量

   ```shell
   
   bin=`dirname "$0"`
   bin=`cd "$bin"; pwd`
   ```

3. 执行`hive-config.sh`,配置hive配置文件的目录

   配置hive conf的目录,只要设置`HIVE_CONF_DIR`的目录,如果通过了命令行(--config)等设置了，就使用设置的,如果没有,就使用hive目录下的conf目录

   ```shell
   HIVE_CONF_DIR="${HIVE_CONF_DIR:-$HIVE_HOME/conf}"
   ```

   设置`HADOOP_HEAPSIZE`,默认是256m

   ```shell
   export HADOOP_HEAPSIZE=${HADOOP_HEAPSIZE:-256}
   ```

4. 根据参数获取要运行的服务

   设置到`SERVICE`变量,通过--version或者`--service`参数指定

   如果没有配置，先判断有木有设置成help,如果没有，`SERVICE`就是`cli`

   

5. 判断有没有设置`hive-env.sh`,有就启动

```shell
if [ -f "${HIVE_CONF_DIR}/hive-env.sh" ]; then
  . "${HIVE_CONF_DIR}/hive-env.sh"
fi
```

6. 判断有没有设置`SPARK_HOME`

   不知道干了什么

   ```shell
   
   if [[ -z "$SPARK_HOME" ]]
   then
     bin=`dirname "$0"`
     # many hadoop installs are in dir/{spark,hive,hadoop,..}
     if test -e $bin/../../spark; then 
       sparkHome=$(readlink -f $bin/../../spark)
       if [[ -d $sparkHome ]]
       then
         export SPARK_HOME=$sparkHome
       fi
     fi
   fi
   ```

7. 设置`CLASSPATH`为`HIVE_CONF_DIR`

   log4j这些文件就可以放在CONF目录，这样就会在CLASSPATH被加载了

   ```shell
   CLASSPATH="${HIVE_CONF_DIR}"
   ```

8. 设置`HIVE_LIB`的目录

   ```shell
   HIVE_LIB=${HIVE_HOME}/lib
   ```

   

9. 判断`hive-exec-xx.jar`存不存在,不存在就报错

   ```
   if [ ! -f ${HIVE_LIB}/hive-exec-*.jar ]; then
     echo "Missing Hive Execution Jar: ${HIVE_LIB}/hive-exec-*.jar"
     exit 1;
   fi
   ```

   下面还判断其他两个jar

   - ${HIVE_LIB}/hive-metastore-*.jar 

   - ${HIVE_LIB}/hive-cli-*.jar ]

10. 添加spark-assembly到CLASSPATH

先判断`SPARK_HOME`村不存在

11. 添加服务的jar，比如serdes

12. 设置压制(supress)HADOOP_HOME的警告,在1.x.x

    ```shell
    export HADOOP_HOME_WARN_SUPPRESS=true 
    ```

    

13. 传递`HADOOP_CLASSPATH`和`HIVE_CLASSPATH`给hadoop

    ```shell
    if [ "$HADOOP_CLASSPATH" != "" ]; then
      export HADOOP_CLASSPATH="${HADOOP_CLASSPATH}:${CLASSPATH}"
    else
      export HADOOP_CLASSPATH="$CLASSPATH"
    fi
    if [ "$HIVE_CLASSPATH" != "" ]; then
      export HADOOP_CLASSPATH="${HADOOP_CLASSPATH}:${HIVE_CLASSPATH}";
    fi
    ```

14.  检查hadoop是否在PATH变量里

    如果在PATH变量里，通过which获取执行路径，在-f 判断是否存在，如果存在，就根据hadoop命令的目录，设置`HADOOP_DIR`的值

15 .确保获取到`HADOOP_HOME`的目录

```shell
HADOOP_HOME=${HADOOP_HOME:-${HADOOP_PREFIX:-$HADOOP_DIR}}
if [ "$HADOOP_HOME" == "" ]; then
  echo "Cannot find hadoop installation: \$HADOOP_HOME or \$HADOOP_PREFIX must be set or hadoop must be in the path";
  exit 4;
fi
```



16. 确保hive使用对应的hadoop版本

    ```shell
    if [ "x$HADOOP_VERSION" == "x" ]; then
        HADOOP_VERSION=$($HADOOP version | awk -F"\t" '/Hadoop/ {print $0}' | cut -d' ' -f 2);
    fi
    
    
    ```

17. 添加HBASE的目录

添加HBASE conf的目录到`HADOOP_CLASSPATH`

18 .执行ext下面的.sh文件

```shell

for i in "$bin"/ext/*.sh ; do
  . $i
done
for i in "$bin"/ext/util/*.sh ; do
  . $i
done
```

19. 运行对应的服务,

    上面执行的ext下面的sh,定义了对应的函数这些，下面服务名应该就是定义的汉斯名

    ```shell
    
    
    SERVICE_LIST=""
    ...
    TORUN=""
    for j in $SERVICE_LIST ; do
      if [ "$j" = "$SERVICE" ] ; then
        TORUN=${j}$HELP
      fi
    done
    
    if [ "$TORUN" = "" ] ; then
      echo "Service $SERVICE not found"
      echo "Available Services: $SERVICE_LIST"
      exit 7
    else
      $TORUN "$@"
    fi
    
    ```

    比如`cli.sh`

    ```shell
    THISSERVICE=cli
    export SERVICE_LIST="${SERVICE_LIST}${THISSERVICE} "
    
    cli () {
      CLASS=org.apache.hadoop.hive.cli.CliDriver
      execHiveCmd $CLASS "$@"
    }
    
    cli_help () {
      CLASS=org.apache.hadoop.hive.cli.CliDriver
      execHiveCmd $CLASS "--help"
    } 
    
    ```

    `exeHiveCmd`的代码为

    ```shell
    
    execHiveCmd () {
      CLASS=$1;
      shift;
    
      # cli specific code
      if [ ! -f ${HIVE_LIB}/hive-cli-*.jar ]; then
        echo "Missing Hive CLI Jar"
        exit 3;
      fi
    
      if $cygwin; then
        HIVE_LIB=`cygpath -w "$HIVE_LIB"`
      fi
    
      # hadoop 20 or newer - skip the aux_jars option. picked up from hiveconf
      exec $HADOOP jar ${HIVE_LIB}/hive-cli-*.jar $CLASS $HIVE_OPTS "$@"
    }
    
    ```

    最后是调用hive-cli-*.jar，将对应参数传递进去



- 什么时候HIVE_LIB下的`hive-exec .xx.jar`会不存在

  

- 

```java
连接Hive的方式HiveAuthzSessionContext类 
public enum CLIENT_TYPE {
    HIVESERVER2, HIVECLI
  };
都有用到SessionState
```





### 使用beeline的方式

使用beeline 需要先启动HiveServer2服务

`hive --service hiveserver2`

打开一个新窗口(或者上面的命令加个&,放入后台)

使用beeline 连接hiveserver2(可先用ss -tnlp查看10000端口有没有开放)

```
!connect jdbc:hive2://localhost:10000
```

通过jps查看 hiveserver 和beeline

```
[root@bigdata ~]# jps -m
5122 RunJar /opt/hive/lib/hive-beeline-1.1.0.jar org.apache.hive.beeline.BeeLine
4579 DataNode
4757 SecondaryNameNode
5333 Jps -m
4886 RunJar /opt/hive/lib/hive-service-1.1.0.jar org.apache.hive.service.server.HiveServer2
4489 NameNode
[root@bigdata ~]# 
```

