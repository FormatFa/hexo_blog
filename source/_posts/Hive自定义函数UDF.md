---
title: Hive自定义函数UDF
date: 2019-10-15 09:24:25
tags:
- hive
---

### 创建UDF 函数流程

1. 编写jar
2. 添加jar 到hive
3. 注册函数

函数有临时函数和永久函数,永久函数注册如果不指定数据库会在当前数据库下，名字为dbname.functionname

### 1. 编写打包jar 包

hive maven 的配置(2.7.7为对应的hadoop版本,1.2.2为hive版本)

```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>1.2.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.7</version>
        </dependency>
    </dependencies>
```

#### UDF

UDF为处理某列数据

- 编写一个类继承UDF类

- 定义`evaluate`函数

  如

```java
public final class Lower extends UDF {
  public Text evaluate(final Text s) {
    if (s == null) { return null; }
    return new Text(s.toString().toLowerCase());
  }
}
```

<!-- more -->

### 2. 添加jar包到hive 中

```
添加本地jar
add jar /root/xx.jar;
添加hdfs上的jar
add jar hdfs:/xx.jar
```

### 3. 注册函数

创建临时函数语法

```
class_name 为上面创建的继承UDF的类
CREATE TEMPORARY FUNCTION function_name AS class_name;
```

创建永久函数语法

```
可以使用using jar 添加jar包,如果hive不是本地模式,using jar 的文件要在不是本地的路径,比如hdfs上
CREATE FUNCTION [db_name.]function_name AS class_name
 [USING JAR|FILE|ARCHIVE 'file_uri' [, JAR|FILE|ARCHIVE 'file_uri'] ];	
```

删除函数

```
DROP TEMPORARY FUNCTION [IF EXISTS] function_name;
DROP FUNCTION [IF EXISTS] function_name;
```

查看创建的函数

```
查看所有函数
show functions;
过滤查看函数(通过正则表达式)
show functions like 'sub*';

```



注册后,就可以直接在hql里使用了

参考连接

- https://cwiki.apache.org/confluence/display/Hive/HivePlugins
- 