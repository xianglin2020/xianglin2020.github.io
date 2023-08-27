---
title: Redis 基础
date: 2020-10-03 20:53:49
categories: [learn]
tags: [redis]
cover:
  image: "https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202101/Redis%E7%BC%93%E5%AD%98.png"
  caption: ""
  alt: ""
  relative: false
---

![Redis 缓存](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202101/Redis%E7%BC%93%E5%AD%98.png)

## Redis 介绍

### Redis 简介

* Redis 是 `key-value` 型 NoSQL 数据库
* Redis 将数据存储在内存中，同时也能持久化到磁盘中
* Redis 常用于缓存，利用内存的高效提高程序的处理速度

### Redis 特点

* 速度快
* 广泛的语言支持
* 支持数据磁盘持久化
* 多种数据结构
* 支持主从复制
* 支持分片
* 分布式与高可用

### Redis 常用配置项

| 配置          | 示例                 | 说明                  |
| ------------- | -------------------- | --------------------- |
| `daemonize`   | `daemonize yes`      | 是否启用后台运行      |
| `port`        | `port 6379`          | 设置端口号            |
| `databases`   | `databases 16`       | 设置 redis 数据库个数 |
| `requirepass` | `requirepass abc123` | 设置 redis 连接密码   |

### Redis通用命令

* `select`
* `set`
* `get`
* `keys`
* `dbsize`
* `exists`
* `del`
* `expire`
* `ttl`

### Redis 数据结构

* `string`：字符串，值的长度不能超过 512 MB，是二进制安全的。
  1. `set key value [EX seconds] [PX milliseconds] [NX|XX]`
     * `EX`：将键的过期时间设置为`seconds`秒，`set key value ex seconds`等同于`setex key seconds value`
     * `PX`：将键的过期时间设置为`milliseconds`毫秒，与`psetex key milliseconds value` 等价
     * `NX|XX`：只有在键不存在、存在时，才对键执行设置操作，`set key value NX`等同于`setnx key value`
  2. `mset key value [key value ...] mget key [key ...]`
  3. `incr key | incrby k increment`
  4. `decr key | decrby key decrement`
  5. `exists key [key ...]  del key [key ...]`
  6. `expire key seconds  ttl key `
* `hash`：键值类型
* `list`：列表，Redis lists基于Linked Lists实现。
* `set`：无序排列
* `zset`：有序排列

## Redis

### Redis 为啥这么快

Redis 支持 100000+QPS。

* 完全基于内存，绝大部分请求是纯粹的内存操作，执行效率高
* 数据结构简单，对数据操作也简单
* 采用单线程，单线程能处理高并发请求
* 使用 I/O 多路复用，Redis 使用的 I/O 多路复用函数：epoll、kqueue、evport

传统的阻塞 I / O 模型

![image-20210116223341983](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202101/image-20210116223341983.png)

### Redis 数据类型

* String：最基本的数据类型，二进制安全。
* Hash：键值类型，String 元素组成的字典，适用于存储对象。
* List：列表，按照 String 元素插入顺序排序。
* Set：String 元素组成的无序集合，基于哈希表实现，不允许重复。
* Sorted Set：通过分数来为集合中的成员进行从小到大的排序。
* 用于计数的 HyperLogLogs，用于支持存储地址位置信息的 Geo

### 从海量 key 里查询出某一个固定前缀的 key

* `KEYS pattern`：查找所有符合给定模式 pattern 的 key。

  * `KEYS` 指令一次性返回所有匹配的 key。
  * 键的数量过大会使服务卡顿。

* `SCAN cursor [MATCH pattern] [COUNT count] [Type type]`

  * 基于游标的迭代器，需要基于上一次的游标延续之前的迭代过程。
  * 以 0 作为游标开始一次新的迭代，直到命令返回游标 0 完成一次遍历。
  * 不保证每次执行都返回某个给定数量的元素，支持模糊查询。
  * 一次返回的数量不可控，只能是大概率符合 count 参数。

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202012/image-20201229224204948.png" alt="image-20201229224204948" style="zoom:50%;" />

### 通过 Redis 实现分布式锁

分布式锁需要解决的问题：

* 互斥性
* 安全性
* 死锁
* 容错

`SETNX key value` ：如果 key 不存在，则创建并赋值。

* 时间复杂度：O(1) 。
* 返回值：设置成功，返回 1 ；设置失败，返回 0 。
* SETNX 是原子操作。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202012/image-20201229225010259.png" alt="image-20201229225010259" style="zoom:67%;" />

使用 `EXPIRE key seconds` 设置过期时间

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202012/image-20201229225447919.png" alt="image-20201229225447919" style="zoom:67%;" />



`set key value [EX seconds] [PX milliseconds] [NX|XX]`

* `EX`：将键的过期时间设置为`seconds`秒，`set key value ex seconds`等同于`setex key seconds value`
* `PX`：将键的过期时间设置为`milliseconds`毫秒，与`psetex key milliseconds value` 等价
* `NX|XX`：只有在键不存在、存在时，才对键执行设置操作，`set key value NX`等同于`setnx key value`

### 使用 Redis 实现异步队列

