---
title: "Redis Note"
date: 2022-06-07T21:13:49+08:00
draft: true
weight: 9
categories: ["Redis"]
tags: ["redis"]
author: ["lavaicer"]
# author: ["Me", "You"] # multiple authors
---

## NoSQL

### 概述

Not only SQL - 非关系型数据库

不依赖业务逻辑方式存储,而以简单的 key-value 方式存储.

- 不遵循 SQL 标准
- 不支持 ACID
- 远超 SQL 的性能

### 适用场景

- 对数据高并发的读写
- 海量数据的读写
- 对数据高扩展性的

### 不适用场景

- 需要事务支持
- 需要 SQL 结构化查询存储,处理复杂的关系

## Centos 安装 Redis

1. 官网下载 https://redis.io/download/#redis-downloads
2. `tar -zxvf redis.tar.gz`
3. `cd redis && make && make install` - 默认安装到 `/usr/local/bin`
   > 在 `/usr/local/bin` 目录下:
   > redis-benchmark:性能测试工具
   > redis-check-aof:修复有问题的 AOF 文件
   > redis-check-rdb:修复有问题的 dump.rdb 文件
   > redis-sentinel:集群使用
   > redis-server:Redis 服务器启动命令
   > redis-cli:客户端操作入口
4. `cp redis.conf /etc/redis.conf`
   > 修改 `/etc/redis.conf` `daemonize yes`
   > 启动 `redis-server /etc/redis.conf`
   > 关闭 `shutdown`

### 相关知识

端口:6379

默认 16 个数据库,类似数组下标从零开始,初始默认零号库

使用 `select <id>` 来切换数据库

所有库统一密码

`dbsize` 查看当前数据库的 key

`flushdb` - 清空当前库

`flushall` - 全部库

Redis 是单线程+多路 IO 复用技术

## key

keys \*:查看当前数据库的所有 key

exists key:判断某个 key 是否存在

type key:查看 key 的类型

del key:删除指定的 key 的数据

unlink key:根据 value 选择非阻塞式删除(仅将 key 从 keyspace 中删除,真正的删除会在后续异步操作)

expire key 10:未给定的 key 设置过期时间

ttl key:查看 key 还有多少过期时间,-1 代表永不过期,-2 代表已过期

## String

string 类型是二进制安全的,其对应的 value 最大可为 512M

### 常用命令

1. SET key value
   设置指定 key 的值。
2. GET key
   获取指定 key 的值。
3. GETRANGE key start end
   返回 key 中字符串值的子字符
4. GETSET key value
   将给定 key 的值设为 value ，并返回 key 的旧值(old value)。
5. GETBIT key offset
   对 key 所储存的字符串值，获取指定偏移量上的位(bit)。
6. MGET key1 [key2..]
   获取所有(一个或多个)给定 key 的值。
7. SETBIT key offset value
   对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。
8. SETEX key seconds value
   将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。
9. SETNX key value
   只有在 key 不存在时设置 key 的值。
10. SETRANGE key offset value
    用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。
11. STRLEN key
    返回 key 所储存的字符串值的长度。
12. MSET key value [key value ...]
    同时设置一个或多个 key-value 对。
13. MSETNX key value [key value ...]
    同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。有一个存在则失败。
14. PSETEX key milliseconds value
    这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。
15. INCR key
    将 key 中储存的数字值增一。
16. INCRBY key increment
    将 key 所储存的值加上给定的增量值（increment） 。
17. INCRBYFLOAT key increment
    将 key 所储存的值加上给定的浮点增量值（increment） 。
18. DECR key
    将 key 中储存的数字值减一。
19. DECRBY key decrement
    key 所储存的值减去给定的减量值（decrement） 。
20. APPEND key value
    如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。

## 数据类型

### List

List 的数据结构为快速列表 quickList

列表元素较少时会采用一块连续的内存存储，zipList(压缩列表)

元素多时采用 quickList，因为前后指针占用空间。

1. BLPOP key1 [key2 ] timeout
   移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
2. BRPOP key1 [key2 ] timeout
   移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
3. BRPOPLPUSH source destination timeout
   从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
4. LINDEX key index
   通过索引获取列表中的元素
5. LINSERT key BEFORE|AFTER pivot value
   在列表的元素前或者后插入元素
6. LLEN key
   获取列表长度
7. LPOP key
   移出并获取列表的第一个元素
8. LPUSH key value1 [value2]
   将一个或多个值插入到列表头部
9. LPUSHX key value
   将一个值插入到已存在的列表头部
10. LRANGE key start stop
    获取列表指定范围内的元素
11. LREM key count value
    移除列表元素
12. LSET key index value
    通过索引设置列表元素的值
13. LTRIM key start stop
    对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
14. RPOP key
    移除列表的最后一个元素，返回值为移除的元素。
15. RPOPLPUSH source destination
    移除列表的最后一个元素，并将该元素添加到另一个列表并返回
16. RPUSH key value1 [value2]
    在列表中添加一个或多个值
17. RPUSHX key value
    为已存在的列表添加值

### Set

自动排重的列表，String 类型的无序集合，底层是一个 value 为 null 的 hash 表,所以添加、删除、查找的复杂度都是Ｏ(1)。

