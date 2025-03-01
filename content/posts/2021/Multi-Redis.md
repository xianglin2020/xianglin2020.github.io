---
title: 多机 Redis
date: 2021-02-27 10:44:46
categories: [learn]
tags: [redis]
summary: "Redis 主从复制、哨兵、集群配置。"
---

## 单机问题

* 机器故障
* 容量瓶颈
* QPS瓶颈

## Redis 安装

使用 Debian 12 从源码中安装 Redis，操作的主目录在 `/root/redis`。

https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-from-source/

```bash
# 安装编译环境
apt install build-essential tcl

# 下载源码包安装
wget https://download.redis.io/redis-stable.tar.gz
tar -xzvf redis-stable.tar.gz
cd redis-stable
make

# 启动 redis
./src/redis-server
```

## Redis 主从复制

提供了数据副本、扩展了读性能。

### 主从结构搭建

有两种方式可以配置 Redis 的复制功能，分别是使用`slaveof host port`命令或修改 Slave 的配置文件。

创建目录存放配置、日志和数据文件。

```bash
# 位于 /root/redis
mkdir conf
mkdir data
mkdir log
```

创建主节点配置文件 `conf/redis.conf`。

```bash
# 放行 IP
bind 0.0.0.0
# 后台启动
daemonize yes
# 日志存储
logfile "/root/redis/log/redis.log"
# rdb 数据文件
dbfilename dump.rdb
# 开启 aof 模式
appendonly yes
appendfilename "appendonly.aof"
# 数据文件目录
dir /root/redis/data
# 密码
requirepass 123456
# 从节点访问主节点的密码
masterauth 123456
# 从节点只读
replica-read-only yes
```

**从节点配置文件增加一行：**

```bash
# 隶属的主节点
slaveof 192.168.56.101 6379
```

启动 redis。

```bash
# 位于 /root/redis
redis-stable/src/redis-server conf/redis.conf
```

登录 redis 查看配置主从信息。

```bash 
redis-stable/src/redis-cli -a 123456

# 登录 redis 后
info replication
```

主节点信息

![image-20241014203617999](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410142036067.png)

从节点信息

![image-20241014203639703](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410142037670.png)

### 取消复制

在 slave 节点上执行 `slaveof no one`。

## Redis Sentinel

### Redis Sentinel 的作用

Redis 的 Sentinel 系统用于管理多个 Redis 服务器（instance），Redis Sentinel 主要有三个作用：

* 监控（Monitoring）：Sentinel 会不断的检查 Redis 主服务器和从服务器是否运行正常。
* 提醒（Notification）：当被监控的某个 Redis 服务器出现问题时，Sentinel 可以通过 API 向管理员或其它应用程序发送通知。
* 自动故障迁移（Atomic failover）：当一个主服务器不能正常工作时，Sentinel 会开始启动一次自动故障迁移。它会将失效主服务器的其中一个从服务器升级为新的主服务器，并让其它从服务器改为同步新主服务器的数据。当客户端试图连接失效主服务器时，集群会向客户端返回新主服务器的地址。
* 配置提供（Configuration provider）：客户端连接 sentinel，以查询当前负责特定服务的 Redis 主节点地址，如果发生故障转移，sentinel 会报告新的地址。

Redis Sentinel 是一个分布式系统，可以在一个架构中运行多个 Sentinel 进程，这些进程使用流言协议（Gossip Protocol）来接收关于主服务器是否下线的信息，并使用投票协议（Agreement Protocol）来决定是否执行自动故障迁移，以及选择哪个从服务器作为新的主服务器。

### Sentinel 搭建

先配置开启主从节点。

创建配置文件 `conf/sentinel.conf`

```bash
bind 0.0.0.0
port 26379
daemonize yes
logfile "/root/redis/log/sentinel.log"
pidfile "/var/run/sentinel.pid"
sentinel monitor mymaster 192.168.56.101 6379 2
sentinel auth-pass mymaster 123456
sentinel down-after-milliseconds mymaster 10000
```

启动 sentinel

```bash
redis-stable/src/redis-server conf/redis.conf

redis-stable/src/redis-sentinel conf/sentinel.conf
```

查看日志

![image-20241014213641945](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410142137521.png)

### 客户端通过 Sentinel 获取主节点地址

1. 连接到一个或多个 sentinel 节点；
2. 通过 `sentinel get-master-addr-by-name <name>` 命令查询主节点信息；
3. 客户端解析 sentinel 返回的主节点信息；
4. 客户端使用获取到的主节点信息连接到主节点，开始进行数据操作；
5. 如果主节点故障，sentinel 会选举新的主节点，客户端需要重新查询 sentinel 获取新的主节点地址并重新连接；
6. 客户端可以订阅 sentinel 的事件通知 `+switch-master`，以便在主节点变更时及时获取通知并重新连接；

   `+convert-to-slave`：切换从节点（元主节点将为从节点）

   `+sdown`：主观下线

### Sentinel 故障转移

1. 检测主节点故障

   sentinel 节点会定期向主节点、从节点和其他 sentinel 节点发送心跳检测（PING 命令）；

   如果主节点在配置的超时时间内（`down-after-milliseconds`）未响应，sentinel 会将其标记为主观下线（SDOWN）；

   当多个 sentinel 节点都认为主节点不可用时，sentinel 会将其标记为客观下线（ODOWN）；

2. sentinel 集群会通过 Raft 算法选举 Sentinel Leader 来负责故障转移；

3. Sentinel Leader 从当前从节点中选择一个作为新的主节点，选举规则包括：

   1. 优先级：从节点的 `slave-priority` 配置值；
   2. 复制偏移量：选择复制数据最完整的从节点；
   3. 运行 ID：选择 run_id 最小的从节点。

