---
title: redis入门
date: 2022-08-10 09:09:04
categories: 数据库
tags:
 - redis
---

# 简介

redis是一个开源的、基于内存的key-value数据库

# 安装

## 使用源代码

### 下载安装包

最新稳定版本下载地址：`https://download.redis.io/redis-stable.tar.gz `

本次使用的是`7.0.1`版本

### 解压并编译

```bash
tar -zxvf redis-stable.tar.gz
cd redis-stable
make
```

编译成功后，在`src`目录下可以找到几个redis的二进制文件，包括：

- **redis-server**:Redis服务端
- **redis-cli**:与Redis-server交互的命令行程序

如果要将这些二进制文件安装到`usr/local/bin`目录，在`redis-stable`目录下执行`make install`命令。

### 前台启动和停止

执行`redis-server`命令前台启动redis服务，按`ctrl-c`停止服务。

### 后台启动和停止

```bash
cp redis.conf /etc/redis.conf
vi /etc/redis.conf # 修改daemonize属性为yes
redis-server /etc/redis.conf # 后台启动
ps -ef | grep redis # 查看redis进程
kill -9 redis-pid # 停止redis
```

## 使用docker

```bash
docker search redis
docker run redis
```

# redis客户端

> `redis-cli`是一个终端程序，用于向redis server发送命令，并读取来自redis server的响应，有两种工作模式：
>
> - 交互式：输入redis命令并收到响应
> - 命令式：附加参数执行`redis-cli`命令，并将响应打印到标准输出

## 常用参数

- --help：命令帮助
- -h：指定主机名或ip地址
- -p：指定端口
- -a：指定密码，建议通过指定`REDISCLI_AUTH`环境变量自动为redis提供密码
- -n：指定数据库编号
- -u：uri模式，`redis://user:password@host:port/dbnum`
- --raw：使用原始输出
- -r：指定命令运行的次数，-1表示无限次运行
- -i：执行各次命令运行的间隔，默认是0
- --stat：滚动打印redis server的统计信息，可用`-i`指定打印间隔
- --bigkeys：查找较大的key
- --scan：显示key列表，类似于`keys *`
- --pattern：key的匹配模式，可与`--scan`、`--bigkeys`、`--hotkeys`联合使用，默认为`*`
- --latency：查看redis server的延迟
- --rdb：rdb文件的远程备份，`redis-cli --rdb /tmp/dump.rdb`

## 命令式

> 即直接在`redis-cli`后跟要执行的命令

```bash
redis-cli incr mycounter > /tmp/mycounter.txt
redis-cli get mycounter
cat /tmp/mycounter.txt
```

## 交互式

> 即先用`redis-cli`连接到redis server后，再执行命令

```bash
redis-cli
ping
5 incr mycounter # 执行5次incr命令
clear # 清屏
```

## 监控redis命令

```bash
# 命令式
redis-cli MONITOR
# 交互式
MONITOR
```

# 配置文件

> redis.conf为redis的配置文件，不使用配置文件启动redis（`redis-server`)时会使用内置的默认配置启动。

## 通过命令行传递参数

```bash
# 启动一个redis server，端口为6380，该server为127.0.0.1:6379的副本
redis-server --port 6380 --replicaof 127.0.0.1 6379
```

## 在服务器运行时更改redis配置

> 即时修改配置对**redis.conf 文件没有影响，**因此在下次重新启动 Redis 时将使用旧配置

```bash
CONFIG GET parameter # 获取配置
CONFIG SET parameter value # 设置配置
```

## redis.conf部分配置说明

```
bind 172.0.0.1 # 表示允许访问的ip
protected-mode no # 关闭保护模式，其他主机访问
daemonize yes # 设置redis后台启动
maxmemory 100MB # 指定redis的内存大小
maxmemory-policy # redis内存满后，key的驱逐策略
maxmemory-samples # lru、lfu、最小ttl算法的采样数量，默认为5
```

# 数据类型

>redis的key通常命名为`object-type:id`，如`user:1000`，key和value最大为512MB

## 通用key命令

```bash
keys * # 查询所有key
type k1 # 获取k1的类型
exists k1 # 查看k1是否存在
del k1 # 删除k1
expire k1 30 # 设置k1的过期时间为30s
pexpire k1 1000 # 设置k1的过期时间为1000ms
ttl k1 # 查看k1的过期时间，单位s -1表示不会过期 -2表示已过期
pttl k1 # 查看k1的过期时间，单位ms -1表示不会过期 -2表示已过期
persist k1 # 删除k1的过期时间，使之永久存在
flushall # 清空所有key
```

## String

```bash
set k1 v1 nx ex 30 # 当k1不存在时，设置key为k1，value为v1，有效期为30
get k1 # 获取k1的值
ttl k1 # 查看k1的过期时间 -1表示不会过期 -2表示已过期
incr k1 # 对k1进行原子加1操作
incrby k1 3 # 对k1进行原子加3操作
decr k1 # 对k1进行原子加1操作
decrby k1 3 # 对k1进行原子加3操作
getset k1 v2 # 设置k1的值为v2，并返回k1的旧值
MSET k1 v1 k2 v2 k3 v3 # 批量设置kv
mget k1 k2 k3 # 批量获取value
```

## List

> 底层通过链表实现

