---
title: Redis Sentinel
date: 2021-02-27 10:44:46
categories: [learn]
tags: [redis]
---

# Redis Sentinel

## Redis Sentinel 的作用

Redis 的 Sentinel 系统用于管理多个 Redis 服务器（instance），Redis Sentinel 主要有三个作用：

* 监控（Monitoring）：Sentinel 会不断的检查 Redis 主服务器和从服务器是否运行正常。
* 提醒（Notification）：当被监控的某个 Redis 服务器出现问题时，Sentinel 可以通过 API 向管理员或其它应用程序发送通知。
* 自动故障迁移（Atomic failover）：当一个主服务器不能正常工作时，Sentinel 会开始启动一次自动故障迁移。它会将失效主服务器的其中一个从服务器升级为新的主服务器，并让其它从服务器改为同步新主服务器的数据。当客户端试图连接失效主服务器时，集群会向客户端返回新主服务器的地址。

Redis Sentinel 是一个分布式系统，可以在一个架构中运行多个 Sentinel 进程，这些进程使用流言协议（Gossip Protocol）来接收关于主服务器是否下线的信息，并使用投票协议（Agreement Protocol）来决定是否执行自动故障迁移，以及选择哪个从服务器作为新的主服务器。

## Redis 主从复制

有两种方式可以配置 Redis 的复制功能，分别是使用`slaveof host port`命令或修改 Slave 的配置文件。

先复制三个配置文件，分别为`master.conf`、`slave1.conf`、`slave2.conf`，修改对应的 `port` 和 `deamon yes` 配置，如图：

![conf](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/121329.png)

然后使用如下命令启动 master 进程：

```bash
src/redis-server conf/master.conf
```

### 使用`slaveof`命令配置复制功能

使用如下命令启动 Slave1 进程：

```bash
src/redis-server conf/slave1.conf
```

然后登陆 Slave1，使用 `slaveof` 命令配置复制功能，如图所示：

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/113540.png" alt="image-20210227113539217" style="zoom: 50%;" /><img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/114440.png" alt="image-20210227114440310" style="zoom:50%;" />

### 使用配置文件配置复制功能

为 `slave2.conf` 中添加如下配置项：

```properties
slaveof 127.0.0.1 6379
```

然后启动 slave2 进程，如图所示：

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/114911.png" alt="image-20210227114910857" style="zoom: 33%;" /><img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/114937.png" alt="image-20210227114937122" style="zoom:50%;" />

连接至 master 进程后使用`info`命令查看 `Replication`：

![image-20210227115121194](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/115121.png)

* [ ] TODO
