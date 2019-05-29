# 第1章 初识Redis

Redis是一种基于键值对（key-value）的NoSQL数据库，与很多键值对数据库不同的是，Redis中的值可以是由string（字符串）、hash（哈希）、list（列表）、set（集合）、zset（有序集合）、Bitmaps（位图）、HyperLogLog、GEO（地理信息定位）等多种数据结构和算法组成，因此Redis可以满足很多的应用场景，而且因为Redis会将所有数据都存放在内存中，所以它的读写性能非常惊人。

Redis还可以将内存的数据利用快照和日志的形式保存到硬盘上，这样在发生类似断电或者机器故障的时候，内存中的数据不会“丢失”。

## 1.2 Redis特性

### 1.2.1 速度快

速度快大致分为以下原因：

（1）Redis的所有数据都是存放在内存中的。

（2）Redis是用C语言实现的，执行速度相对会更快。

（3）Redis使用了单线程架构，预防了多线程可能产生的竞争问题。

### 1.2.2 基于键值对的数据结构服务器

它主要提供了5种数据结构：字符串、哈希、列表、集合、有序集合。

### 1.2.3 丰富的功能

除了5种数据结构，Redis还提供了许多额外的功能：

（1）提供了键过期功能，可以用来实现缓存。

（2）提供了发布订阅功能，可以用来实现消息系统。

（3）支持Lua脚本功能，可以利用Lua创造出新的Redis命令。

（4）提供了简单的事务功能，能在一定程度上保证事务特性。

（5）提供了流水线（Pipeline）功能，这样客户端能将一批命令一次性传到Redis，减少了网络的开销。

### 1.2.4 简单稳定

### 1.2.5 客户端语言多

### 1.2.6 持久化

通常看，将数据放在内存中是不安全的，一旦发生断电或者机器故障，重要的数据可能就会丢失，因此Redis提供了两种持久化方式：RDB和AOF，即可以用两种策略将内存的数据保存到硬盘中，这样就保证了数据的可持久性。

### 1.2.7 主从复制

Redis提供了复制功能，实现了多个相同数据的Redis副本，复制功能是分布式Redis的基础。

![images\Redis主从复制架构.png](images\Redis主从复制架构.png)

### 1.2.8 高可用和分布式

Redis从2.8版本正式提供了高可用实现Redis Sentinel，它能够保证Redis节点的故障发现和故障自动转移。Redis从3.0版本正式提供了分布式实现Redis Cluster，它是Redis真正的分布式实现，提供了高可用、读写和容量的扩展性。

## 1.3 Redis使用场景

### 1.3.1 Redis可以做什么

#### 1.3.1.1 缓存

合理地使用缓存不仅可以加快数据的访问速度，而且能够有效地降低后端数据源的压力。Redis提供了键值过期时间设置，并且也提供了灵活控制最大内存和内存溢出后的淘汰策略。

#### 1.3.1.2 排行榜系统

#### 1.3.1.3 计数器应用

#### 1.3.1.4 社交网络

#### 1.3.1.5 消息队列系统

### 1.3.2 Redis不可以做什么

Redis的数据是存放在内存中的，对于大数据量的数据，就不适合存放在Redis中。站在数据冷热的角度看，数据分为热数据和冷数据，热数据通常是指需要频繁操作的数据，反之为冷数据。冷数据存放到Redis中基本上是对于内存的一种浪费，但是对于一些热数据可以放在Redis中加速读写，也可以减轻后端存储的负载，可以说是事半功倍。

## 1.5 安装和启动Redis

### 1.5.1 安装Redis

```shell
$ wget http://download.redis.io/releases/redis-3.0.7.tar.gz
$ tar xzf redis-3.0.7.tar.gz
$ ln -s redis-3.0.7 redis
$ cd redis
$ make
$ make install
```

1. 下载Redis指定版本的源码压缩包到当前目录。
2. 解压缩Redis源码压缩包。
3. 建立一个redis目录的软连接，指向redis（有利于Redis未来版本升级）。
4. 进入redis目录。
5. 编译（编译之前确保操作系统已经安装gcc）。
6. 安装（安装是将Redis的相关运行文件放到/usr/local/bin/下，这样就可以在任意目录下执行Redis的命令）。

### 1.5.2 配置、启动、操作、关闭Redis

Redis安装之后，src和/usr/local/bin目录下多了几个以redis开头可执行文件，我们称之为Redis Shell，这些可执行文件可以做很多事情，例如可以启动和停止Redis、可以检测和修复Redis的持久化文件，还可以检测Redis的性能。

![images\Redis可执行文件说明.png](images\Redis可执行文件说明.png)

#### 1.5.2.1 启动Redis

有三种方法启动Redis：默认配置、运行配置、配置文件启动：

（1）默认配置：

​	这种方法会使用Redis的默认配置来启动

```shell
redis-server
```

注：Redis建议要使用配置文件来启动。

（2）运行启动：

redis-server加上要修改配置名和值（可以是多对），没有设置的配置将使用默认配置：

```shell
redis-server --configKey1 configValue1 --configKey2 configValue2
```

例如，如果要用6380作为端口启动Redis，那么可以执行：

```shell
redis-server --port 6380
```

虽然运行配置可以自定义配置，但是如果需要修改的配置较多或者希望将配置保存到文件中，不建议使用这种方式。

（3）配置文件启动：

将配置写到指定文件里，例如我们将配置写到了/opt/redis/redis.conf中，那么只需要执行如下命令即可启动Redis：

```shell
redis-server /opt/redis/redis.conf
```

#### 1.5.2.2 Redis命令行客户端

（1）第一种交互方式：

通过redis-cli-h{host}-p{port}的方式连接到Redis服务，之后所有的操作都是通过交互的方式实现，不需要再执行redis-cli：

```shell
redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
```

（2）第二种交互方式：

用redis-cli-h ip{host}-p{port}{command}就可以直接得到命令的返回结果：

```shell
redis-cli -h 127.0.0.1 -p 6379 get hello
"world"
```

注：如果没有-h参数，那么默认连接127.0.0.1；如果没有-p，那么默认6379端口，也就是说如果-h和-p都没写就是连接127.0.0.1：6379这个Redis实例。

#### 1.5.2.3 停止Redis服务

Redis提供了shutdown命令来停止Redis服务：

```shell
redis-cli shutdown
```

注：

（1）Redis关闭的过程：断开与客户端的连接、持久化文件生成，是一种相对优雅的关闭方式。

（2）除了可以通过shutdown命令关闭Redis服务以外，还可以通过kill进程号的方式关闭掉Redis，但是不要粗暴地使用kill-9强制杀死Redis服务，不但不会做持久化操作，还会造成缓冲区等资源不能被优雅关闭，极端情况会造成AOF和复制丢失数据的情况。

（3）shutdown还有一个参数，代表是否在关闭Redis前，生成持久化文件：

```shell
redis-cli shutdown nosave|save
```



# 第2章 API的理解和使用











