---
title: Hive基本介绍
date: 2019-10-11 18:06:52
tags:
- hive笔记
- 大数据
categories:
- hive
---

文章连接:

https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-HiveTutorial

### 概念

Hive 是一个基于Hadoop 的数据仓库

### 数据单位

按照粒度的大小可以分为

- DataBases

  数据库

- Tables

  表

- Partitions

  分区

- Buckets

  桶

### 数据类型

数据类型分为原始数据类型，复杂数据类型

<!-- more -->

#### 原始数据类型

- Integers

  整数

  - TINYINT—1 byte integer
  - SMALLINT—2 byte integer
  - INT—4 byte integer
  - BIGINT—8 byte integer

- Boolean type

  - BOOLEAN—TRUE/FALSE

- 浮点数

  - FLOAT—single precision
  - DOUBLE—Double precision

- Fixed point numbers

  - DECIMAL—a fixed point value of user defined scale and precision

    定点数 还是指定 规格的小数

- String types

  - STRING—sequence of characters in a specified character set
  - VARCHAR— 有最大长度 sequence of characters in a specified character set with a maximum length
  - CHAR—指定长度的 sequence of characters in a specified character set with a defined length 

- Date and time types

  - TIMESTAMP — A date and time without a timezone ("LocalDateTime" semantics) 没有时区日期时间
  - TIMESTAMP WITH LOCAL TIME ZONE — A point in time measured down to nanoseconds ("Instant" semantics)
  - DATE—a date 日期

- Binary types

  - BINARY—a sequence of bytes

### 复杂数据类型

复杂类型由原始类型或者其他混合类型组成

- Structs

  结构体 , 元素通过.来访问,如`xx.a`

- Maps 

  键值对的

- Arrays

  数组,必须是相同类型的,通过 [n]索引来访问



### 内置功能和函数

```
查看所有函数
show functions
查看函数信息
describle function <function_name>
查看函数详情
describle function extended <function_name>
```



> hive的关键词是不区分大小写的

### 内置操作

- 关系操作

| 关系操作符    | 操作数类型 | 描述                                                         |
| ------------- | ---------- | ------------------------------------------------------------ |
| A = B         | 所有类型   |                                                              |
| A != B        |            |                                                              |
| A < B         |            |                                                              |
| A <= B        |            |                                                              |
| A > B         |            |                                                              |
| A >= B        |            |                                                              |
| A IS NULL     |            |                                                              |
| A IS NOT NULL |            |                                                              |
| A LIKE B      | strings    |                                                              |
| A RLIKE B     | strings    | 如果A满足简单的SQL 规则B _符号匹配任意字符,%匹配任意数量的字符 %和;转义用\ |
| A REGEXP B    | strings    | 和RLIKE一样                                                  |



| 算术操作符 | 操作数类型 |      |
| ---------- | ---------- | ---- |
| A + B      | 所有类型   |      |
| A - B      |            |      |
| A * B      |            |      |
| A / B      |            |      |
| A %  B     |            |      |
| A & B      |            |      |
| A \| B     |            |      |
| A ^ B      |            |      |
| ~A         |            |      |


| 逻辑操作符 |         |      |
| ---------- | ------- | ---- |
| A AND B    | boolean |      |
| A && B     |         |      |
| A OR B     |         |      |
| A\|\|B     |         |      |
| NOT A      |         |      |
| !A         |         |      |



| 复杂类型 |                            |      |
| -------- | -------------------------- | ---- |
| A[n]     | 数组                       |      |
| M[key]   | M是Map<K,V>类型,key是K类型 |      |
| S.x      | s是struct                  |      |

### 内置函数