```bash
lpush l1 v1 v2 v3 v4 # 从链表左侧（头部）插入数据
rpush l1 v5 v6 v7 v8 # 从链表右侧（尾部）插入数据
lrange l1 0 -1 # 查看链表所有元素，end为-1最后一个元素，为-2表示倒数第二个元素，一次类推
lpop l1# 从左侧弹出1个元素
rpop l1 2 # 从右侧弹出2个元素
ltrim l1 0 2 # 只保留链表下表0到2的数据，其他的删除
BRPOP l1 l2 timeout # 阻塞的rpop，timeout时间内有数据时返回，无数据时一直等待
blpop l1 l2 timeout # 阻塞的lpop，timeout时间内有数据时返回，无数据时一直等待
llen l1 # 获取l1的长度
```

## Hash

```bash
HSET user:1000 username zhangsan age 20 # 设置user:1000的username为zhangsan，age为20
HGET user:1000 username # 获取user:1000的用户名
HMGET user:1000 username age # 获取user:1000的用户名和年龄
HGETALL user:1000 # 获取user：1000的所有属性
HINCRBY user:1000 age 10 # 给user:1000的年龄加10
```

## Set

> 无序且唯一的字符串集合

```bash
SADD s1 1 2 3 4 # 向s1集合中添加4个元素
SMEMBERS s1 # 查看集合s1中的元素
SISMEMBER s1 1 # 判断1是否为s1中的元素，返回1表示是，0表示不是
SINTER s1 s2 # 获取s1和s2的交集
SPOP s1 2 # 从s1中随机删除两个元素并返回
SCARD s1 # 获取s1中元素的数量
SRANDMEMBER s1 2 # 从s1中随机返回两个元素，不删除
```

## ZSet

> 有序且唯一的字符串集合，类似于Set，但ZSet中有个用户排序的属性score，通过skip list和hash table实现

```bash
# 向z1中添加zhangsan、lisi、wangwu，score分别为10、20、30
ZADD z1 10 zhangsan 20 lisi 30 wangwu 
# 获取z1的所有元素，withscores表示输出score
ZRANGE z1 0 -1 withscores
# 通过score范围获取z1中的元素
ZRANGEBYSCORE z1 10 30 withscores
# 以相反的排序输出z1中的所有元素
ZREVRANGE z1 0 -1
# 删除元素zhangsan
ZREM z1 zhangsan
# 通过score范围批量删除元素
ZREMRANGEBYSCORE z1 10 20
# 获取lisi的排名
ZRANK z1 lisi
# 以相反的排序获取lisi的排名
ZREVRANK z1 lisi
```

## Bitmap

> Bitmap不是实际的数据类型，而是在 String 类型上定义的一组面向位的操作。由于字符串是二进制安全 blob，它们的最大长度为 512 MB，因此它们适合设置最多 2^32 个不同的位，每位的值只能为0或1.

```bash
SETBIT b1 0 1 # 设置b1第0位的值为1
GETBIT b1 0 # 获取b1第0位的值
BITOP AND b3 b1 b2 # 将b1和b2按位与操作后赋给b3
BITOP OR b4 b1 b2 # 将b1和b2按位或操作后赋给b4
BITOP XOR b5 b1 b2 # 将b1和b2按位异或（相同为0，不同为1）操作后赋给b5
BITOP NOT b6 b1 # 将b1按位非操作后赋给b6
BITCOUNT b1 # 统计b1中位为1的个数
BITPOS b1 1 # 获取b1中第一个值为1的位的下标
```

## HyperLogLog

> HyperLogLog 是一种概率数据结构，常用于统计集合的**基数**（集合中元素的数量）

```bash
PFADD hll 1 2 3 4
PFCOUNT hll
```

## Geospatial

> 存放地理位置

# 键驱逐

> 即redis内存满后，key的Eviction policy，在redis.conf中的maxmemory-policy属性指定

`maxmemory-policy`可用策略：

- **noeviction（默认）**：达到内存限制时不保存新值。当数据库使用复制时，这适用于主数据库
- **allkeys-lru**：保留最近使用的密钥；删除最近最少使用 (LRU) 键
- **allkeys-lfu** : 保存常用键；删除最不常用 (LFU) 键
- **volatile-lru**：类似于lru，key必须设置expire
- **volatile-lfu**：类似于lfu，key必须设置expire
- **allkeys-random**：随机删除键为添加的新数据腾出空间
- **volatile-random**：类似于random,key必须设置expire
- **volatile-ttl**：删除最近最少使用的，设置了expire的，ttl值最小的键

# 主从复制

> Redis 如何通过复制支持高可用性和故障转移
>
> - Redis 使用异步复制
> - 一个master可以有多个replicas
> - 副本能够接受来自其他副本的连接
> - Redis 复制时master是非阻塞的

## 配置

在从机的`redis.conf`文件中修改配置：

```properties
# replicaof masterip masterport
# 旧版本的命令为slaveof
replicaof 192.168.1.1 6379
# 副本是否只读，默认是yes
replica-read-only yes 
# master的密码
masterauth 123456
```

## 命令

```bash
INFO replication # 显示与副本有关的信息
ROLE # 显示master和replica的复制状态、复制偏移量、副本列表等
```

# 哨兵

