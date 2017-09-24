---
layout: post
title: Redis & Codis学习笔记
categories: [database]
---

[TOC]

## 前言

 最近工作要用到Codis做数据缓存。之前虽然也一直在用着，但都是粗粒度的存取使用对于Codis和Redis都没有系统去学习。这次乘此机会系统学习一番。为了让学习效果更好，边学习边做了一些笔记记录。



## Redis

Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。

Redis 与其他 key - value 缓存产品有以下三个特点：

- **Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。**
- **Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。因此Redis也被称作数据结构服务器。**
- **Redis支持数据的备份，即master-slave模式的数据备份。**



### 优势

1. 读写性能极高
2. 丰富的数据类型 —— String/Lists/Hashes/Sets/Ordered Sets等
3. 原子操作 —— 所有操作都是原子性的。同时Redis还支持对几个操作全并后的原子性执行。
4. 丰富的特性 —— 支持publish/subscribe,通知,key 过期等等特性。

需要主要的是，Redis运行在内存中，但是可以持久化到磁盘。对于不同数据集进行高速读写时候要注意权衡内存。数据量不可以大于硬件内存。



### 键命令

Redis 键命令用于管理 redis 的键。

``` bash
redis 127.0.0.1:6379> COMMAND KEY_NAME

# 删除键
127.0.0.1:6379> DEL MYHASH2
(integer) 1

# 序列化键 （存储对象）
127.0.0.1:6379> SET name foo
OK
127.0.0.1:6379> DUMP name
"\x00\x03foo\x06\x00\xe5\xa4Q[\xb8\x94\x86\x04"
127.0.0.1:6379> DUMP UNKNOW
(nil)

# 检查KEY 是否存在
127.0.0.1:6379> EXISTS name
(integer) 1
127.0.0.1:6379> EXISTS name_2
(integer) 0

# 为键设置过期时间（秒）
127.0.0.1:6379> EXPIRE name 2
(integer) 1
127.0.0.1:6379> EXISTS name
(integer) 0

# 为键设置过期时间时间戳（秒）
127.0.0.1:6379> EXPIREAT name 1506233631000
(integer) 0

# 设置过期时间以毫秒计算
127.0.0.1:6379> PEXPIRE name 2000
(integer) 0

# 设置过期时间时间戳（毫秒）
127.0.0.1:6379> PEXPIREAT name 1506233631000
(integer) 0

# 查找符合给定模式的key
127.0.0.1:6379> KEYS *
1) "MYHASH"
2) "name:foo"
3) "myKey"
4) "foo"
127.0.0.1:6379> KEYS my*
1) "myKey"
127.0.0.1:6379> KEYS *o
1) "name:foo"
2) "foo"

# 将KEY 移动到指定数据库
MOVE key db 

# 删除KEY 过期时间，持久化KEY
127.0.0.1:6379> PERSIST MYHASH
(integer) 0

# 查询KEY 过期时间
127.0.0.1:6379> EXPIRE foo 200 
(integer) 1
# 毫秒
127.0.0.1:6379> PTTL foo
(integer) 192008
# 秒
127.0.0.1:6379> TTL foo
(integer) 187

# 从当前数据库随机返回一个KEY
127.0.0.1:6379> RANDOMKEY
"name:foo"
127.0.0.1:6379> RANDOMKEY
"MYHASH"

# 修改KEY键名称
# 仅当foo2不存在时
127.0.0.1:6379> RENAMENX foo foo2
(integer) 1
# 强制修改（覆盖）
127.0.0.1:6379> RENAMENX foo2 MYHASH
OK

# 返回KEY存储值类型
127.0.0.1:6379> TYPE MYHASH
hash
127.0.0.1:6379> TYPE myKey
string
```



### Redis 数据库

用过MYSQL的都知道，我们通常会将自己不同的业务使用不同的数据库名字（database）做区分。我们可能多个应用是在同台MYSQL SERVER的，但是却是不同的数据库。

Redis 我们可以开着很多个实例存储着不同的数据。然后使用数据库这个概念来让不同的应用程序数据彼此分开，但又可以共同存储在我们的实例之上。

