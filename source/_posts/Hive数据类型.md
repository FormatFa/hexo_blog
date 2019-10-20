---
title: Hive数据类型
date: 2019-10-11 21:04:50
tags:
- hive笔记
- hive
categories:
- hive笔记
---

https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-HandlingofNULLValues



### 数字类型

- [`TINYINT`](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-tinyint) (1-byte signed integer, from `-128` to `127`)

- [`SMALLINT`](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-smallint) (2-byte signed integer, from `-32,768` to `32,767`)

- ```
  INT/INTEGER (4-byte signed integer, from -2,147,483,648 to 2,147,483,647)
  ```

- [`BIGINT`](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-bigint) (8-byte signed integer, from `-9,223,372,036,854,775,808` to `9,223,372,036,854,775,807`)

- `FLOAT` (4-byte single precision floating point number)

- `DOUBLE` (8-byte double precision floating point number)

- ```
  DOUBLE PRECISION (alias for DOUBLE, only available starting with Hive 2.2.0)
  ```

- `DECIMAL`

  - Introduced in Hive [0.11.0](https://issues.apache.org/jira/browse/HIVE-2693) with a precision of 38 digits
  - Hive [0.13.0](https://issues.apache.org/jira/browse/HIVE-3976) introduced user-definable precision and scale

- `NUMERIC` (same as `DECIMAL`, starting with [Hive 3.0.0](https://issues.apache.org/jira/browse/HIVE-16764))

Date/Time 类型

- [`TIMESTAMP`](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-timestamp) (Note: Only available starting with Hive [0.8.0](https://issues.apache.org/jira/browse/HIVE-2272))
- [`DATE`](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-date) (Note: Only available starting with Hive [0.12.0](https://issues.apache.org/jira/browse/HIVE-4055))
- [`INTERVAL`](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-Intervals) (Note: Only available starting with Hive [1.2.0](https://issues.apache.org/jira/browse/HIVE-9792))

<!-- more -->

### String Types

- [`STRING`](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-string)
- [`VARCHAR`](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-varchar) (Note: Only available starting with Hive [0.12.0](https://issues.apache.org/jira/browse/HIVE-4844))
- [`CHAR`](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-char) (Note: Only available starting with Hive [0.13.0](https://issues.apache.org/jira/browse/HIVE-5191))

### 其他类型

- `BOOLEAN`
- `BINARY` (Note: Only available starting with Hive [0.8.0](https://issues.apache.org/jira/browse/HIVE-2380))

### 复杂类型

- arrays: `ARRAY<data_type>` (Note: negative values and non-constant expressions are allowed as of [Hive 0.14](https://issues.apache.org/jira/browse/HIVE-7325).)
- maps: `MAP<primitive_type, data_type>` (Note: negative values and non-constant expressions are allowed as of [Hive 0.14](https://issues.apache.org/jira/browse/HIVE-7325).)
- structs: `STRUCT<col_name : data_type [COMMENT col_comment], ...>`
- union: `UNIONTYPE<data_type, data_type, ...>` (Note: Only available starting with Hive [0.7.0](https://issues.apache.org/jira/browse/HIVE-537).)

## 列类型

### 整数

### 字符串string

字符串可以用单引号或者双引号表示

### Varchar

varchar 创建时指定最大的长度(在1到65535之间),超过的会被丢弃

varchar和string一样,字符串后面的空格会影响比较的结果

### char

char 和 varchar类似,最大的长度是255,小于的会用空格补充

后面的空格不会影响比较的结果

### timestamps

支持传统的UNIX时间戳,和可选的纳秒精度

支持的转换

- 整数数字类型:解析成秒的UNIX时间戳
- 浮点数字类型:解析成带有小数的秒的UNIX时间戳
- 字符串:遵循JDBC java.sql.Timestamp 格式"`YYYY-MM-DD HH:MM:SS.fffffffff`" (9 decimal place precision)





### Dates

Date类型表示特定的year/month/day,用YYYY-MM-DD的形式,比如'`2019-09-02`,支持的范围为0000-01-01到9999-12-31

> Date类型在Hive 0.12里引入

### Casting Dates

Date只能由Date,Timestamp,字符串 转换来或者转换成

指定转换的格式:https://cwiki.apache.org/confluence/display/Hive/CAST...FORMAT+with+SQL%3A2016+datetime+formats

| 有效的转换              |                                                              |
| ----------------------- | ------------------------------------------------------------ |
| cast(timestamp as date) | The year/month/day of the timestamp is determined, based on the local timezone, and returned as a date value. |
| cast(string as date)    | If  the string is in the form 'YYYY-MM-DD', then a date value corresponding  to that year/month/day is returned. If the string value does not match  this formate, then NULL is returned. |
| cast(date as timestamp) | A timestamp value is generated corresponding to midnight of the year/month/day of the date value, based on the local timezone. |
| cast(date as string)    | The year/month/day represented by the Date is formatted as a string in the form 'YYYY-MM-DD'. |
| cast(date as date)      | Same date value                                              |

### Decimal 常量

大于BIGINT的整数常量必须用Decimal(39,0)处理,要用前缀BD

例如

`select CAST(18446744073709001000BD AS DECIMAL(38,0)) from my_table limit 1;`



### 空值处理

缺失值用特殊的NULL值代表. 导入有NULL值的字段表使用的SerDe的文档(默认文本格式使用LazySimpleSerDe ,导入数据时将字符串\N解析为NULL)