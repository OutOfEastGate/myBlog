---
title: Redis6学习笔记
date: 2022-04-24 20:58:53
tags: 数据库
categories: 学习笔记-计网/计组/数据库
cover: https://w.wallhaven.cc/full/72/wallhaven-72rd8e.jpg
swiper_index: 1
description: 一个神奇的数据库
---
# 	NoSQL数据库概述
## 什么是NoSQL数据库
NoSQL(NoSQL = Not Only SQL )，意即“不仅仅是SQL”，泛指**非关系型的数据库**。 

NoSQL 不依赖业务逻辑方式存储，而以简单的**key-value模式**存储。因此大大的增加了数据库的扩展能力。
- 	不遵循SQL标准。
- 	不支持ACID。原子性 一致性 隔离性 持续性
- 	远超于SQL的性能。
##	NoSQL适用场景 
- 	对数据高并发的读写
- 	海量数据的读写
- 	对数据高可扩展性的
##	NoSQL不适用场景
- 	需要事务支持
- 	基于sql的结构化查询存储，处理复杂的关系,需要即席查询。
## Redis的优点
- 	几乎覆盖了Memcached的绝大部分功能
- 	数据都在内存中，支持持久化，主要用作备份恢复
- 	除了支持简单的key-value模式，还支持多种数据结构的存储，比如**list、set、hash、zset**等。
- 	一般是作为缓存数据库辅助持久化的数据库

### Redis是单线程+多路IO复用技术
多路复用是指使用一个线程来检查多个文件描述符（Socket）的就绪状态，比如调用select和poll函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池）

串行   vs   多线程+锁（memcached） vs   单线程+多路IO复用(Redis)

## Redis 使用 Windows下使用

```
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set name wht
OK
127.0.0.1:6379> get name
"wht"
127.0.0.1:6379>
```

## Redis linux下使用(常用命令)

redis一共有16个数据库，默认使用第一个数据库 ，可以使用select来切换数据库

```
[root@VM-12-7-centos ~]# redis-cli
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> dbsize
(integer) 0
```

插入key值

```
127.0.0.1:6379> set name wht
OK
```

查看当前所有key值

```
127.0.0.1:6379> keys *
1) "name"
```

查看key值内容

```
127.0.0.1:6379> set name wht
OK
127.0.0.1:6379> get name
"wht"
```

删除某个key值 mov +key值+当前数据库

```
127.0.0.1:6379> move name 1
(integer) 1
```

给某个key值加上过期时间和查看剩余时间

```
127.0.0.1:6379> set name wht
OK
127.0.0.1:6379> expire name 10
(integer) 1
127.0.0.1:6379> ttl name
(integer) 7
127.0.0.1:6379> ttl name
(integer) 1
127.0.0.1:6379
```

查看当前key的类型

```
127.0.0.1:6379> keys *
1) "age"
2) "name"
127.0.0.1:6379> type age
string
127.0.0.1:6379> type name
string
```

清空当库的全部key值

```
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> keys *
(empty array)
```

清除所有数据库

```
flushall
```

查看当前key值是否存在

```
exists name
```



> **Redis是单线程**
>
> Redis是基于内存操作的，CPU并不是Redis的性能瓶颈，主要由机器内存和网络带宽决定

### 为什么单线程还这么快

Redis所示c语言写的，官方数据，100000+QPS

核心：Redis将所有数据都放在内存中，所以使用单线程去操作是效率最高的

多线程上下文切换是个耗时的操作

# Redis五大基本数据



## String字符串

给key值追加字段 append

```
127.0.0.1:6379> set name wht
OK
127.0.0.1:6379> append name wht2
(integer) 7
```

查看当前key值得长度

```
127.0.0.1:6379> get name
"whtwht2"
127.0.0.1:6379> STRLEN name
(integer) 7
```

加一减一操作

```
127.0.0.1:6379> get view
"0"
127.0.0.1:6379> incr view
(integer) 1
127.0.0.1:6379> get view 
"1"
127.0.0.1:6379> decr view 
(integer) 0
127.0.0.1:6379> get view 
"0"
```