> Redis Sentinel 在不使用Redis Cluster时为 Redis 提供高可用性。
>
> Redis Sentinel 还提供其他附带任务，例如监控、通知并充当客户端的配置提供程序。
>
> 这是宏观层面的 Sentinel 能力的完整列表（即*大图*）：
>
> - **监测**：Sentinel 会不断检查您的主实例和副本实例是否按预期工作。
> - **通知**：当一个受监控的 Redis 实例出现问题时，Sentinel 可以通过 API 通知系统管理员或其他计算机序。
> - **自动故障转移**：如果一个 master 没有按预期工作，Sentinel 可以启动一个故障转移过程，其中一个副本被提升为 master，其他额外的副本被重新配置为使用新的 master，并且应用程序连接redis时，会被告知要使用新地址。
> - **配置提供者**：Sentinel 充当客户端服务发现的权威来源：客户端连接到 Sentinel 以请求负责给定服务的当前 Redis 主服务器的地址。如果发生故障转移，Sentinels 将报告新地址。

## 快速启动哨兵模式

```bash
cp ./sentinel.conf /etc/sentinel.conf # 拷贝redis目录下的sentinel配置文件
# 启动哨兵模式，同redis-sentinel /etc/sentinel.conf
redis-server /etc/sentinel.conf --sentinel 
```

## 配置哨兵

```properties
# master name为mymasterip为127.0.0.1，端口为6379,
# 2表示quorum数量，即有几个sentinel同意master故障时，该master才被标记为故障
sentinel monitor mymaster 127.0.0.1 6379 2
# sentinel经过60000ms后收不到master的响应后，认为该master已故障
sentinel down-after-milliseconds mymaster 60000
# 设置故障转移超时时间为180000ms
sentinel failover-timeout mymaster 180000
# 设置故障转移后，重新使用新master的副本数
sentinel parallel-syncs mymaster 1
```

## 案例

> 下面以搭建一主两从三哨兵的架构来演示；
>
> node：192.168.5.121 	master(6379) 	sentinel1(26379)
>
> ​				192.168.5.111 	slave(6379) 	sentinel2(26379)
>
> ​				192.168.5.112 	slave(6379) 	sentinel3(26379)

两个slave节点的`redis.conf`配置复制属性：

```properties
replicaof 192.168.5.121 6379
```

三个sentinel的`sentinel.conf`调整如下属性：

```properties
port 26379
daemonize yes # 允许后台启动
logfile "/tmp/sentinel.log"
sentinel monitor mymaster 192.168.5.121 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
```

分别启动三台redis server及三台redis sentinel

```bash
redis-server /etc/redis.conf # 启动redis server
redis-server /etc/sentinel.conf --sentinel # 启动redis sentinel
ps -ef | grep redis # 查看启动结果
```

查看复制信息及哨兵信息

```bash
redis-cli -p 6379 # 连接redis server
info replication # 查看复制信息
# 退出redis-cli  Ctrl + c
redis-cli -p 26379 # 连接redis sentinel
# 查看哨兵信息
# num-other-sentinels为2，即Sentinel已经为这个master检测到了另外两个Sentinel，日志中有`+sentinel`
# flags是master，如果master故障，flags会变成s_down或o_down
# num-slaves为2，表示有两个副本
sentinel master mymaster 
```

手动停掉master节点后，观察sentinel信息

```bash
ps -ef | grep redis
kill -9 redispid
# 等待5s，即down-after-milliseconds配置的时间
redis-cli -p 26379 
sentinel master mymaster 
sentinel get-master-addr-by-name mymaster
```

## 命令

```bash
sentinel master mymaster # 查看master信息
sentinel replicas mymaster # 查看master的副本信息
sentinel sentinels mymaster # 查看其他sentinel信息
SENTINEL GET-MASTER-ADDR-BY-NAME mymaster # 获取master地址
sentinel help # 命令帮助
```

# 持久化

> redis持久化是将数据写入磁盘存储，方式有：
>
> - **RDB**(Redis Dababase)：每隔一段时间保存一次数据库快照
> - **AOF**(Append Only File)：保存服务器收到的每个写操作，服务器重启后，重新执行这些操作，当日志很大时，Redis在后台会**重写日志**
> - **不持久化**：数据仅存放在内存中
> - **RDB + AOF**：同时开启rdb和aof，当数据库重启时，会使用aof来重新加载数据，因为rdb保存的快照可能不完整

## 配置

```properties
# RDB相关
# redis默认开启RDB
# save后的参数表示3600s内，有1个key发生变化，就生成RDB文件
# save "" 表示关闭RDB
save 3600 1 
save 300 100
save 60 10000
# RDB文件的名称
dbfilename "dump.rdb" 
# 工作目录
dir "/usr/local/redis/redis-stable" 

# AOF相关
# 开启aof持久化
appendonly yes
# 每秒同步一次aof文件
appendfsync everysec
```

## 命令

```bash
save # 手动保存RDB快照
bgsave # 手动后台保存RDB快照
BGREWRITEAOF # 后台重写aof文件
```

# 发布订阅

> 一个客户端订阅某个channel，另一个客户端向这个channel里发布数据，订阅该channel的客户端可接收到消息。

## 通道名称订阅

```bash
# 打开一个终端
redis-cli
SUBSCRIBE channel1 # 订阅channel1通道
# 重新打开一个终端
redis-cli
PUBLISH channel1 hello # 向channel1通道发送消息hello，观察订阅终端的输出
```

## 模式匹配订阅

```bash
# 打开一个终端
redis-cli
PSUBSCRIBE cha* # 订阅以cha开头的通道
# 重新打开一个终端
redis-cli
PUBLISH channel1 hello # 向channel1通道发送消息hello，观察订阅终端的输出
```