| 返回类型          | 函数签名                                         | 描述                                                         |
| ----------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| T                 | round(double a)                                  | returns the rounded BIGINT value of the double               |
| BIGINT            | floor(double a)                                  | returns the maximum BIGINT value that is equal or less than the double |
| BIGINT            | ceil(double a)                                   | returns the minimum BIGINT value that is equal or greater than the double |
| double            | rand(), rand(int seed)                           | returns  a random number (that changes from row to row). Specifiying the seed  will make sure the generated random number sequence is deterministic. |
| string            | concat(string A, string B,...)                   | returns  the string resulting from concatenating B after A. For example,  concat('foo', 'bar') results in 'foobar'. This function accepts  arbitrary number of arguments and return the concatenation of all of  them. |
| string            | substr(string A, int start)                      | returns  the substring of A starting from start position till the end of string  A. For example, substr('foobar', 4) results in 'bar' |
| string            | substr(string A, int start, int length)          | returns the substring of A starting from start position with the given length, for example,  substr('foobar', 4, 2) results in 'ba' |
| string            | upper(string A)                                  | returns the string resulting from converting all characters of A to upper case, for example, upper('fOoBaR') results in 'FOOBAR' |
| string            | ucase(string A)                                  | Same as upper                                                |
| string            | lower(string A)                                  | returns the string resulting from converting all characters of B to lower case, for example, lower('fOoBaR') results in 'foobar' |
| string            | lcase(string A)                                  | Same as lower                                                |
| string            | trim(string A)                                   | returns the string resulting from trimming spaces from both ends of A, for example, trim(' foobar ') results in 'foobar' |
| string            | ltrim(string A)                                  | returns  the string resulting from trimming spaces from the beginning(left hand  side) of A. For example, ltrim(' foobar ') results in 'foobar ' |
| string            | rtrim(string A)                                  | returns  the string resulting from trimming spaces from the end(right hand side)  of A. For example, rtrim(' foobar ') results in ' foobar' |
| string            | regexp_replace(string A, string B, string C)     | returns the string resulting from replacing all substrings in B that match the Java regular expression syntax(See [Java regular expressions syntax](http://java.sun.com/j2se/1.4.2/docs/api/java/util/regex/Pattern.html)) with C. For example, regexp_replace('foobar', 'oo\|ar', ) returns 'fb' |
| int               | size(Map<K.V>)                                   | returns the number of elements in the map type               |
| int               | size(Array<T>)                                   | returns the number of elements in the array type             |
| *value of <type>* | cast(*<expr>* as *<type>*)                       | converts  the results of the expression expr to <type>, for example,  cast('1' as BIGINT) will convert the string '1' to it integral  representation. A null is returned if the conversion does not succeed. |
| string            | from_unixtime(int unixtime)                      | convert  the number of seconds from the UNIX epoch (1970-01-01 00:00:00 UTC) to a  string representing the timestamp of that moment in the current system  time zone in the format of "1970-01-01 00:00:00" |
| string            | to_date(string timestamp)                        | 返回时间戳字符串的日期部分  Return the date part of a timestamp string: to_date("1970-01-01 00:00:00") = "1970-01-01" |
| int               | year(string date)                                | Return the year part of a date or a timestamp string: year("1970-01-01 00:00:00") = 1970, year("1970-01-01") = 1970 |
| int               | month(string date)                               | Return the month part of a date or a timestamp string: month("1970-11-01 00:00:00") = 11, month("1970-11-01") = 11 |
| int               | day(string date)                                 | Return the day part of a date or a timestamp string: day("1970-11-01 00:00:00") = 1, day("1970-11-01") = 1 |
| string            | get_json_object(string json_string, string path) | Extract  json object from a json string based on json path specified, and return  json string of the extracted json object. It will return null if the  input json string is invalid. |

### 内置聚合函数

| 返回类型 | 函数签名                                              | 描述                                                         |
| -------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| T        | count(*), count(expr), count(DISTINCT expr[, expr_.]) | count(*)—Returns the total number of retrieved rows, including rows containing NULL values; count(expr)—Returns the number of rows for which the supplied expression is non-NULL; count(DISTINCT expr[, expr])—Returns the number of rows for which the supplied expression(s) are unique and non-NULL. |
| DOUBLE   | sum(col), sum(DISTINCT col)                           | returns the sum of the elements in the group or the sum of the distinct values of the column in the group |
| DOUBLE   | avg(col), avg(DISTINCT col)                           | returns the average of the elements in the group or the average of the distinct values of the column in the group |
| DOUBLE   | min(col)                                              | returns the minimum value of the column in the group         |
| DOUBLE   | max(col)                                              | returns the maximum value of the column in the group         |

### Hive 提供基本的SQL操作

- 使用where条件过滤任意的行

- 使用select语句从表里选择合适的列

- 使用join连接两张表

- 在一个表里使用多个group by 求值

- 保存查询结果到其他表

- 下载查询结果到本地目录

- 保存查询结果到hadoop dfs 目录

- 管理表和分区

- Ability to plug in custom scripts in the language of choice for custom map/reduce jobs.

  ---

  