连续增加和连续减少

```
127.0.0.1:6379> get view 
"0"
127.0.0.1:6379> INCRBY view 10
(integer) 10
127.0.0.1:6379> DECRBY view 5
(integer) 5
```

获取某个区间的字符(如果是0到-1表示获取全部字符串)

```
127.0.0.1:6379> get name
"whttttttt"
127.0.0.1:6379> GETRANGE name 0 3
"whtt"
```

**setex(set with expire)**  设置key3的值为hello 30秒后过期 

```
127.0.0.1:6379> setex key3 30 "hello"
OK
127.0.0.1:6379> ttl key3
(integer) 21
```

**setnx (set if not exist)** 不存在则设置，在分布式锁中常用

```
127.0.0.1:6379> setnx name "wht2"
(integer) 0
127.0.0.1:6379> get name
"whttttttt"#还是原来的值，因为已经存在，所以创建失败
```

**mset** 批量创建

```
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
3) "k3"
```

**mget** 批量获取

```
127.0.0.1:6379> mget k1 k2 k3
1) "v1"
2) "v2"
3) "v3"
```

**msetnx** 如果不存在则批量创建(遵循原子性原则)

```
127.0.0.1:6379> msetnx k1 v1 k2 v2 k4 v4
(integer) 0
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
3) "k3	#因为k1 和k2存在，所以k4没有创建成功
```

设置一个json字符串保存数据

```
set user1:1{name:zhangsan,age:3}

127.0.0.1:6379> get user1:1
"{name:zhangsan,age:3}"
```

user:{id}:{filed} 

```
127.0.0.1:6379> mset user2:1:name wht user2:1:age 18
OK
127.0.0.1:6379> mget user2:1:name user2:1:age
1) "wht"
2) "18"
```

**getset** 先get再set，如果不存在则创建,返回的是当前get的值

```
127.0.0.1:6379> GETSET db redis
(nil)
127.0.0.1:6379> get db
"redis"
```

> 字符串时应用场景最多的地方，不仅可以是字符串，还可以是数字

## List

所有list命令都是l开头的

**LPUSH  LRANGE**  list作为堆栈使用，先进后出

```
127.0.0.1:6379> LPUSH list one
(integer) 1
127.0.0.1:6379> LPUSH list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3

127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
```

**RPUSH LPUSH** list作为队列使用,可以从左边或者右边插入

```
127.0.0.1:6379> RPUSH list right
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "right"
127.0.0.1:6379> LPUSH list left
(integer) 5
127.0.0.1:6379> LRANGE list 0 -1
1) "left"
2) "three"
3) "two"
4) "one"
5) "right"
```

**LPOP RPOP** 出栈

```
127.0.0.1:6379> LPOP list
"left"
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "right"
```

**LINDEX** 通过索引获取元素

```
127.0.0.1:6379> LINDEX list 0
"three"
```

**LLEN**获取list长度

```
127.0.0.1:6379> LLEN list
(integer) 4
```

**LREM** 移除指定元素，可以指定数量

```
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "three"
3) "two"
4) "right"
127.0.0.1:6379> LREM list 2 three
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "right"
```

**ltrim** 截取指定list

```
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "three"
3) "two"
4) "right"
127.0.0.1:6379> ltrim list 1 2
OK
127.0.0.1:6379> LRANGE list 0 -1 #list已经被改变了
1) "three"
2) "two"
```

 **RPOPLPUSH** 移除列表最后一个元素，移入新的lsit

```
127.0.0.1:6379> RPOPLPUSH list newlist
"two"
127.0.0.1:6379> LRANGE newlist 0 -1
1) "two"
```

**LINSERT** +before 和after  在指定字段后面或者前面插入新的值

```
127.0.0.1:6379> LINSERT list before "three" new 
(integer) 2
127.0.0.1:6379> LINSERT list after three new2
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "new"
2) "three"
3) "new2"
```

> 使用场景 ：消息队列，栈

## set（集合）

set集合的命令都是以s开头的

> **集合是不能重复的**,且**无序**

插入**sadd** 和查询**smembers**