## 分片发布订阅

```bash
# 打开一个终端
redis-cli
SSUBSCRIBE channel1 # 订阅channel1通道
# 重新打开一个终端
redis-cli
SPUBLISH channel1 hello # 向channel1通道发送消息hello，观察订阅终端的输出
```

## 其他命令

```bash
UNSUBSCRIBE channel1 # 取消订阅channel1通道
UNSUBSCRIBE # 取消订阅所有通道
PUNSUBSCRIBE cha* # 取消订阅以cha开头的通道
PUNSUBSCRIBE # 取消订阅所有通道
SUNSUBSCRIBE channel1 # 取消订阅channel1通道
SUNSUBSCRIBE # 取消订阅所有通道
```

# 集群

## 数据分片

- redis cluster使用数据分片，将所有的key都映射到slot上
- redis cluster共有16384个slot
- redis cluster中可使用哈希标签`{}`，保证一些相关的key映射到同一个slot上，如`user:{123}:username和user:{123}:age`一定位于同一个slot

## 搭建集群

> 搭建一个三主三从的redis cluster，节点分配如下：
>
> 192.168.5.121	6379	 6380
>
> 192.168.5.111	6379	 6380	
>
> 192.168.5.112	6379	6380	

六个节点**redis.conf**配置

```properties
port 6379 # 端口要换
appendonly yes
protected-mode no
daemonize yes
#bind 172.0.0.1 
cluster-enabled yes # 开启集群模式
cluster-config-file nodes.conf # 指定各实例存储节点信息的文件
cluster-node-timeout 5000 # 集群节点超时时间ms
```

分别启动六个节点

```bash
redis-server /etc/redis.6379.conf
redis-server /etc/redis.6380.conf
# 这里要注意，搭建集群的节点必须是新节点，不能有数据，有数据的话需要清理掉
redis-cli
flushall
cluster reset hard
```

创建集群

```bash
# 连接任意一个节点执行
# cluster-replicas表示副本数，六个节点，一个副本，即三主三从
# redis-cli将提出一个配置。输入yes接受建议的配置
# 输出[OK] All 16384 slots covered即为成功
redis-cli --cluster create 192.168.5.121:6379 192.168.5.121:6380 \
192.168.5.111:6379 192.168.5.111:6380 192.168.5.112:6379 192.168.5.112:6380 \
--cluster-replicas 1
# 查看集群信息
cluster info
# 查看集群节点
cluster nodes
```

连接集群

```bash
# -c参数以集群的方式连接redis服务器
# 集群模式具有重定向功能，避免set key时，key的hash值所在的slot不属于当前的redis server
redis-cli -c -p 6379
```

测试集群

```bash
# 查看集群状态
redis-cli --cluster check 192.168.5.121:6379
# 手动停掉一个master节点
kill -9 redis-server-pid
# 再次查看集群状态
redis-cli --cluster check 192.168.5.121:6379
# 重启master节点
redis-server /etc/redis.6379.conf
# 再次查看集群状态
redis-cli --cluster check 192.168.5.121:6379
```

## 集群扩容

> 先在对上面搭建的三主三从集群添加两个节点，使集群扩容为四主四从，两个节点为
>
> 192.168.5.110	6379	6380

启动两个节点

```bash
redis-server /etc/redis.6379.conf
redis-server /etc/redis.6380.conf
```

添加节点

```bash
# 第一个ip：port为要添加的node，第二ip：port为现有集群中任一node
redis-cli --cluster add-node 192.168.5.110:6379	 192.168.5.121:6379
# 检查集群状态，发现新节点已加入集群，但slot为0，且没有从节点
redis-cli --cluster check 192.168.5.121:6379
```

给新节点分配slot

```bash
# ip:port为集群中任一节点
# How many slots do you want to move (from 1 to 16384)? 输入4096 16384/master个数
# What is the receiving node ID? 输入集群中新节点的id，redis-cli --cluster check 192.168.5.121:6379 查看id
# Source node #1: 输入all
# Do you want to proceed with the proposed reshard plan (yes/no)? 输入yes
redis-cli --cluster reshard 192.168.5.121:6379
# 检查集群状态，新节点已分配slot
redis-cli --cluster check 192.168.5.121:6379
```

加入从节点

```bash
# 第一个ip:port为新加入的从节点，第二ip：port为现有集群中任一节点
# cluster-master-id指定主节点的id
redis-cli --cluster add-node 192.168.5.110:6380 192.168.5.121:6379 --cluster-slave \
--cluster-master-id a53d4839a64933b96f59ebc06fbeb32d7699e618
# 检查集群状态，至此集群已为四主四从架构
redis-cli --cluster check 192.168.5.121:6379
```

## 集群缩容

> 现在再将上面扩容后的四主四从的集群缩容为三主三从，去掉192.168.5.110上的两个节点

从集群中删除从节点

```bash
# ip：port为现有集群中任一节点，最后一个参数为要删除节点的id
redis-cli --cluster del-node 192.168.5.121:6379 c745ac3f5c80628174a8f2dc0fca689539613704
# 检查集群状态，从节点已删除
redis-cli --cluster check 192.168.5.121:6379
```

重新分配主节点上的slot