redis下，数据库是由一个整数索引标识，而不是由一个数据库名称。默认情况下，一个客户端连接到数据库0。

redis配置文件中**databses** 控制一个有多少个数据库。

可以通过命令来切换不同的数据库。

``` bash
select 1
```







### 数据类型

**1. String**

String是redis最基本的类型。

String是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。

**一个键最大能存储512MB。**

``` bash
# 设置值
127.0.0.1:6379> SET foo "zhuangjy"
OK

# 获取值
127.0.0.1:6379> GET foo
"zhuangjy"

# 获取值的子串
127.0.0.1:6379> GETRANGE foo 0 5
"zhuang"

# 为某个KEY设置一个新值同时返回旧值
127.0.0.1:6379> GETSET foo zhuangjy2
"zhuangjy"

# 对 key 所储存的字符串值，获取指定偏移量上的位(bit) 用于快速修改获取的值
127.0.0.1:6379> GETBIT foo 1

# 获取多个值
127.0.0.1:6379> MGET foo foo2
1) "zhuangjy2"
2) "zz"

# 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit) 用于快速修改
127.0.0.1:6379> SETBIT foo 2 0
(integer) 1
127.0.0.1:6379> GET foo
"Zhuangjy2"

# 设置值，同时设置过期时间（秒）
127.0.0.1:6379> SETEX foo3 5 hello
OK
127.0.0.1:6379> GET foo3
"hello"
127.0.0.1:6379> GET foo3
(nil)

# 只有在key不存在时设置key的值
127.0.0.1:6379> SETNX foo3 hello
(integer) 1
127.0.0.1:6379> SETNX foo3 hello
(integer) 0

# 覆写指定key 从指定offset开始
127.0.0.1:6379> SETRANGE foo3 3 world
(integer) 8
127.0.0.1:6379> GET foo3
"helworld"

# 返回指定KEY 字符串长度
127.0.0.1:6379> STRLEN foo2
(integer) 2

# 设置一个或多个KEY VALUE
127.0.0.1:6379> MSET foo0 foo foo1 foo1 foo2 foo2
OK

# 设置一个或多个KEY VALUE 当且仅仅当所有给定的KEY不存在
127.0.0.1:6379> MSETNX foo0 foo foo1 foo1 foo2 foo2
(integer) 0

# 设置一个KEY值且指定过期时间（以毫秒为单位）
127.0.0.1:6379> PSETEX foo2 1000 foo2
OK

# 为指定KEY对应值加一（只对数字值）
127.0.0.1:6379> SET foo 1
OK
127.0.0.1:6379> INCR foo
(integer) 2

# 为指定KEY加上指定数字
127.0.0.1:6379> INCRBY foo 10
(integer) 12

# 为指定KEY加上指定浮点增量
127.0.0.1:6379> INCRBYFLOAT foo 20.23
"32.230000000000004"

# 为指定KEY减去指定增量
127.0.0.1:6379> SET foo 10
OK
127.0.0.1:6379> DECRBY foo 10
(integer) 0

# 指定VALUE为一个字符串，为其添加APPEND操作
127.0.0.1:6379> SET foo hello
OK
127.0.0.1:6379> APPEND foo " world"
(integer) 11
127.0.0.1:6379> get foo
"hello world"
```



**2. Hash**

Redis hash 是一个键名对集合。

Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