```
127.0.0.1:6379> sadd myset hello
(integer) 1
127.0.0.1:6379> sadd myset wht
(integer) 1
127.0.0.1:6379> sadd myset wht2
(integer) 1
127.0.0.1:6379> smembers myset
1) "wht"
2) "hello"
3) "wht2"
```

**sismember**判断某一个元素是不是在集合中’

```
sismember myset wht
```

**scard** 查看当前集合中有多少个元素

```
127.0.0.1:6379> SCARD myset
(integer) 3
```

**srem**(remove)从当前集合中移除某个元素

```
127.0.0.1:6379> SREM myset hello
(integer) 1
```

**srandmember**随机抽取一个集合的元素(类似抽奖)

```
127.0.0.1:6379> srandmember myset
"wht2"
127.0.0.1:6379> srandmember myset
"wht"
```

**spop** 随机删除一个集合中的元素

```
spop myset
```

**smove**将制定元素移动到另外一个集合中

```
127.0.0.1:6379> smove myset myset2  wht7 
(integer) 1
127.0.0.1:6379> smembers myset2
1) "bbb"
2) "aaa"
3) "wht7"
```

**sdiff**求差集   **sinter**求交集    **sunion**求并集

```
127.0.0.1:6379> smembers myset2
1) "bbb"
2) "aaa"
3) "wht7"
127.0.0.1:6379> smembers myset
1) "wht8"
2) "aaa"
127.0.0.1:6379> SDIFF myset myset2
1) "wht8"
127.0.0.1:6379> sinter myset myset2
1) "aaa"
127.0.0.1:6379> SUNION myset myset2
1) "bbb"
2) "wht8"
3) "aaa"
4) "wht7"
```

> 交集可以做共同好友，共同关注，可能认识的人等功能

## Hash（哈希）

Map集合，key-value集合，这时候是map集合

所有hash命令都已h开头！

**hset** 加入值 ，**hget**取值

```
127.0.0.1:6379> hset myhash field1 wht1
(integer) 1
127.0.0.1:6379> hget myhash field1
"wht1"
```

**hmset**加入多个值， **mget**获取多个值,  **hgetall**获取所有数据

```
127.0.0.1:6379> hmset myhash field1 hello1 field2 hello2
OK
127.0.0.1:6379> hmget myhash field1 field2
1) "wht1"
2) "hello2"
127.0.0.1:6379> hgetall myhash
1) "field1"
2) "hello1"
5) "field2"
6) "hello2"
```

> 设置值若有重复值则会覆盖原来的值

**hdel**删除指定的key字段

```
hdel myhash filed1
```

**hlen**获取hash表的字段长度

```
127.0.0.1:6379> hlen myhash
(integer) 2
```

**exists**当前键是否存在

```
127.0.0.1:6379> exists myhash field1
(integer) 1
```

**hinceby**给指定值增加指定值

```
127.0.0.1:6379> hset myhash field3 5
(integer) 1
127.0.0.1:6379> hincrby myhash field3 2
(integer) 7
```

**hsetnx** 如果不存在则创建，否则创建失败

设置键值对的方式

```
127.0.0.1:6379> hset user:1 name wht
(integer) 1
127.0.0.1:6379> hget user:1 name
"wht"
```

## Zset有序集合

**zadd**添加 ，**zrange** 查询

```
127.0.0.1:6379> zadd myset 1 one
(integer) 1
127.0.0.1:6379> zadd myset 2 two 3 three
(integer) 2
127.0.0.1:6379> zrange myset 0 -1
1) "one"
2) "two"
3) "three"
```

**zrangebyscore**按照score从小到大排序

```redis
127.0.0.1:6379> zrangebyscore myset -inf +inf withscores
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"
```

**ZREVRANGEBYSCORE**按照score从大到小排序

```
127.0.0.1:6379> ZREVRANGEBYSCORE myset +inf -inf
1) "three"
2) "two"
```

**zrevrange**按照索引从大到小排序 反之rerange是按照索引从小到大排序

```
127.0.0.1:6379> zrevrange myset 0 -1
1) "three"
2) "two"
```