```bash
# 将要删除的主节点上的slot分配给其他master
# How many slots do you want to move (from 1 to 16384)? 输入4096，即master的slot数
# What is the receiving node ID? 输入一个master的id，即把4096个slot全部给这个master
# Source node #1: 输入要删除的master的的id
# Source node #2: 输入done
# Do you want to proceed with the proposed reshard plan (yes/no)? 输入yes
redis-cli --cluster reshard 192.168.5.121:6379
# 检查集群状态，该master上已没有slot
redis-cli --cluster check 192.168.5.121:6379
```

从集群中删除主节点

```bash
# ip：port为现有集群中任一节点，最后一个参数为要删除节点的id
redis-cli --cluster del-node 192.168.5.121:6379 c745ac3f5c80628174a8f2dc0fca689539613704
# 检查集群状态，至此集群已恢复为三主三从
redis-cli --cluster check 192.168.5.121:6379
```

# 事务

> - redis事务能保证原子性
> - redis事务不会回滚

## 常用命令

- MULTI：开启一个事务
- EXEC：提交事务
- DISCARD：丢弃事务
- WATCH：监测一个key，当这个key发生变化时，事务终止

## 案例

```bash
# 在事务中设置a和b
MULTI
set a a1
set b b1
EXEC

# incr c 会报错，事务中止，但不会回滚，c和d已正常赋值
MULTI
set c c1
set d d1
incr c
set e e1
EXEC

# 丢弃一个事务
MULTI
set f f1
set g g1
DISCARD

# 监控一个key
# 新开一个终端执行set a操作，a的值发生变化，该事务中止
WATCH a
MULTI
set h h1
set l l1
EXEC
```

# Jedis使用

> Redis推出的java client，其api跟redis-cli中的命令基本一致，方便上手
>
> git地址：`https://github.com/redis/jedis`

## maven依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>4.2.0</version>
</dependency>
```

## 获取连接

```java
Jedis jedis = new Jedis(new HostAndPort("192.168.5.121", 6379), DefaultJedisClientConfig.builder().build());
```

## 多线程中使用Jedis

```java
// 多线程下使用单个jedis实例可能有莫名其妙的错误，从连接池中获取实例可用于多线程环境
// 该线程池可定义为静态变量
JedisPool jedisPool = new JedisPool(new JedisPoolConfig(), "192.168.5.121", 6379);
try (Jedis jedis = jedisPool.getResource()) {
    jedis.set("testk", "testv");
    System.out.println(jedis.get("testk"));
    jedis.zadd("z1", 10, "java");
    jedis.zadd("z1", 20, "c++");
    Set<Tuple> z1 = jedis.zrangeWithScores("z1", 0, -1);
    for (Tuple tuple : z1) {
        System.out.println(tuple.getScore() + "  " + tuple.getElement());
    }
}
jedisPool.close();
```

## 设置主从复制

```java
// 设置当前redis为192.168.5.112:6380的从机，若redis.conf中配置从机只读的话，该jedis不能执行写操作
jedis.slaveof("192.168.5.112", 6380);
// 将jedis实例升级为主机，而不是任何一个主机的从机
jedis.slaveofNoOne();
```

## 事务

```java
public void transaction() {
    jedis.watch("k1", "k2"); // watch key
    Transaction transaction = jedis.multi(); // 开启redis事务
    transaction.set("foo", "bar");
    Response<String> foo = transaction.get("foo"); // 获取返回值
    transaction.lpush("games", "lol", "cf", "dnf");
    Response<List<String>> games = transaction.lrange("games", 0, -1);
    if (transaction.get("k1").equals("v1")) { // 这种判断是错误的，不能在事务中使用事务的中查询结果
        // do some thing
    }
    transaction.exec(); // 提交事务
    // transaction.discard();

    // 打印结果，必须在exec方法后
    System.out.println(foo.get());
    System.out.println(games.get());
}
```

## Pipeline

>有时你需要发送一堆不同的命令。一个非常酷的方法是使用流水线，并且比原始的方法具有更好的性能。这样，您无需等待响应即可发送命令，并且您实际上在最后读取了响应，这样更快。

```java
public void pipeline() {
    Pipeline pipelined = jedis.pipelined();
    pipelined.set("car", "baoma");
    pipelined.zadd("language", 10, "java");
    pipelined.zadd("language", 20, "c");
    pipelined.zadd("language", 30, "c++");
    Response<String> car = pipelined.get("car");
    Response<List<Tuple>> language = pipelined.zrangeWithScores("language", 0, -1);
    pipelined.sync();
    System.out.println(car.get());
    for (Tuple tuple : language.get()) {
        System.out.println(tuple.getScore() + " - " + tuple.getElement());
    }
}
```

## 发布订阅

> 要订阅 Redis 中的频道，请创建 JedisPubSub 实例并在 Jedis 实例上调用 subscribe;
>
> 请注意，订阅是一个**阻塞**操作，因为它会在调用订阅的线程上轮询 Redis 以获取响应。单个 JedisPubSub 实例可用于订阅多个频道。您可以在现有 JedisPubSub 实例上调用 subscribe 或 psubscribe 来更改您的订阅。

```java
/**
 * redis监听器
 */
public class RedisListener extends JedisPubSub {
    @Override
    public void onMessage(String channel, String message) {
        System.out.println("onMessage: " + channel + " - " + message);
        super.onMessage(channel, message);
    }

    @Override
    public void onPMessage(String pattern, String channel, String message) {
        System.out.println("onPMessage: " + pattern + " - " + channel + " - " + message);
        super.onPMessage(pattern, channel, message);
    }