``` bash
# HSET
# HSET key field value
# 将 hash key中的域的值设置为value.若不存在创建新的，若存在则覆盖
127.0.0.1:6379> HSET MYHASH username zhuangjy
(integer) 1

# HGET
# HGET key field
# 返回hash 指定field的值
127.0.0.1:6379> HGET MYHASH username
"zhuangjy"

# HSETNX
# HSETNX key field value
# 即HSET NOT EXIST 同HSET区别在于KEY不存在时才成功，否则不做任何操作
127.0.0.1:6379> HSETNX MYHASH username zhuangjy2
(integer) 0
127.0.0.1:6379> HGET MYHASH username
"zhuangjy"
127.0.0.1:6379> HSETNX MYHASH age 12
(integer) 1
127.0.0.1:6379> HGET MYHASH age
"12"

# HMSET
# HMSET eky field value [field value...]
# 即 Multiple HSET 将多个键值对添加到HASH中，若有重复则覆盖
127.0.0.1:6379> HMSET MYHASH username foo age 20 score 100 height 180cm
OK

# HGETALL
# HGETALL key
# 返回Hash 中指定key所有域和值
127.0.0.1:6379> HGETALL MYHASH
1) "username"
2) "foo"
3) "age"
4) "20"
5) "score"
6) "100"
7) "height"
8) "180cm"


# HMGET
# HMGET key field [field ...]
# 返回hash中一个或多个给定域的值，若该给定域不存在则返回一个nil值
127.0.0.1:6379> HMGET MYHASH username age unknow
1) "foo"
2) "20"
3) (nil)


# HDEL
# HDEL key field [filed ...]
# 删除哈希表中一个或多个指定域，若不存在则忽略
127.0.0.1:6379> HDEL MYHASH username age unknow
(integer) 2


# HLEN
# HLEN key
# 返回Hash 中key对应的field的数量
127.0.0.1:6379> HLEN MYHASH
(integer) 2


# HEXISTS
# HEXISTS key field
# 查看hash中指定field是否存在,存在返回1 不存在返回0
127.0.0.1:6379> HEXISTS MYHASH username
(integer) 0
127.0.0.1:6379> HEXISTS MYHASH score
(integer) 1


# HVALS
# HVALS key
# 获得Hash 中key对应的所有values
127.0.0.1:6379> HVALS MYHASH
1) "100"
2) "180cm"


# HINCRBY
# HINCRBY key field increment
# 为Hash的域添加上增量increment，increment可以为负数（相当于减法）
# 若Hash不存在则一个新的Hash表被创建并且执行该操作
# 若field不存在，那么在执行前，域的初始值为0
# 对一个存储字符串的域调用该方法会造成一个错误
# 本操作的值限制在64位(bit)有符号数字表示之内。
127.0.0.1:6379> HINCRBY MYHASH2 score 100
(integer) 100
127.0.0.1:6379> HGET MYHASH2 score
"100"
127.0.0.1:6379> HINCRBY MYHASH2 score -50
(integer) 50
127.0.0.1:6379> HGET MYHASH2 score
"50"
127.0.0.1:6379> HINCRBY MYHASH2 score1 -50
(integer) -50
127.0.0.1:6379> HGET MYHASH2 score1
"-50"
```



**3. List**

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

一个列表最多可以包含 232 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。

``` bash
# 将一个值或者多个值插入到列表头部
lpush foo foo1 foo2 foo3

# 将一个值插入到已存在的列表的头部
127.0.0.1:6379> lpushx foo foo4
(integer) 4
127.0.0.1:6379> lpushx foo_unexist foo4
(integer) 0
127.0.0.1:6379> lpush foo_unexist foo4
(integer) 1

# 移除列表的N个元素，如果列表没有元素会阻塞直到等待超时或发现可弹出元素位置
# BLPOP key number timeout
127.0.0.1:6379> BLPOP foo 1 10
(nil)
(10.49s)

# 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
127.0.0.1:6379> BRPOP foo 1
1) "foo"
2) "foo1"

# 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
127.0.0.1:6379> BRPOPLPUSH foo foo1 3
"foo"

# 通过索引获取列表中的元素
127.0.0.1:6379> LINDEX foo 1
"foo2"

# 获取列表长度
127.0.0.1:6379> LLEN foo
(integer) 3

# 移出并获取列表的第一个元素
127.0.0.1:6379> LPOP foo
"foo3"

# 获取指定范围的元素
127.0.0.1:6379> LRANGE foo 1 10
1) "foo1"

# 通过索引设置指定值
127.0.0.1:6379> LSET foo 1 hello
OK

# 通过对一个列表进行修剪（取子列）
127.0.0.1:6379> LRANGE foo 0 10
1) "foo2"
2) "foo4"
3) "foo3"
4) "foo2"
5) "hello"
127.0.0.1:6379> LTRIM foo 1 3
OK
127.0.0.1:6379> LRANGE foo 0 10
1) "foo4"
2) "foo3"
3) "foo2"

# 移除并且获取列表的最后一个元素
127.0.0.1:6379> RPOP foo
"foo2"

# 移除列表的最后一个元素，并将该元素添加到另一个列表并返回
127.0.0.1:6379> RPOPLPUSH foo foo2
"foo3"

# 在列表末尾添加一个或者多个值
127.0.0.1:6379> RPUSH foo foo4 foo5
(integer) 3

# 为已存在的列表添加值
127.0.0.1:6379> RPUSHX foo footest
(integer) 4

# 在指定值的前后插入元素
redis> RPUSH mylist "World"
(integer) 2
redis> LINSERT mylist BEFORE "World" "There"
(integer) 3
redis> LRANGE mylist 0 -1
1) "Hello"
2) "There"
3) "World"

# 移除指定值
# redis 127.0.0.1:6379> LREM KEY_NAME COUNT VALUE
# count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。
# count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。
# count = 0 : 移除表中所有与 VALUE 相等的值。
redis 127.0.0.1:6379> LREM mylist -2 "hello"
(integer) 2



```