* 方式一：

  使用 Redis 的 List 数据类型作为队列，使用 `RPUSH` 生产消息和 `LPOP` 消费消息。

  缺点：不能阻塞等待消息。

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202012/image-20201230205130900.png" alt="image-20201230205130900" style="zoom:67%;" />

* 方式二：

  使用 `BLPOP key [key...] timeout` ：阻塞直到队列有消息或者超时。

   缺点：消息不能被多个消费者使用。

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202012/image-20201230205425336.png" alt="image-20201230205425336" style="zoom:67%;" />

* pub / sub ：主题订阅模式：

  发送者 pub 发送消息，订阅者 sub 接收消息；订阅者可以订阅任意数量的频道。

  缺点：消息的发布是无状态的，无法保证可达。
  
  ![image-20210117114605047](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202101/image-20210117114605047.png)

### Redis 持久化

* RDB（快照）持久化，保存某个时间点的全量数据的快照。

  * `SAVE`：阻塞 Redis 的服务器进程，直到 RDB 文件被创建完毕。
  * `BGSAVE`：fork 出一个子进程来创建 RDB 文件，不阻塞服务器进程。
  * `lastsave`：上次成功保存快照的时间戳。

  缺点：

  * 内存数据的全量同步，数据量大会由于 I/O 而严重影响性能。
  * 可能因为 Redis 服务异常丢失从当前至最近一次快照期间的数据。

  自动触发 RDB 持久化的方式：

  * 根据 redis.conf 配置的触发时机定时触发（使用 `BGSAVE`）。
  * 主从复制时，主节点自动触发。
  * 执行 Debug Reload。
  * 执行 shutdown 且没有开启 AOF 持久化。

* AOF（Append-Only-File）持久化：保存写状态

  * 记录下除了查询以外的所有变更数据库状态的指令。
  * 以 append 的形式追加保存到 AOF 文件中。
  * 使用 `appendonly yes` 配置开启 AOF 持久化。
  * `appendfsync everysec`

  日志重写解决 AOF 文件不断增大的问题：

  > 在执行 `BGREWRITEAOF` 命令时,Redis 服务器会维护一个 AOF 重写缓冲区,该缓冲区会在子进程创建
  > 新AOF文件期间,记录服务器执行的所有写命令。当子进程完成创建新AOF文件的工作之后,服务器会将
  > 重写缓冲区中的所有内容追加到新AOF文件的末尾,使得新旧两个AOF文件所保存的数据库状态一致。最
  > 后,服务器用新的AOF文件替换旧的AOF文件,以此来完成AOF文件重写操作。

  * 调用 fork，新建一个子进程；
  * 子进程把新的 AOF 写到一个临时文件里，不依赖原来的 AOF 文件；
  * 主基础持续将新的变动同时写到内存和原来的 AOF 文件里；
  * 主进程获取子进程重写 AOF 的完成信号，往新的 AOF 同步增量变动；
  * 使用新的 AOF 文件替换掉旧的 AOF 文件。

* 混合持久化

  Redis 4.0 开始支持 RDB 和 AOF 的混合持久化（默认关闭,可以通过配置项 `aof-use-rdb-preamble` 开启）。

  AOF 重写的时候就直接把 RDB 的内容写到 AOF 文件开头。这样做的好处是可以结合 RDB 和 AOF 的优点, 快速加载同时避免丢失过多的数据。当然缺点也是有的, AOF 里面的 RDB 部分是压缩格式不再是 AOF 格式,可读性较差。

### 为什么使用 Pipline

Redis 是一种基于客户端-服务端模型及请求/响应协议的 TCP 服务，Pipeline 批量执行指令，节省 IO 时间。

### Redis 的同步机制

* 全同步过程

    * Slave 发送 sync 命令到 Master
    * Master 启动一个后台进程，将 Redis 中的数据快照保存到文件中
    * Master 将保存数据快照期间接收到的写命令缓存起来
    * Master 完成写文件操作后，将该文件发送给 Slave
    * Slave 使用新的 RDB 文件替换掉旧的 RDB 文件，并根据其恢复到内存中
    * Master 将这期间收集的增量写命令发送给 Slave 端

* 增量同步过程

    * Master 接受到用户的操作指令，判断是否需要传播到 Slave
    * 将该操作记录追加到 AOF 文件
    * 将操作传播到其他 Slave：1. 对齐主从库；2. 往响应缓存写入指令
    * 将缓存中的数据发送给 Slave

* Redis Sentinel

    解决主从同步 Master 宕机后的主从切换问题

    * 监控：检查主从服务器是否运行正常
    * 提醒：通过 API 向管理员或其它应用程序发送故障通知
    * 自动故障迁移：主从切换

### Redis集群原理

* 一致性哈希算法：对 2^32 取模，将哈希值空间组成虚拟的圆环。
* 将数据 key 使用相同的 hash 函数计算出哈希值。
* 引入虚拟节点解决数据倾斜的问题。

#### Redis 集群的数据分片

Redis 集群有 16384 个哈希槽，每个 key 通过 CRC16 校验后对 16384 取模来决定放置在哪个槽，集群中的每个节点负责一部分哈希槽。