    @Override
    public void onSubscribe(String channel, int subscribedChannels) {
        System.out.println("onSubscribe: " + channel + " - " + subscribedChannels);
        super.onSubscribe(channel, subscribedChannels);
    }

    @Override
    public void onUnsubscribe(String channel, int subscribedChannels) {
        System.out.println("onUnsubscribe: " + channel + " - " + subscribedChannels);
        super.onUnsubscribe(channel, subscribedChannels);
    }

    @Override
    public void onPUnsubscribe(String pattern, int subscribedChannels) {
        System.out.println("onPUnsubscribe: " + pattern + " - " + subscribedChannels);
        super.onPUnsubscribe(pattern, subscribedChannels);
    }

    @Override
    public void onPSubscribe(String pattern, int subscribedChannels) {
        System.out.println("onPSubscribe: " + pattern + " - " + subscribedChannels);
        super.onPSubscribe(pattern, subscribedChannels);
    }
}



public void sub() {
    Jedis jedis = jedisPool.getResource();
    RedisListener redisListener = new RedisListener();
    // 阻塞当前线程，订阅mychannel通道
    jedis.subscribe(redisListener, "mychannel");
}
```

## 连接集群

```java
public void cluster() {
    HashSet<HostAndPort> clusterNodes = new HashSet<>();
    clusterNodes.add(new HostAndPort("192.168.5.121", 6379));
    clusterNodes.add(new HostAndPort("192.168.5.121", 6380));
    clusterNodes.add(new HostAndPort("192.168.5.121", 6381));
    clusterNodes.add(new HostAndPort("192.168.5.121", 6381));
    clusterNodes.add(new HostAndPort("192.168.5.121", 6383));
    clusterNodes.add(new HostAndPort("192.168.5.121", 6384));
    // JedisCluster有多个构造方法，按实际情况调不同的构造方法
    JedisCluster jedisCluster = new JedisCluster(clusterNodes);
    jedisCluster.set("k1", "v2");
}
```

# Lettuce使用

> Lettuce是一个可扩展的Redis客户端，用于构建非阻塞响应式应用程序。
>
> Lettuce 是一个基于[netty](https://netty.io/)和 Reactor 的可扩展的线程安全 Redis 客户端。Lettuce 提供[同步](https://lettuce.io/core/release/reference/index.html#basic-usage)、[异步](https://lettuce.io/core/release/reference/index.html#asynchronous-api)和[反应式](https://lettuce.io/core/release/reference/index.html#reactive-api)API 来与 Redis 交互。

## maven依赖

```xml
<dependency>
    <groupId>io.lettuce</groupId>
    <artifactId>lettuce-core</artifactId>
    <version>6.1.8.RELEASE</version>