Set 是字典结构，字典使用哈希表实现。

1. SADD key member1 [member2]
   向集合添加一个或多个成员
2. SCARD key
   获取集合的成员数
3. SDIFF key1 [key2]
   返回第一个集合与其他集合之间的差异。
4. SDIFFSTORE destination key1 [key2]
   返回给定所有集合的差集并存储在 destination 中
5. SINTER key1 [key2]
   返回给定所有集合的交集
6. SINTERSTORE destination key1 [key2]
   返回给定所有集合的交集并存储在 destination 中
7. SISMEMBER key member
   判断 member 元素是否是集合 key 的成员
8. SMEMBERS key
   返回集合中的所有成员
9. SMOVE source destination member
   将 member 元素从 source 集合移动到 destination 集合
10. SPOP key
    移除并返回集合中的一个随机元素
11. SRANDMEMBER key [count]
    返回集合中一个或多个随机数
12. SREM key member1 [member2]
    移除集合中一个或多个成员
13. SUNION key1 [key2]
    返回所有给定集合的并集
14. SUNIONSTORE destination key1 [key2]
    所有给定集合的并集存储在 destination 集合中
15. SSCAN key cursor [MATCH pattern] [COUNT count]
    迭代集合中的元素

### Sorted Set

没有重复的有序 Set

底层数据结构：

- hashTable
- 跳跃表

1. ZADD key score1 member1 [score2 member2]
   向有序集合添加一个或多个成员，或者更新已存在成员的分数
2. ZCARD key
   获取有序集合的成员数
3. ZCOUNT key min max
   计算在有序集合中指定区间分数的成员数
4. ZINCRBY key increment member
   有序集合中对指定成员的分数加上增量 increment
5. ZINTERSTORE destination numkeys key [key ...]
   计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 destination 中
6. ZLEXCOUNT key min max
   在有序集合中计算指定字典区间内成员数量
7. ZRANGE key start stop [WITHSCORES]
   通过索引区间返回有序集合指定区间内的成员
8. ZRANGEBYLEX key min max [LIMIT offset count]
   通过字典区间返回有序集合的成员
9. ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT]
   通过分数返回有序集合指定区间内的成员
10. ZRANK key member
    返回有序集合中指定成员的索引
11. ZREM key member [member ...]
    移除有序集合中的一个或多个成员
12. ZREMRANGEBYLEX key min max
    移除有序集合中给定的字典区间的所有成员
13. ZREMRANGEBYRANK key start stop
    移除有序集合中给定的排名区间的所有成员
14. ZREMRANGEBYSCORE key min max
    移除有序集合中给定的分数区间的所有成员
15. ZREVRANGE key start stop [WITHSCORES]
    返回有序集中指定区间内的成员，通过索引，分数从高到低
16. ZREVRANGEBYSCORE key max min [WITHSCORES]
    返回有序集中指定分数区间内的成员，分数从高到低排序
17. ZREVRANK key member
    返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
18. ZSCORE key member
    返回有序集中，成员的分数值
19. ZUNIONSTORE destination numkeys key [key ...]
    计算给定的一个或多个有序集的并集，并存储在新的 key 中
20. ZSCAN key cursor [MATCH pattern] [COUNT count]
    迭代有序集合中的元素（包括元素成员和元素分值）

### Hash

Redis hash 是一个 filed 和 value 的映射表，hash 特别适合存储对象。

Hash 对应的数据结构是两种 zipList/hashTable。当 filed-value 长度较短且数量较少时，使用 zipList。

1. HDEL key field1 [field2]
   删除一个或多个哈希表字段
2. HEXISTS key field
   查看哈希表 key 中，指定的字段是否存在。
3. HGET key field
   获取存储在哈希表中指定字段的值。
4. HGETALL key
   获取在哈希表中指定 key 的所有字段和值
5. HINCRBY key field increment
   为哈希表 key 中的指定字段的整数值加上增量 increment 。
6. HINCRBYFLOAT key field increment
   为哈希表 key 中的指定字段的浮点数值加上增量 increment 。
7. HKEYS key
   获取所有哈希表中的字段
8. HLEN key
   获取哈希表中字段的数量
9. HMGET key field1 [field2]
   获取所有给定字段的值
10. HMSET key field1 value1 [field2 value2 ]
    同时将多个 field-value (域-值)对设置到哈希表 key 中。
11. HSET key field value
    将哈希表 key 中的字段 field 的值设为 value 。
12. HSETNX key field value
    只有在字段 field 不存在时，设置哈希表字段的值。
13. HVALS key
    获取哈希表中所有值。
14. HSCAN key cursor [MATCH pattern] [COUNT count]
    迭代哈希表中的键值对。

## **Redis6 新数据类型**

## 配置文件

远程登陆

注释　`bind 127.0.0.1 -::1`
更改　`protected-mode no`

设置密码

`config get requirepass`
`config set requirepass "123456"`
`auth 123456`

## 发布和订阅

一种消息通信模式，发送者 pub 发送消息，订阅者 sub 接收消息。

Redis 客户端可以订阅任意数量的频道。

客户端１：`SUBSCRIBE chan1`
客户端２：`PUBLISH chan1 arch`
