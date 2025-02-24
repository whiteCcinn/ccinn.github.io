---
title: 【Redis】- 配置管理
date: 2018-02-23 15:28:00
categories: [Redis]
tags: [Redis]
---

# 说明

redis 的常用知识就不详细说了。记录一下我认为值得记录的知识点

## 基本常用的配置参数、命令

1. **daemonize no**
   配置 redis 是否以守护进程的方式运行，默认 no
   pidfile /var/run/redis.pid

2. Redis 以守护进程方式运行时，pid 的写入文件。默认为/var/run/redis.pid

3. port 6379
   监听端口,默认为 6379

4. bind 127.0.0.1
   绑定的主机地址

5. timeout 300
   客户端限制多长时间后关闭连接

6. loglevel verbose
   日志级别，支持 debug/verbos/notice/warning，默认为 verbose
   <!-- more -->
7. databases 16
   数据库的数量，默认为 0.可以使用 `SELECT <dbid>`命令在连接上指定数据库 id

8. save
   指定在多长时间内，有多少次更新操作，就将数据同步到数据文件。（Redis 默认的持久化方式：RDB 方式）
   redis 默认配置文件中配置了三个条件:

```
save 900 1
save 300 10
save 60 10000
```

表示 900 秒内有 1 个更改、300 秒内有 10 个更改以及 60 秒内有 10000 个更改。
可以存在多个条件，条件之间是“或”的关系，只要满足其中一个条件，就会进行快照。 如果想要禁用自动快照，只需要将所有的 save 参数删除即可。

10. rdbcompression yes
    指定存储至本地数据库时是否压缩数据，默认 yes。
    redis 采用 LZF 压缩，若需节省 CPU 时间可以关闭，但会导致数据库文件变大。

11. dbfilenam dump.rdb
    制定本地数据库文件名 默认为 dump.rdb

12. dir ./
    制定本地数据库存放目录

13. slaveof
    设置当本机为 slave 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动从 master 进行数据同步

14. masterauth
    当 master 服务设置了密码保护时，slave 服务连接 master 的密码

15. requirepass boobared
    设置 redis 连接密码，如果配置了连接密码，客户端在连接 redis 时需要通过`AUTH <password>`命令提供密码，默认关闭

16. maxclients 128
    最大连接数，默认无限制。0 表示不限制
    超过限制时，redis 会遍历新的连接并向客户端返回 max number of clients reached

17. maxmemory
    最大内存限制
    达到最大内存后，Redis 会先尝试清除已到期活即将到期的 key。当此方法处理后，仍然到达最大内存设置，将无法再进行写入操作，仍可以进行读取操作。Redis 新的 vm 机制会把 key 存放在内存，value 存放在 swap 区

18. appendonly no
    是否每次更新操作后进行日志记录(持久化：AOF)
    redis 默认情况下异步把数据写入磁盘，如果不开启，可能会在断电时丢失一部分数据。

19. appendfilename appendonly.aof
    更新日志文件名

20. appendfsync everysec
    日志更新条件，可选

- no：等操作系统进行数据缓存同步到磁盘（30s 一次） - 快
- always：每次更新操作后都会系统调用 fsync()将数据写 到磁盘 - 慢，安全
- everysec ：每秒一次 - 折中，默认

20. auto-aof-rewrite-percentage 100
    当目前的 AOF 文件大小超过上一次重写时的 AOF 文件大小的百分之多少时会再次进行重写，如果之前没有重写过，则以启动时的 AOF 文件大小为依据

21. auto-aof-rewrite-min-size 64mb
    允许重写的最小 AOF 文件大小

22. vm-enabled no
    指定是否启用虚拟内存机制，默认 no

23. vm-swap-file /tmp/redis.swap
    虚拟内存文件路径，默认为/temp/redis.swap，不可多个 Redis 实例共享

24. vm-max-memory 0
    将所有大于该值的数据存入虚拟内存，无论 vm-max-memory 设置多小，所有索引数据都是内存存储的（keys）。当 vm-max-memory 设置为 0 时，所有 value 都存储于磁盘中

25. vm-page-size 32
    redis swap 文件分成了很多的 page，一个对象可以保存在多个 page 上，但一个 page 上不能被多个对象共享，vm-page-size 要根据存储的数据大小来设定，如果存储很多小对象，page 大小最好设置为 32 或 64bytes；如果存储大对象，则可使用更大的 page 如果不确定，就使用默认值

26. vm-pages 134217728
    设置 swap 文件中的 page 数量，由于页表是存放在内存中的，在磁盘上没 9 个 pages 将消耗 1byte 的内存

27. vm-maxthreads 4
    swap 文件的线程数，最好不要超过机器的核心数，如果设置为 0，那么所有对 swap 文件的操作都是串行的，可能会造成比较长时间的延迟。默认为 4

28. glueoutputbuf yes
    设置在向客户端应答时，是否把较小的包合并为一个包发送，默认开启

29. hash-max-zipmap-entries 64
    hash-max-zipmap-value 512
    在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法

30. include /path/to/local.conf
    指定包含其他的配置文件，可以在同一主机上多个 Redis 实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件

