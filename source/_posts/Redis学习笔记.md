---
title: Redis学习笔记
date: 2019-10-28 11:23:01
tags:
---

redis 一共有16个数据库,下标0-15

```
切换数据库
select 1
```

redis 数据用key value的格式存储数据

通用操作

| 操作               |                     |      |
| ------------------ | ------------------- | ---- |
| keys *             | 查询库所有值        |      |
| exists key         | 判断键是否存在      |      |
| type key           | 查看键类型          |      |
| del                | 删除键              |      |
| expire key seconds | 设置键过期时间      |      |
| ttl key            | 查看还有多少秒过期  |      |
| dbsize             | 查看数据库key的数量 |      |
| flushdb            | 清空当前库          |      |
| flushall           | 清空所有库          |      |
|                    |                     |      |
|                    |                     |      |
|                    |                     |      |
|                    |                     |      |
|                    |                     |      |
|                    |                     |      |
|                    |                     |      |
|                    |                     |      |
|                    |                     |      |
|                    |                     |      |

redis value 数据类型

- string

最基本的数据类型,一个字符串最大是512m

| 操作                          |                                 |      |
| ----------------------------- | ------------------------------- | ---- |
| get <key>                     | 查询键值                        |      |
| set <key> <value>             | 设置键值                        |      |
| append key value              | 添加value到原值的末尾           |      |
| strlen                        | 获取值长度                      |      |
| setnx key value               | key不存在时设置key的值          |      |
| incr                          | 值增1，值只能是数字             |      |
| desc                          | 值减1，同上                     |      |
| incrby/decr key 长度          | 增加减掉指定值                  |      |
| mset key1 value1 key2 value2  | 同时设置一个或者多个key-value对 |      |
| mget key1 , key2              | 获取多个值                      |      |
| msetnx key1 value key2 value2 | 设置多个值,但key不存在时        |      |
| getrange key start end        | 获取值的范围                    |      |
| setrange key start end        | 设置值                          |      |
| setex key 过期时间 value      | 设置过期时间，单位秒            |      |
| getset key value              | 获取新值，同时设置旧值          |      |
|                               |                                 |      |
|                               |                                 |      |
|                               |                                 |      |
|                               |                                 |      |



- list

  底层是个双向列表

  | 操作                              |                          |      |
  | --------------------------------- | ------------------------ | ---- |
  | lpush/rpush key value1 value2     | 左边或者右边插入值       |      |
  | lpop/rpop                         | pop一个值                |      |
  | rpoplpush                         | 右边取出一个值，插到左边 |      |
  | lrange key start end              | 通过索引下标获取元素     |      |
  | lindex key index                  | 通过索引获取元素         |      |
  | llen                              | 获取列表长度             |      |
  | linsert key before value newvalue | 在value前面插入newvalue  |      |
  | lrem key n value                  | 从左边删除n个value       |      |
  |                                   |                          |      |
  |                                   |                          |      |
  |                                   |                          |      |
  |                                   |                          |      |
  |                                   |                          |      |
  |                                   |                          |      |
  |                                   |                          |      |
  |                                   |                          |      |
  |                                   |                          |      |
  |                                   |                          |      |
  |                                   |                          |      |

  

- set

  没有重复数据

  | 操作                |                           |      |
  | ------------------- | ------------------------- | ---- |
  | sadd key value v2   | 添加值                    |      |
  | smembers key        | 取出所有值                |      |
  | sismember key value | 判断集合是不是有这个value |      |
  | scard               | 集合元素个数              |      |
  | srem key v1 v2....  | 删除元素                  |      |
  | spop                | 随机取出一个值            |      |
  | srandmember key n   | 随机取出n个值             |      |
  | sinter key1 key2    | 取出n个值交集             |      |
  | sunion key1 key2    | 返回两个交集的并集        |      |
  | sdiff               | 返回两个元素的差集        |      |
  |                     |                           |      |
  |                     |                           |      |
  |                     |                           |      |
  |                     |                           |      |
  |                     |                           |      |
  |                     |                           |      |
  |                     |                           |      |
  |                     |                           |      |
  |                     |                           |      |

  

- hash

hash是个键值对集合,<key,value>

| 操作                        |                                          |      |
| --------------------------- | ---------------------------------------- | ---- |
| hset key field value        | 设置key集合里的field的值为value          |      |
| hget key field              | 获取field                                |      |
| hmset key field1 field2     | 批量设置hash的值                         |      |
| hexists key field           | 判断field是否存在                        |      |
| hkeys key                   | 获取所有field                            |      |
| hvals key                   | 获取所有value                            |      |
| hincrby key field increment | 给指定的field加上增量increment           |      |
| hsetnx key field value      | 当field不存在时，设置field的value为value |      |
|                             |                                          |      |
|                             |                                          |      |
|                             |                                          |      |
|                             |                                          |      |
|                             |                                          |      |
|                             |                                          |      |
|                             |                                          |      |
|                             |                                          |      |
|                             |                                          |      |
|                             |                                          |      |
|                             |                                          |      |



- zset

  类似set,但是每个元素都关联了一个评分(score),用这个评分来排序集合里的元素

|                                                         |                                     |
| ------------------------------------------------------- | ----------------------------------- |
| zadd key score value score value...                     | 将一个或者多个元素添加到key的集合里 |
| zrange key start stop withscores                        | 返回下标在start 到 stop之间的元素,  |
| zrangebyscore key min max withscores limit offset count | 返回score在min到max之间的成员       |
| zrangebyscore key max min withscores limit offset count | 同上，大到小                        |
| zincrby key increment  value                            | 为元素score加上增量                 |
| zrem key value                                          | 删除指定值的元素                    |
| zcount key min max                                      | 统计分数区间内的个数                |
| zrank key value                                         | 返回值在集合里的排名，0开始         |
|                                                         |                                     |
|                                                         |                                     |
|                                                         |                                     |
|                                                         |                                     |
|                                                         |                                     |
|                                                         |                                     |
|                                                         |                                     |
|                                                         |                                     |
|                                                         |                                     |
|                                                         |                                     |
|                                                         |                                     |