**4. Set**

Redis的Set是string类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

Redis 中 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

``` bash
# SADD key member1 [member2] 
# 向集合添加一个或者多个成员
127.0.0.1:6379> SADD foo foo1
(integer) 1

# SCARD key
# 获取集合的成员数
127.0.0.1:6379> SCARD foo
(integer) 1

# SDIFF key1 [key2] 
# 返回给定的所有集合的差集
127.0.0.1:6379> SADD foo foo1 foo2
(integer) 2
127.0.0.1:6379> SADD foo1 foo2 foo3
(integer) 2
127.0.0.1:6379> SDIFF foo foo1
1) "foo1"

# SDIFFSTORE destination key1 [key2] 
# 返回给定所有集合的差集并存储在 destination 中

# SINTER key1 [key2] 
# 返回给定所有集合的交集

# SINTERSTORE destination key1 [key2] 
# 返回给定所有集合的交集并存储在 destination 中
```

更多方法详见 [Redis 集合(Set)](http://www.runoob.com/redis/redis-sets.html)



**5. sorted set**

Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

``` bash
redis 127.0.0.1:6379> ZADD runoobkey 1 redis
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 2 mongodb
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 3 mysql
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 3 mysql
(integer) 0
redis 127.0.0.1:6379> ZADD runoobkey 4 mysql
(integer) 0
redis 127.0.0.1:6379> ZRANGE runoobkey 0 10 WITHSCORES

1) "redis"
2) "1"
3) "mongodb"
4) "2"
5) "mysql"
6) "4"
```

更多方法见： [Redis 有序集合(sorted set)](http://www.runoob.com/redis/redis-sorted-sets.html)



**6. HyperLogLog**

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 **HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。**

``` bash
redis 127.0.0.1:6379> PFADD runoobkey "redis"

1) (integer) 1

redis 127.0.0.1:6379> PFADD runoobkey "mongodb"

1) (integer) 1

redis 127.0.0.1:6379> PFADD runoobkey "mysql"

1) (integer) 1

redis 127.0.0.1:6379> PFCOUNT runoobkey

(integer) 3
```

更多方法详见：[Redis HyperLogLog](http://www.runoob.com/redis/redis-hyperloglog.html)



### 发布订阅

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

Redis 客户端可以订阅任意数量的频道。

**简单的订阅&发布**

1. 订阅

```bash
redis 127.0.0.1:6379> SUBSCRIBE redisChat

Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1
```

2. 发布

``` bash
redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"

(integer) 1

redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis by runoob.com"

(integer) 1

# 订阅者的客户端会显示如下消息
1) "message"
2) "redisChat"
3) "Redis is a great caching technique"
1) "message"
2) "redisChat"
3) "Learn redis by runoob.com"
```

订阅与发布类似于我们开发常用的观察者模式。在之前学习的Zookeeper也有类似的功能。

使用这个功能我们可以指定一些redis中的Key当这些Key产生变化时产生一些相应的操作。

一般用于通知清楚缓存、实时消息系统等等。



### Redis 事务

Redis 事务可以**一次执行多个命令**， 并且带有以下两个重要的保证：

- 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
- 事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。

一个事务从开始到执行会经历以下三个阶段：

- 开始事务。
- 命令入队。
- 执行事务。



以下是一个事务的例子， 它先以 **MULTI** 开始一个事务， 然后将多个命令入队到事务中， 最后由 **EXEC** 命令触发事务， 一并执行事务中的所有命令：

``` bash
redis 127.0.0.1:6379> MULTI
OK

redis 127.0.0.1:6379> SET book-name "Mastering C++ in 21 days"
QUEUED

redis 127.0.0.1:6379> GET book-name
QUEUED

redis 127.0.0.1:6379> SADD tag "C++" "Programming" "Mastering Series"
QUEUED

redis 127.0.0.1:6379> SMEMBERS tag
QUEUED

redis 127.0.0.1:6379> EXEC
1) OK
2) "Mastering C++ in 21 days"
3) (integer) 3
4) 1) "Mastering Series"
   2) "C++"
   3) "Programming"
```

除此之外，Redis提供了方法WATCH方法，监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。



### 管理命令

**1. 连接远程redis服务器**

``` bash
redis-cli -h host -p port -a password
```

**2. 检查是否连接正常**

``` bash
127.0.0.1:6379> PING
PONG
```

**3. 验证密码**

``` bash
redis 127.0.0.1:6379> AUTH "password"
OK
```

**4. Echo**

``` bash
127.0.0.1:6379> ECHO "SSS"
"SSS"
```

除此之外Redis提供了大量的服务器操作。详见[Redis 服务器](http://www.runoob.com/redis/redis-server.html)。



### Redis的主从复制

1. redis的复制功能是支持多个数据库之间的数据同步。一类是主数据库（master）一类是从数据库（slave），主数据库可以进行读写操作，当发生写操作的时候自动将数据同步到从数据库，而从数据库一般是只读的，并接收主数据库同步过来的数据，一个主数据库可以有多个从数据库，而一个从数据库只能有一个主数据库。
2. 通过redis的复制功能可以很好的实现数据库的读写分离，提高服务器的负载能力。主数据库主要进行写操作，而从数据库负责读操作。

**总结起来，就是Redis的实现是 一写多读。 **



### Pre Sharding

Redis 使用一致性哈希做数据路由，将数据分散在不同的Redis实例中。但是这样就会有一个问题，如果我们需要扩容的话。有可能会造成原先计算出来的同样的key落在新加入的机器中。导致之前的缓存都将失效。针对这个问题使用Pre Sharding技术即可解决。

Pre Sharding 实际就是在同一台机器上部署多个Redis实例的方式，当容量不够时将多个实例拆分到不同的机器上，这样实际就达到了扩容的效果。Pre-Sharding方法是将每一个台物理机上，运行多个不同端口的Redis实例，假如有三个物理机，每个物理机运行三个Redis实例，那么我们的分片列表中实际有9个Redis实例，当我们需要扩容时，增加一台物理机来代替9个中的一个redis，有人说，这样不还是9个么，是的，但是以前服务器上面有三个redis，压力很大的，这样做，相当于单独分离出来并且将数据一起copy给新的服务器（Redis主从复制）。值得注意的是，还需要修改客户端被代替的redis的IP和端口为现在新的服务器，只要顺序不变，不会影响一致性哈希分片。

具体过程如下：

1. 在新机器上启动好对应端口的Redis实例。
2. 配置新端口为待迁移端口的从库。
3. 待复制完成，与主库完成同步后，切换所有客户端配置到新的从库的端口。
4. 配置从库为新的主库。
5. 移除老的端口实例。
6. 重复上述过程迁移好所有的端口到指定服务器上。

以上拆分流程是Redis作者提出的一个平滑迁移的过程，不过该拆分方法还是很依赖Redis本身的复制功能的，如果主库快照数据文件过大，这个复制的过程也会很久，同时会给主库带来压力。所以做这个拆分的过程最好选择为业务访问低峰时段进行。



### Hash Tag 与多机路由

有一些操作诸如MSET单机时使用是好好的，多机时可能就需要一定的技巧了。

因为单实例上的MSET是一个原子性(atomic)操作，所有给定 key 都会在同一时间内被设置，某些给定 key 被更新而另一些给定 key 没有改变的情况，不可能发生。

而集群上虽然也支持同时设置多个key，但不再是原子性操作。会存在某些给定 key 被更新而另外一些给定 key 没有改变的情况。其原因是需要设置的多个key可能分配到不同的机器上。



**解决办法——Hash Tag**

分片，就是一个hash的过程：对key做md5，sha1等hash算法，根据hash值分配到不同的机器上。

为了实现将key分到相同机器，就需要相同的hash值，即相同的key（改变hash算法也行，但不简单）。

但key相同是不现实的，因为key都有不同的用途。例如user:user1:ids保存用户的tweets ID，user:user1:tweets保存tweet的具体内容，两个key不可能同名。

仔细观察user:user1:ids和user:user1:tweets，两个key其实有相同的地方，即user1。能不能拿这一部分去计算hash呢？

这就是 [Hash Tag](https://github.com/twitter/twemproxy/blob/master/notes/recommendation.md#hash-tags) 。允许用key的部分字符串来计算hash。

**当一个key包含 {} 的时候，就不对整个key做hash，而仅对 {} 包括的字符串做hash。**

假设hash算法为sha1。对user:{user1}:ids和user:{user1}:tweets，其hash值都等同于sha1(user1)。



## Codis

### 概述

Codis 是一个分布式 Redis 解决方案, 对于上层的应用来说, 连接到 Codis Proxy 和连接原生的 Redis Server 没有显著区别 ([不支持的命令列表](https://github.com/CodisLabs/codis/blob/release3.2/doc/unsupported_cmds.md)), 上层应用可以像使用单机的 Redis 一样使用, Codis 底层会处理请求的转发, 不停机的数据迁移等工作, 所有后边的一切事情, 对于前面的客户端来说是透明的, 可以简单的认为后边连接的是一个内存无限大的 Redis 服务。

### 优点

动态扩容/缩容。增减redis实例对client完全透明、不需要重启服务，不需要业务方担心 Redis 内存爆掉的问题. 也不用担心申请太大, 造成浪费. 业务方也不需要自己维护 Redis.存储海量 Key/Value (Value <= 1M)

### 分片

Codis 采用 Pre-sharding 的技术来实现数据的分片, 默认分成 1024 个 slots (0-1023), 对于每个key来说, 通过以下公式确定所属的 Slot Id : SlotId = crc32(key) % 1024。每一个 slot 都会有一个且必须有一个特定的 server group id 来表示这个 slot 的数据由哪个 server group 来提供。数据的迁移也是以slot为单位的。

### 数据安全性

Codis 并不是一个多副本的系统 (用纯内存来做多副本还是很贵的), 如果 Codis 底下的 redis 机器没有配从, 也没开 bgsave, 如果挂了, 那么最坏情况下会丢失这部分的数据, 但是集群的数据不会全失效。如果上一种情况下配了从, 这种情况, 主挂了, 到从切上来这段时间, 客户端的部分写入会失败. 主从之前没来得及同步的小部分数据会丢失.第二种情况, 业务短时间内爆炸性增长, 内存短时间内不可预见的暴涨(就和你用数据库磁盘满了一样), Codis还没来得及扩容, 同时数据迁移的速度小于暴涨的速度, 此时会触发 Redis 的 LRU 策略, 会淘汰老的 Key. 这种情况也是无解...不过按照现在的运维经验, 我们会尽量预分配一些 buffer, 内存使用量大概 80% 的时候, 我们就会开始扩容.除此之外, 正常的数据迁移, 扩容缩容, 数据都是安全的.