</dependency>
```

## 简单使用

```java
// redis://password@ip:port/databasenumber
RedisClient redisClient = RedisClient.create("redis://@192.168.5.121:6379/0");
StatefulRedisConnection<String, String> connect = redisClient.connect();
RedisCommands<String, String> syncCommands = connect.sync();
syncCommands.set("lettuce", "hello");
connect.close();
redisClient.shutdown();
```

## 连接redis

使用uri：

```java
RedisURI.create("redis://localhost/");
```

使用build：

```java
RedisURI.Builder.redis("localhost", 6379).auth("password").database(1).build();
```

使用new对象：

```java
new RedisURI("localhost", 6379, 60, TimeUnit.SECONDS);
```

### uri格式

单机：

```
redis :// [[username :] password@] host [:port][/database]
          [?[timeout=timeout[d|h|m|s|ms|us|ns]]
```

哨兵：

```
redis-sentinel :// [[username :] password@] host1[:port1] [, host2[:port2]] [, hostN[:portN]] [/database]
                   [?[timeout=timeout[d|h|m|s|ms|us|ns]] [&sentinelMasterId=sentinelMasterId]
```

### 同步api

```java
// 构建redisURI
RedisURI redisURI = RedisURI.Builder
    .redis("192.168.5.121", 6379)
    .withDatabase(0)
    .withTimeout(Duration.ofSeconds(60))
    .build();
// 创建redis client实例
RedisClient redisClient = RedisClient.create("redis://@192.168.5.121:6379/0");
// 打开redis连接
StatefulRedisConnection<String, String> connect = redisClient.connect();
// 获取同步api
RedisCommands<String, String> syncCommands = connect.sync();
// 执行get命令
String lettuce = syncCommands.get("lettuce");
System.out.println(lettuce);
// 关闭redis连接
connect.close();
// 关闭redis client实例
redisClient.shutdown();
```

### 异步api

```java
// 构建redisURI
RedisURI redisURI = RedisURI.Builder
    .redis("192.168.5.121", 6379)
    .withDatabase(0)
    .withTimeout(Duration.ofSeconds(60))
    .build();
// 创建redis client实例
RedisClient redisClient = RedisClient.create(redisURI);
// 打开redis连接
StatefulRedisConnection<String, String> connect = redisClient.connect();
// 获取异步api
RedisAsyncCommands<String, String> asyncCommands = connect.async();
// 执行get命令
RedisFuture<String> lettuce = asyncCommands.get("lettuce");
try {
    // 获取RedisFuture中的值
    System.out.println(lettuce.get(1, TimeUnit.MINUTES));
} catch (Exception e) {
    e.printStackTrace();
}
// 获取RedisFuture中的值
lettuce.thenAccept(System.out::println);
// 关闭redis连接
connect.close();
// 关闭redis client实例
redisClient.shutdown();
```

### 响应式api

> 参照官网自行理解

## 发布/订阅

```java
RedisClient redisClient = RedisClient.create("redis://@192.168.5.121:6379/0");
StatefulRedisPubSubConnection<String, String> pubsubConnect 
    = redisClient.connectPubSub();
// 注意：不要在内部回调方法中使用阻塞调用的方法
pubsubConnect.addListener(new RedisPubSubListener<String, String>() {
    @Override
    public void message(String channel, String message) {
        System.out.println("信道" + channel + "中收到消息：" + message);
    }

    @Override
    public void message(String pattern, String channel, String message) {

    }

    @Override
    public void subscribed(String channel, long count) {

    }

    @Override
    public void psubscribed(String pattern, long count) {

    }

    @Override
    public void unsubscribed(String channel, long count) {

    }

    @Override
    public void punsubscribed(String pattern, long count) {

    }
});

// 同步订阅
RedisPubSubCommands<String, String> sync = pubsubConnect.sync();
sync.subscribe("channel1", "channel2");

// 异步订阅
//RedisPubSubAsyncCommands<String, String> async = pubsubConnect.async();
//RedisFuture<Void> aa = async.subscribe("channel1");
try {
    Thread.sleep(1000000);
} catch (Exception e) {
    e.printStackTrace();
}
pubsubConnect.close();
redisClient.shutdown();
```

## 事务

**同步事务**

```java
RedisClient redisClient = RedisClient.create("redis://@192.168.5.121:6379/0");
StatefulRedisConnection<String, String> connect = redisClient.connect();
RedisCommands<String, String> sync = connect.sync();
sync.watch("k1");
String multi = sync.multi();
String set = sync.set("multi", "ok");
String set1 = sync.set("test", "yes");
//sync.discard();
TransactionResult exec = sync.exec();
System.out.println(exec);
connect.close();
redisClient.shutdown();
```

---

**异步事务**

```java
RedisClient redisClient = RedisClient.create("redis://@192.168.5.121:6379/0");
StatefulRedisConnection<String, String> connect = redisClient.connect();
RedisAsyncCommands<String, String> async = connect.async();
async.watch("k1");
RedisFuture<String> multi = async.multi();
RedisFuture<String> set = async.set("k2", "v2");
RedisFuture<String> set1 = async.set("k3", "v3");
RedisFuture<TransactionResult> exec = async.exec();
try {
    System.out.println(multi.get());
    System.out.println(set.get());
    System.out.println(set1.get());
    System.out.println(exec.get());
} catch (InterruptedException e) {
    e.printStackTrace();
} catch (ExecutionException e) {
    e.printStackTrace();
}
connect.close();
redisClient.shutdown();
```

## 集群

```java
RedisURI node1 = RedisURI.create("node1", 6379);
RedisURI node2 = RedisURI.create("node2", 6379);
RedisClusterClient redisClusterClient = 
    RedisClusterClient.create(Arrays.asList(node1, node2));
StatefulRedisClusterConnection<String, String> connect = redisClusterClient.connect();
RedisAdvancedClusterCommands<String, String> sync = connect.sync();

// some operatio

connect.close();
redisClusterClient.shutdown();
```

# Spring Data Redis使用

> spring data redis为spring对redis的封装，底层依赖于jedis或lettuce

## maven依赖

```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.7.1</version>
</dependency>
```

## 连接到redis

> 主要通过`org.springframework.data.redis.connection`包下的`RedisConnection`和`RedisConnectionFactory`两个接口来连接

### 使用lettuce连接

**maven依赖**

```xml
<dependency>
    <groupId>io.lettuce</groupId>
    <artifactId>lettuce-core</artifactId>
    <version>6.1.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.7.1</version>
</dependency>
```

---

**注入RedisConnectionFactory**

```java
@Configuration
public class AppConfig {

    @Bean
    public LettuceConnectionFactory lettuceConnectionFactory() {
        // 配置从副本读取数据
        LettuceClientConfiguration clientConfig = LettuceClientConfiguration.builder()
                .readFrom(ReadFrom.REPLICA_PREFERRED)
                .build();

        RedisStandaloneConfiguration serverConfig = new RedisStandaloneConfiguration("server", 6379);
        return new LettuceConnectionFactory(serverConfig, clientConfig);
    }

}
```

### 使用jedis连接

**maven依赖**

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>4.2.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.7.1</version>
</dependency>
```

---

**注入RedisConnectionFactory**

```java
@Configuration
public class AppConfig {
	
    @Bean
    public JedisConnectionFactory jedisConnectionFactory() {
        return new JedisConnectionFactory(new RedisStandaloneConfiguration("192.168.5.121", 6379));
    }
}
```

## redis哨兵

```java
/**
* Jedis
*/
@Bean
public RedisConnectionFactory jedisConnectionFactory() {
    RedisSentinelConfiguration sentinelConfig = new RedisSentinelConfiguration()
        .master("mymaster")
        .sentinel("127.0.0.1", 26379)
        .sentinel("127.0.0.1", 26380);
    return new JedisConnectionFactory(sentinelConfig);
}

/**
* Lettuce
*/
@Bean
public RedisConnectionFactory lettuceConnectionFactory() {
    RedisSentinelConfiguration sentinelConfig = new RedisSentinelConfiguration()
        .master("mymaster")
        .sentinel("127.0.0.1", 26379)
        .sentinel("127.0.0.1", 26380);
    return new LettuceConnectionFactory(sentinelConfig);
}
```

## 使用RedisTemplate

> `org.springframework.data.redis.core`包下封装了对redis的各种操作，如`RedisTemplate`、`ValueOperations`、`BoundValueOperations`等;
>
> `XXOperations`提供对redis中某种类型的操作；
>
> `BoundXXOperations`提供绑定某个key后的操作；

```java
@Configuration
public class RedisConfig {

    @Bean
    public LettuceConnectionFactory lettuceConnectionFactory() {
        RedisStandaloneConfiguration serverConfig = new RedisStandaloneConfiguration("192.168.5.121", 6379);
        return new LettuceConnectionFactory(serverConfig);
    }

    @Bean
    public RedisTemplate redisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(lettuceConnectionFactory);
        return redisTemplate;
    }
    
    @Bean
    public StringRedisTemplate stringRedisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
        stringRedisTemplate.setConnectionFactory(lettuceConnectionFactory);
        return stringRedisTemplate;
    }

}
```

```java
@Autowired
private RedisTemplate redisTemplate;
@Resource(name = "redisTemplate")
private ListOperations<String, String> listOperations;
@Autowired
private StringRedisTemplate stringRedisTemplate;

@Test
public void t1() {
    listOperations.leftPush("l1", "java");
    stringRedisTemplate.opsForValue().set("k1", "v1");
}
```

## 序列化器

> `RedisTemplate`默认使用`JdkSerializationRedisSerializer`，这样存到redis中的数据可能为乱码，可使用以下序列化器：
>
> - `StringRedisSerializer`:处理字符串数据
> - `GenericJackson2JsonRedisSerializer`:处理json格式数据

**RedisTemplate注入**

```java
@Bean
public RedisTemplate redisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
    RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
    redisTemplate.setKeySerializer(RedisSerializer.string());
    redisTemplate.setValueSerializer(RedisSerializer.string());
    redisTemplate.setHashKeySerializer(RedisSerializer.string());
    redisTemplate.setHashValueSerializer(RedisSerializer.string());
    redisTemplate.setConnectionFactory(lettuceConnectionFactory);
    return redisTemplate;
}
```

## 发布订阅

**发布**

```java
redisTemplate.convertAndSend("channel1", "msg-");
```

---

**订阅**

```java
@Component
public class MyMsgListener implements MessageListener {
    @Override
    public void onMessage(Message message, byte[] pattern) {
        System.out.println(new String(pattern));
        System.out.println(new String(message.getChannel()));
        System.out.println(new String(message.getBody()));
    }
}
```

```java
@Bean
public RedisMessageListenerContainer redisMessageListenerContainer(LettuceConnectionFactory lettuceConnectionFactory, MyMsgListener myMsgListener) {
    RedisMessageListenerContainer container = new RedisMessageListenerContainer();
    container.setConnectionFactory(lettuceConnectionFactory);
    List<Topic> topics = new ArrayList<>();
    topics.add(new ChannelTopic("channel1"));
    topics.add(new PatternTopic("*test"));
    container.addMessageListener(myMsgListener, topics);
    return container;
}
```

## 事务

```java
redisTemplate.execute(new SessionCallback() {
    @Override
    public List<Object> execute(RedisOperations operations) throws DataAccessException {
        operations.multi();
        operations.opsForValue().set("k1", "v1");
        operations.opsForValue().increment("k1");
        operations.opsForValue().set("k2", "v2");
        return operations.exec();
    }
});
```

## 集群

**注册`RedisConnectionFactory`时使用集群配置**

```java
@Bean
public RedisConnectionFactory redisConnectionFactory() {

    List<String> clusterNodes = new ArrayList<>();
    clusterNodes.add("127.0.0.1:6379");
    clusterNodes.add("127.0.0.1:6380");
    clusterNodes.add("127.0.0.1:6381");
    clusterNodes.add("127.0.0.1:6382");
    clusterNodes.add("127.0.0.1:6383");
    clusterNodes.add("127.0.0.1:6384");
    return new LettuceConnectionFactory(new RedisClusterConfiguration(clusterNodes));
}
```

# spring boot集成

> spring boot集成了spring data redis，默认使用lettuce，并自动装配了`RedisConnectionFactory`和`RedisTemplate`，配置类为`RedisProperties`

**常用配置**

```java
@Bean
public RedisTemplate redisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
    RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
    redisTemplate.setKeySerializer(RedisSerializer.string());
    redisTemplate.setValueSerializer(RedisSerializer.string());
    redisTemplate.setHashKeySerializer(RedisSerializer.string());
    redisTemplate.setHashValueSerializer(RedisSerializer.string());
    redisTemplate.setConnectionFactory(lettuceConnectionFactory);
    return redisTemplate;
}
```

**yml配置（使用lettuce）**

```yaml
spring:  
  redis:
    password: 123456
    database: 0
    cluster:
      nodes:
        - 192.169.1.101:6379
        - 192.169.1.202:6379
        - 192.169.1.203:6379
        - 192.169.1.211:6379
        - 192.169.1.181:6379
        - 192.169.1.182:6379
    lettuce:
      pool: # 需要依赖commons-pool2包
        enabled: true
        max-active: 8
        max-idle: 8
        min-idle: 0
        max-wait: -1ms
      cluster:
        refresh:
          adaptive: true
          period: 15s
```