**zrem**移除某个元素

```
zrem myset one
```

**zcard**获取有序集合中的个数

 **zcount**获取某个区间的个数

```
127.0.0.1:6379> ZCOUNT myset 2 3
(integer) 2
```

> 应用场景：排行榜，班级排序

# 三种特殊数据类型

## geospatial地理位置

在redis3.2中支持推出地理位置功能，根据经度和纬度

**geoadd**添加地理位置

```
GEOADD china:city 116.40 39.90 beijing
GEOADD china:city 121.47 31.23 shanghai
```

**geopos**获取地理位置

```
127.0.0.1:6379> geopos china:city beijing shanghai
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
2) 1) "121.47000163793563843"
   2) "31.22999903975783553"
```

**geodist**获取两个位置的距离，单位默认是米，可以在最后指定单位

```
127.0.0.1:6379> geodist china:city beijing shanghai
"1067378.7564"
127.0.0.1:6379> geodist china:city beijing shanghai km
"1067.3788"
```

**georadius**以给定值为半径，在圆圈内寻找

```
127.0.0.1:6379> georadius china:city 110 40 1000 km
1) "beijing"
```

> 功能：附近的人

**georadiusbymember**以城市为圆心

```
127.0.0.1:6379> georadiusbymember china:city beijing 1000 km
1) "beijing"
```

##  Hyperloglog基数

基数就是不重复元素的个数，比如网站的访问人数，一个人访问一个网站多次是记作一次的

会有一定的错误率但在可接受范围内

**pfadd**增加 **pfcount**计算基数个数

```
127.0.0.1:6379> pfadd mykey a b c d e f
(integer) 1
127.0.0.1:6379> pfadd mykey2 a b c d e f g
(integer) 1
127.0.0.1:6379> pfcount mykey mykey2
(integer) 7
```

> 如果允许容错，则可以使用hyperloglog，否则应该使用集合

## Bitmaps

> 位存储 占用内存较少

> 用途：统计用户活跃度，打卡

**setbit** 设置状态 **bitcount**查询1的个数

```
127.0.0.1:6379> setbit sign 0 1
(integer) 0
127.0.0.1:6379> setbit sign 1 0
(integer) 0
127.0.0.1:6379> setbit sign 2 1
(integer) 0
127.0.0.1:6379> bitcount sign
(integer) 2
```

# Redis事务

redis单条命令是符合原子性的

但redis的事务是没有原子性和隔离级别的

所有命令并没有直接被执行，而是发起执行命令时才会被执行

redis事务

- 开启事务
- 命令入队
- 执行事务

**multi**开启事务，**exec**执行队列

```
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> get k1
QUEUED
127.0.0.1:6379(TX)> get k2
QUEUED
127.0.0.1:6379(TX)> EXEC
1) OK
2) OK
3) "v1"
4) "v2"
```

**discard**放弃事务,队列里的命令并不会执行

```
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> DISCARD
OK
127.0.0.1:6379> get k3
(nil)
```

事务错误类型

> 编译型异常 ，代码或命令有问题，事务中的所有命令都不会执行
>
> 运行时出现异常，事务队列中存在语法异常，执行命令时，其他命令可以正常执行，错误命令抛出异常

# 乐观锁

## 什么是悲观锁和乐观锁

悲观锁：认为什么时候都会出问题，无论做什么都会加锁，影响性能

乐观锁：认为不会出问题，所以不会加锁，更新数据的时候去判断一下，在此期间是否有人修改过这个数据

watch可以开启监视对象

# Redis发布订阅

**subscribe**订阅wht

```
127.0.0.1:6379> subscribe wht
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "wht"
3) (integer) 1
1) "message"
2) "wht"
3) "hello redis"
```

**publish**发布订阅

```
127.0.0.1:6379> PUBLISH wht "hello redis"
(integer) 1
127.0.0.1:6379> 
```

# Redis集群

**info replication**查看集群信息

```
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_failover_state:no-failover
master_replid:8a44af6f2d0b6738eae98e2f3c5f0ef76503378c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```