4. Sentinel Leader 会向选中的从节点发送 `slaveof no one` 命令，将其提升为新的主节点；

5. Sentinel Leader 会向其他从节点发送 `slaveof` 命令，让它们开始复制新的主节点；

6. Sentinel 会通过发布订阅向客户端发送 `+switch-master` 事件，通知主节点已变更；

7. 故障转移完成后 sentinel 会更新其内部状态，同时将故障转移结果持久化到配置文件中，以便再重启后保持一致性。

8. 如果旧的主节点恢复，Sentinel 会将其降级为从节点，并让其复制新的主节点。

### sentinel 工作原理

Sentinel 内部有3个定时任务：

* 每1秒每个 sentinel 对其它 sentinel 和 redis 节点执行 `PING` 操作（监控）；
* 每2秒每个 sentinel 通过 master 节点的 channel 交换信息（publish/subscribe）；
* 每10秒每个 sentinel 会对 master 和 slave 执行 `INFO` 命令。

## Redis 集群架构

Redis Cluster 主要解决并发量和数据量问题。

### 集群搭建

使用 3 个节点启动 6 个 redis 服务，节点 ip 分别为 192.168.56.10[1,2,3]，redis 服务端口分别为 637[1-6]。

创建目录存放配置、数据和日志文件。

```bash
mkdir -p redis/cluster/conf redis/cluster/data redis/cluster/log
```

创建配置文件 `redis/cluster/conf/redis-6371.conf`

```bash
bind 0.0.0.0
port 6371
daemonize yes
logfile "/root/redis/cluster/log/redis-6371.log"
dbfilename dump-6371.rdb
appendonly yes
appendfilename "appendonly-6371.aof"
dir /root/redis/cluster/data
requirepass 123456
masterauth 123456

# 集群配置
cluster-enabled yes
cluster-config-file nodes-6371.conf
cluster-node-timeout 15000
cluster-announce-ip 192.168.56.101
cluster-announce-port 6371
cluster-announce-bus-port 16371
```

不同服务使用的配置文件不同，体现在端口和命名后缀上，将与 6371 和 56.101 有关的值依次递增，在 3 个节点上创建 6 份配置文件。

依次启动 6 个 redis 服务

```bash
redis-stable/src/redis-server cluster/conf/redis-6371.conf
```

随便选取一个节点执行 cluster 创建操作

```bash
redis-stable/src/redis-cli -a 123456 --cluster create 192.168.56.101:6371 192.168.56.101:6372 192.168.56.102:6373 192.168.56.102:6374 192.168.56.103:6375 192.168.56.103:6376 --cluster-replicas 1
```

![image-20241016114106169](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410161141307.png)

确认配置无误后输入 `yes` 即可

![image-20241016114139993](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410161141098.png)

### 集群管理

#### 集群状态

检查集群状态

```bash
redis-stable/src/redis-cli -a 123456 --cluster check 192.168.56.101:6371
```

查看集群信息

```bash
# 连接到 redis cluster
redis-stable/src/redis-cli -c -a 123456 -h 192.168.56.101 -p 6371

# 集群信息
cluster info

# 节点信息
cluster nodes
```

#### 性能测试

集群性能测试

```bash
redis-stable/src/redis-benchmark -a 123456 -h 192.168.56.101 -p 6371 --cluster -t set,get -r 1000000 -n 1000000 -c 1000
```

#### 添加节点

添加主节点

```bash
# 添加主节点
# add-node new-node exists-master-node
redis-stable/src/redis-cli -a 123456 --cluster add-node 192.168.56.103:6377 192.168.56.103:6375 --cluster-master-id db9b855f6db790d5a99c7062bd1a98107e1949ce
```

![image-20241017084258937](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410170845318.png)

```bash
# 重新分片
redis-stable/src/redis-cli -a 123456 --cluster reshard 192.168.56.101:6371 --cluster-from 76d2931297beabb29c1263e45dba6a6b2635d194 --cluster-to b6ff5ce0bc981184e21f5b2e7fcd32056999cd35 --cluster-slots 2000 --cluster-yes
```

![image-20241017085136009](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410170851115.png)

添加从节点

```bash
# add-node new-node replication-from-node
redis-stable/src/redis-cli -a 123456 --cluster add-node 192.168.56.103:6378 192.168.56.103:6377 --cluster-slave --cluster-master-id b6ff5ce0bc981184e21f5b2e7fcd32056999cd35
```

![image-20241017085909248](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410170859330.png)

#### 删除节点

删除从节点

```bash
# 从集群中移除从节点
redis-stable/src/redis-cli -a 123456 --cluster del-node 192.168.56.103:6378 b2ee915237e7e1163090ce73000592d6b3445d90
# 停止 redis 服务
redis-stable/src/redis-cli -a 123456 -h 192.168.56.103 -p 6378 shutdown
```

![image-20241017091435943](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410170914027.png)

删除主节点

先执行重新分片命令，将主节点上的 slot 移到其它主节点上。

```bash
# 交换式命令
redis-stable/src/redis-cli -a 123456 --cluster reshard 192.168.56.101:6371
```

![image-20241017092150658](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410170921749.png)

删除节点

```bash
# 从集群中移除节点
redis-stable/src/redis-cli -a 123456 --cluster del-node 192.168.56.103:6377 b6ff5ce0bc981184e21f5b2e7fcd32056999cd35
# 停止 redis 服务
redis-stable/src/redis-cli -a 123456 -h 192.168.56.103 -p 6377 shutdown
```

## Redis 数据迁移

[RedisShake](https://tair-opensource.github.io/RedisShake/zh/)

