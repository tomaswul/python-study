---
title: redis学习总结
date: 2018-10-20 10:06:39
tags:
---

### 1.redis介绍

Redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。REmoteredis Dictionary Server(Redis)远程字典服务器，为网站提供高速缓存服务
[详细介绍](https://baike.baidu.com/item/Redis/6549233?fr=aladdin)

### 2.redis的安装和启动过程
#### 2.1 redis安装
redis的安装选择的源代码构建安装,先获取redis的源代码包
```
wget http://download.redis.io/releases/redis-4.0.11.tar.gz
```

减压压缩包

```
tar -zxvf redis-4.0.11.tar.gz
```

进入解压文件

```
cd redis-4.0.11
```

安装和导入

```
make & make install
```

#### 2.2 redis启动

启动前的配置,redis-server服务器启动前必须要进行配置，如果不进行配置将可能导致攻击者通过上传密钥配置直接获取系统的root权限

在启动服务器之后可以使用redis-benchmark -h ip -a 密码	测试redis服务器性能

1.将redis-4.0.11文件下的redis.conf文件复制到主目录下

```
cp ./redis-4.0.11/redis.conf redis.conf
```

2.使用vim对主目录下的redis.conf进行配置，主要修改一下的部分

```
vim redis.conf

修改的部分:
bind 172.18.28.55    绑定ip地址(这儿绑定的是内网的地址)

port 6666			绑定指定的端口(默认是6379)

requirepass ******	设置redis服务器的地址

appendonly yes		开启aof存储数据

appendfilename "appendonly.aof" 默认该文件在redis服务器启动的文件下文件名为appendonly.aof
```

3.开启redis服务器

```
redis-server redis.conf > redis.log &
```

4.使用redis客户端连接上服务器

```
redis-cli -h 172.18.28.55 -p 6666

auth <password>
```

### 3.常用的redis的数据类型

redis具体的操作指令，和数据类型介绍可以去[帮助](http://redisdoc.com/)

#### 3.1字符串类型

setnx:如果不存在才赋值
setex:在设置键值对的时候同时设置存活时间
mset:一次放很多键值对
mget:一个获取多个键值对

#### 3.2 哈希表(hash)

hset:设置hash类型
hget:获取值 hget key filed
hgetall:获取对应key的所有值（hgetall key）
hmget:一次性获取多个值
hmset:一次性赋值多个

	hmset stu1 id 101 name baiyuan age 12 gender male
hdel:删除哈希数据
	hdel stu1 age
hexists:判断对应键是否存在某字典
	hexists stu1 mile
hlen:统计键有多少字段
hkey:取出对应键的所有字段
hvals:取出对应键的所有值
hscan:遍历键值对

#### 3.3 列表(List)

lpush:向列表放原始(从左边开始放)

	lpush list1 1 2 3 4 5

lpop:从左边开始取
rpop:从右边开始取
rpush:(从右边开始放)
lrange:指定范围取元素()
	lrange list1 start end
	lrange list1 0 -1
lset:修改列表指定下标的值
	lset list 1 1000
blpop:如果列表没东西，且时间未超时就阻塞，有东西拿走，超时就结束（从左边取）
	blpop list1 20
brpop：如果列表没东西，且时间未超时就阻塞，有东西拿走，超时就结束（从右边取）
brpoplpush:从右边取一个元素，并把这个元素放到另一个列表的左边（阻塞式）

#### 3.4 集合(Set)
sadd:向集合添加元素
	sadd set1 10 20 10 20 30
smebers:查看集合中的元素	
	smerbers set1
sinter:交集
	sinter 集合1 集合2
sunion:并集
	sunion 集合1 集合2
sdiff:差集
	sdiff 集合1 集合2
sismenber:判断元素在不在集合中
	sismenber 集合 元素
spop:从集合中取出一个元素
srandmenber:从集合中随机返回一个元素（实际没有拿走）
srem:移除集合中的一个或者多个元素，如果不存在就忽略

#### 3.5有序集合(zsort)
zadd:添加有序集合
	zadd 集合名 值 元素
zrange:查看元素
	zrange zs1 0 -1
	zrange zs1 0 -1 withscores 显示元素的时候把分数值也显示出来
zrangebyscore:指定搜索范围来搜索数据
	zrangebyscore zs1 10 20
zrank:查看元素的排名
	zrank zs1 apple
zreverange:从大到小排序查询
	zreverange zs1
#### 3.6其他常用操作

事务:
mult:开始事务
	要执行的操作
exec:执行 ( 实际执行)
discard：放弃执行

服务器:
bgsave:后台保存
dbsize: 查看数据库有多少键
slaveof:把redis设置成那个的奴隶（主从复制，读写分离）
shutdown:关闭服务器
info:查看服务器相关信息

type(值)：查看对应值的类型

redis-check-aof -fix appendonly.aof	修复aof的文件

### 4.主从复制及哨兵
#### 4.1主从复制

主从复制一般来说是1组n从的结构，其中主人负责进行写数据，而奴隶复制读数据，主从复制的配置中，主人不需要进行任何设置，只需要配置奴隶的配置文件即可。

本次的设置主人的信息如下:

```
ip 172.18.28.55
port 6666
password 123456
```

需要设置三个奴隶: 主从复制中的奴隶的配置需要在自己服务器上的redis.conf进行配置，因此复制了redis,conf文件分别为redis01.conf,redis02.conf,redis03.conf

redis01.conf的配置:

```
 bind 172.18.28.55		奴隶服务器1的主机ip
 port 6667 				奴隶服务1的端口
 slaveof 172.18.28.55 6666   第一个ip是主人的ip，6666表示主人的端口
 masterauth 123456	这是设置主人的密码
 
```

redis02.conf的配置

```
 bind 172.18.28.55		奴隶服务器2的主机ip
 port 6668 				奴隶服务2的端口
 slaveof 172.18.28.55 6666   第一个ip是主人的ip，6666表示主人的端口
 masterauth 123456	这是设置主人的密码
```

redis03.conf的配置

```
 bind 172.18.28.55		奴隶服务器3的主机ip
 port 6669 				奴隶服务3的端口
 slaveof 172.18.28.55 6666   第一个ip是主人的ip，6666表示主人的端口
 masterauth 123456	这是设置主人的密码
```

开启redis(master)服务器:

```
redis-server redis.conf > redis.log &
```

开启三个redis(salve)服务器:

```
redis-server redis01.conf > redis01.log &
redis-server redis02.conf > redis02.log &
redis-server redis03.conf > redis03.log &
```

测试主人服务器和奴隶服务器

主人服务器:

```
[root@izwz99cyop0b1vo0ot11d5z ~]# !1019
redis-cli -h 172.18.28.55 -p 6666
172.18.28.55:6666> auth 123456
OK
172.18.28.55:6666> info replication
# Replication
role:master
connected_slaves:3
slave0:ip=172.18.28.55,port=6667,state=online,offset=742,lag=0
slave1:ip=172.18.28.55,port=6668,state=online,offset=742,lag=1
slave2:ip=172.18.28.55,port=6669,state=online,offset=742,lag=0
master_replid:c65891b7f68a6fcfca0479adc617eb09443f5d63
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:742
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:742
```

奴隶服务器:

```
[root@izwz99cyop0b1vo0ot11d5z ~]# redis-cli -h 172.18.28.55 -p 6667
172.18.28.55:6667> auth 123456
OK
172.18.28.55:6667> info replication
# Replication
role:slave
master_host:172.18.28.55
master_port:6666
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:924
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:c65891b7f68a6fcfca0479adc617eb09443f5d63
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:924
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:924
```

#### 4.2哨兵服务器的配置

哨兵服务器是在主从复制中起到监视主人状态的服务器，一旦主服务器宕机了，那么哨兵服务器就会从从服务器(奴隶)中选出新的主人，这样就可以避免因为主人死掉而使业务中断。

哨兵服务器的的设置需要从解压缩目录中拷贝一份sentinel.conf,配置这个文件中的部分内容，从而设置成哨兵服务器

哨兵服务器sentinel.conf的配置如下:

```
 bind 172.18.28.55	绑定哨兵服务器的ip
 port 26379	绑定哨兵服务器的端口(默认是26379)
 sentinel monitor mymaster 172.18.28.55 6666  设置主人服务器和端口号
 sentinel down-after-milliseconds mymaster 10000 设置主人服务器挂掉时间多少后(单位毫秒)服务器，开始选主人
 sentinel failover-timeout mymaster 180000 设置故障恢复时间，在3分钟之内原来的主服务器连上现在的集群就，原来的主人将作为奴隶，3分钟之后没有连上那么原来的主服务器将不能再连接这个集群

 
 sentinel parallel-syncs mymaster 2 设置主人挂掉之后，新选出的主人有几个奴隶需要进行业务转移
 
 entinel auth-pass mymaster 123456 设置主人的密码
```

启动哨兵服务器:

```
redis-server sentinel.conf --sentinel
```

哨兵服务器是通过心跳事件来判断主人死掉还是活着的


### 5.补充知识
网站优化的两大定律:
	1.缓存 - 用空间换取时间 - Redis/Memecached
	2.削峰 - 能推迟的事情都不要马上做 - 消息队列RabbitMQ/RocketMQ
设置指定的端口和密码来启动redis服务器
```
redis-server --port 1234 --requirepass 123456 --appendonly yes > redis.log &
```
redis的数据存储场景:
热(点)数据 - 经常被访问的数据


redis放的应该是体量不是特别大的热点数据