## 五种基本数据类型

#### 1.String 类型

- 一个 key，一个 value

- 二进制安全

- 一个 key 最大存储 512mb

- 基本命令：set、get

- 应用场景：String 是最常用的一种数据类型，普通的 key/ value 存储都可以归为此类.即可以完全实现目前 Memcached 的功能，并且效率更高。还可以享受 Redis 的定时持久化，操作日志及 Replication 等功能。除了提供与 Memcached 一样的 get、set、incr、decr 等.

普通缓存的用处。

#### 2.List 类型

- 简单的字符串列表，类似于队列，但是是双向的，因为是双链表

- 有序

- 不去重

- 基本命令：lpush、lpop、rpush、rpop、lrange

- 应用场景：Redis list 的应用场景非常多，也是 Redis 最重要的数据结构之一，比如 twitter 的关注列表，粉丝列表等都可以用 Redis 的 list 结构来实现。Lists 就是链表，相信略有数据结构知识的人都应该能理解其结构。使用 Lists 结构，我们可以轻松地实现最新消息排行等功能。Lists 的另一个应用就是消息队列，可以利用 Lists 的 PUSH 操作，将任务存在 Lists 中，然后工作线程再用 POP 操作将任务取出进行执行。Redis 还提供了操作 Lists 中某一段的 api，你可以直接查询，删除 Lists 中某一段的元素。

#### 3.Set 类型

- 无序

- 去重

- 哈希表实现，添加，删除，查询的复杂度都是 O(1)

- 基本命令：sadd、smembers

- 应用场景：Redis set 对外提供的功能与 list 类似是一个列表的功能，特殊之处在于 set 是可以自动排重的，当你需要存储一个列表数据，又不希望出现重复数据时，set 是一个很好的选择，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。Sets 集合的概念就是一堆不重复值的组合。利用 Redis 提供的 Sets 数据结构，可以存储一些集合性的数据，比如在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis 还为集合提供了求交集、并集、差集等操作，可以非常方便的实现如共同关注、共同喜好、二度好友等功能，对上面的所有集合操作，你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。

#### 4.Zset 类型

- 根据 Score 从小到大排序（有序）

- 去重（但是会更新 Score）

- 基本命令：zadd、zrangebyscore

- 使用场景：Redis sorted set 的使用场景与 set 类似，区别是 set 不是自动有序的，而 sorted set 可以通过用户额外提供一个优先级(score)的参数来为成员排序，并且是插入有序的，即自动排序。当你需要一个有序的并且不重复的集合列表，那么可以选择 sorted set 数据结构，比如 twitter 的 public timeline 可以以发表时间作为 score 来存储，这样获取时就是自动按时间排好序的。另外还可以用 Sorted Sets 来做带权重的队列，比如普通消息的 score 为 1，重要消息的 score 为 2，然后工作线程可以选择按 score 的倒序来获取工作任务。让重要的任务优先执行。

#### 5.Hash 类型

- 类似于 nosql，一个 key 里面可以有多个 field 和 value 的映射

- 基本命令：hmset、hgetall

- 应用场景：在 Memcached 中，我们经常将一些结构化的信息打包成 HashMap，在客户端序列化后存储为一个字符串的值，比如用户的昵称、年龄、性别、积分等，这时候在需要修改其中某一项时，通常需要将所有值取出反序列化后，修改某一项的值，再序列化存储回去。这样不仅增大了开销，也不适用于一些可能并发操作的场合（比如两个并发的操作都需要修改积分）。而 Redis 的 Hash 结构可以使你像在数据库中 Update 一个属性一样只修改某一项属性值。

## redis 的持久化

- RDB 快照，全量备份

- AOF append only file 增量备份

Redis 允许同时开启 AOF 和 RDB，既保证了数据安全又使得进行备份等操作十分容易。此时重新启动 Redis 后 Redis 会使用 AOF 文件来恢复数据，因为 AOF 方式的持久化可能丢失的数据更少

1. save 和 bgsave 的区别。

- save 会阻塞主进程

- bgsave 会 fork 一个子进程进行快照备份，不会阻塞主进程

aof 文件备份与 dump 文件备份不同。dump 文件的编码格式和存储格式与数据库一致，而且 dump 文件中备份的是数据库的当前快照，意思就是，不管数据之前什么样，只要 BGSAVE 了，dump 文件就会刷新成当前数据库数据。

当 redis 重启时，会按照以下优先级进行启动：

如果只配置 AOF,重启时加载 AOF 文件恢复数据；
如果同时 配置了 RBD 和 AOF,启动是只加载 AOF 文件恢复数据;
如果只配置 RBD,启动时将加载 dump 文件恢复数据。

注意：只要配置了 aof，但是没有 aof 文件，这个时候启动的数据库会是空的

## redis 集群

```conf
port 7001 #修改端口号，从7001到7006

cluster-enabled yes #开启cluster，去掉注释
cluster-config-file nodes.conf
cluster-node-timeout 15000
appendonly yes
```

复制六份，修改对应的端口号

启动 6 个服务，分别加载不同的配置文件，即可实现 redis 集群。

客户端连接集群 redis-cli 需要带上 -c
