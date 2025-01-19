---
title: Zookeeper 基础
date: 2020-12-13 13:25:23
categories: [learn]
tags: [Zookeeper]
cover:
  image: "https://zookeeper.apache.org/images/zookeeper_small.gif"
  caption: ""
  alt: ""
  relative: false
---

# Zookeeper

## 理解 Zookeeper

### 为什么需要 Zookeeper

* 用起来像单机但是又比单机更可靠
* 类似于 Leader 在团队里面的协调作用
* 在多节点中尽量压缩数据同步时间

### Zookeeper 是什么

* Zookeeper 是开源的高性能的分布式应用协调系统，一个高性能的分布式数据一致性解决方案
* 它的设计目标是将那些复杂的容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户。

### Zookeeper 的特点

* 顺序一致性：从同一客户端发起的事务请求，会严格的按照顺序被应用到 Zookeeper 服务器中
* 原子性：所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的
* 单一视图：无论连接哪一个 Zookeeper 节点，看到的服务端数据模型都是一致性的
* 可靠性：一旦一次更改请求被应用，更改的结果就会被持久化，直到被下一次更改覆盖
* 及时性

### Zookeeper 与 CAP 的关系

* CP：一致性+分区容错性
* 任何时候都能得到一致的数据结果，同时系统对网络具有容错性
* 它不能保证每次服务请求的可用性

### Zookeeper 的作用

* 分布式服务注册与订阅
* 统一配置文件
* 生成分布式唯一 ID：通过 Zookeeper 的顺序节点生成全局唯一ID
* Master 节点选举
* 分布式锁：通过创建唯一节点获得分布式锁，当获取锁的一方执行完代码或挂掉后就释放锁
* 数据发布/订阅：通过 Watcher 机制可以很方便的实现数据的发布与订阅。当你将数据发布到 Zookeeper 被监听的节点上，其它机器可以通过监听 Zookeeper 上节点的变化来实现配置的

## Zookeeper 的重要概念

### 数据模型 DataModel

![image-20201213141229663](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/172917.png)

* Zookeeper 数据模型采用层次化的多叉树结构，每个节点上可以存储数据（数字、字符串、二进制序列）
* 每个节点还可以用于多个子节点，最上层是根节点，用`/`表示
* 每个数据节点在 Zookeeper 中被称为 znode，它是 Zookeeper 中数据的最小单元
* 每个 znode 都有一个唯一的路径标识

### 数据节点 Znode

### znode 的四种类型

* 持久节点（PRESISTENT）：一旦创建就一直存在即是 Zookeeper 集群宕机，直到被删除
* 临时节点（EPHEMERAL）：临时节点与客户端会话（Session）绑定，会话消失则节点消失。临时节点只能做叶子节点，不能创建子节点
* 持久顺序节点（PRESISTENT_SEQUENTIAL）：子节点的名称具有顺序
* 临时顺序节点（EPHEMERAL_SEQUENTIAL）：子节点的名称具有顺序

### Znode 的数据结构

每个 znode 由两部分组成

* stat：状态信息
* data：节点存放的数据的具体内容

```shell
[zk: 127.0.0.1:2181(CONNECTED) 6] get /zk_node
content

[zk: 127.0.0.1:2181(CONNECTED) 7] stat /zookeeper
# create ZXID 数据节点被创建时的事务 ID
cZxid = 0x0
# create time 节点的创建时间
ctime = Thu Jan 01 08:00:00 CST 1970
# modified ZXID 节点最后一次更新时的事务 ID
mZxid = 0x0
# modified time 节点最后一次更新时间
mtime = Thu Jan 01 08:00:00 CST 1970
# 该节点的子节点列表最后一次修改时的事务 ID，只有子节点列表变更才会更新 pZxid，子节点内容变更不会更新
pZxid = 0x0
# 子节点版本号，当前节点的子节点每次变化时值增加 1
cversion = -2
# 数据节点内容版本号，节点创建时为 0，每更新一次节点内容（无论数据有无变化）该版本号值增加 1
dataVersion = 0
# 节点的 ACL 版本号，表示该节点 ACL 信息变更次数
aclVersion = 0
# 创建该临时节点的会话 sessionId，如果当前节点为持久节点，则 ephemeralOwner = 0
ephemeralOwner = 0x0
# 数据节点内容长度
dataLength = 0
# 当前节点的子节点个数
numChildren = 2
```

### 权限控制 ACL

Zookeeper 采用 ACL（AccessControlLists）策略进行权限控制，类似于 UNIX文件系统的权限控制

对于 znode 操作的权限，Zookeeper 提供了一下 5 种：

* CREATE：能够创建子节点
* READ：能回去节点数据和列出其子节点
* WRITE：能设置/更新节点数据
* DELETE：能删除子节点
* ADMIN：能设置节点 ACL 的权限

对于身份认证，提供了一下几种方式：

* world：默认方式，任何用户都可以无条件访问
* auth：代表任何已认证的用户
* digest：用户名:密码认证方式：username:password
* IP：对指定 IP 进行限制

### Watcher 事件监听器

![img](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/180938.png)

* Watcher 事件监听器，是 Zookeeper 中一个很重要的特征。
* Zookeeper 允许用户在指定的节点上注册一些 Watcher，并且在一些特定事件触发时，Zookeeper 服务端会将事件通知到感兴趣的客户端上去，该特性是 Zookeeper 实现分布式协调服务的重要特性。
* 使用场景：统一资源配置
* Watcher 事件类型
  * NodeCreated
  * NodeDeleted
  * NodeDataChanged
  * NodeChildrenChanged

### 会话 Session

* Session 可以看做是 Zookeeper 服务器与客户端之间的一个 TCP 长连接，客户端能够通过心跳检测与服务器保持有效的会话，也能够向 Zookeeper 服务器发送请求并接受响应，同时还能够通过该连接接受来自服务器的 Watcher 事件通知。
* Session 有一个属性叫做 sessionTimeout，代表会话的超时时间。客户端断开连接后，只要在 sessionTimeout 规定的时间内能够重新连接上集群中任意一台服务器，那么之前创建的会话仍然有效。
* 在为客户端创建会话之前，服务端首先会为每个客户端都分配一个 sessionID，sessionID 是 Zookeeper 会话的一个重要标识，是全局唯一的。

## Zookeeper 集群

![image-20201213134852212](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/181709.png)

* 每个 Server 代表一个运行 Zookeeper 服务的服务器。组成 Zookeeper 服务的服务器会在内存中维护当前的服务器状态，并且每台服务器之间都互相保持着通信。
* 集群间通过 ZAB 协议（Zookeeper Atomic BroadCast）来保持数据的一致性。

### Zookeeper 集群角色

Zookeeper 没有选择传统的 Master/Slave 概念，而是引入了 Leader、Follower和 Observer 三种角色。

![zookeeper集群中的角色.png](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/182405.png)

* Zookeeper 集群中所有机器通过一个 Leader 选举过程来选定一台称为 Leader 的机器。
* Leader 可以为客户端提供读写服务。除 Leader 外，Follower 和 Observer 都只能提供读服务。
* Follower 和 Observer 的唯一区别在于 Observer 集群不参与 Leader 的选举过程，也不参与写操作的“过半写成功”策略。

当 Leader 服务器出现异常时，就会进入 Leader 选举过程，这个过程会选举产生新的 Leader 服务器。

### Zookeeper 集群中服务器状态

* LOOKING：寻找 Leader
* LEADING：Leader 状态，对应节点为 Leader
* FOLLOWING：Follower 状态，对应节点为 Follower
* OBSERVING：Observer 状态，对应节点为 Observer，该节点不参与 Leader 选举

### Zookeeper 集群最好是基数台

* Zookeeper 集群在宕机几个 Zookeeper 服务器后，如果剩下的 Zookeeper 服务器个数大于宕机个数的话整个 Zookeeper 才依然可用。

## ZAB 协议和 Paxos 算法

### ZAB 协议

* ZAB（Zookeeper Atomic BroadCast 原子广播）协议是为分布式协调服务 Zookeeper 专门设计的一种支持崩溃恢复的原子广播协议。
* 在 Zookeeper 中，主要依赖 ZAB 协议来实现分布式数据一致性。基于该协议，Zookeeper 实现了一种主备模式的系统架构来保持集群中各个副本之间的数据一致性。

### ZAB 协议的两种基本模式

* 崩溃恢复：当整个服务框架在启动过程中，或是当 Leader 服务器出现网络中断、崩溃退出与重启等异常情况时，ZAB 协议就会进入恢复模式并选举产生新的 Leader 服务器。当选举产生了新的 Leader 服务器，同时集群中已经有过半的机器与该 Leader 服务器完成了状态同步之后，ZAB 协议就会退出恢复模式。其中，状态同步是指数据同步，用来保证集群中存在过半的机器能够和 Leader 服务器的数据状态保持一致。
* 消息广播：当集群中已经有过半的 Follower 服务器完成了和 Leader 服务器的状态同步，那么整个服务框架就可以进入消息广播模式了。当一台同样遵守 ZAB 协议的服务器启动后加入到集群中时，如果此时集群中已经存在一个 Leader 服务器在负责进行消息广播，那么新加入的服务器就会自觉地进入数据恢复模式：找到 Leader 所在的服务器，并与其进行数据同步，然后一起参与到消息广播流程中去。

## 安装、配置 Zookeeper

### CentOS 下安装 Zookeeper

* 使用 wget 命令下载 Zookeeper

  ```shell
  wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz
  ```

* 解压使用

  ```shell
  tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz
  ```

* 启动停止

  ```shell
  ➜  bin ./zkServer.sh start
  /usr/bin/java
  ZooKeeper JMX enabled by default
  Using config: /root/zookeeper/apache-zookeeper-3.6.2-bin/bin/../conf/zoo.cfg
  Starting zookeeper ... STARTED
  ➜  bin ./zkServer.sh stop
  /usr/bin/java
  ZooKeeper JMX enabled by default
  Using config: /root/zookeeper/apache-zookeeper-3.6.2-bin/bin/../conf/zoo.cfg
  Stopping zookeeper ... STOPPED
  ```

  

## Zookeeper 的使用

### Zookeeper 的常用命令

* 启动 Zookeeper

  ```shell
  ➜  apache-zookeeper-3.6.2-bin bin/zkServer.sh start
  /usr/bin/java
  ZooKeeper JMX enabled by default
  Using config: /root/zookeeper/apache-zookeeper-3.6.2-bin/bin/../conf/zoo.cfg
  Starting zookeeper ... STARTED
  ```

* 连接到 Zookeeper

  ```shell
  ➜  apache-zookeeper-3.6.2-bin bin/zkCli.sh -server 127.0.0.1:2181
  /usr/bin/java
  Connecting to 127.0.0.1:2181
  # ....
  WatchedEvent state:SyncConnected type:None path:null
  [zk: 127.0.0.1:2181(CONNECTED) 0]
  ```

* 查看节点`ls`

  ```shell
  [zk: 127.0.0.1:2181(CONNECTED) 1] ls /
  [zookeeper]
  
  [zk: 127.0.0.1:2181(CONNECTED) 2] ls /zookeeper
  [config, quota]
  ```

* 查看状态`stat`

  ```shell
  [zk: 127.0.0.1:2181(CONNECTED) 3] stat /
  cZxid = 0x0
  ctime = Thu Jan 01 08:00:00 CST 1970 # 创建时间
  mZxid = 0x0
  mtime = Thu Jan 01 08:00:00 CST 1970 # 修改时间
  pZxid = 0x0
  cversion = -1
  dataVersion = 0
  aclVersion = 0
  ephemeralOwner = 0x0 # 节点类型
  dataLength = 0
  numChildren = 1 # 子节点数量
  ```


* 创建、删除节点

  ```shell
  # 创建节点
  [zk: 127.0.0.1:2181(CONNECTED) 11] create /xianglin 123
  Created /xianglin
  
  # 创建顺序节点
  [zk: 127.0.0.1:2181(CONNECTED) 0] create -s /xianglin/s
  Created /xianglin/s0000000000
  [zk: 127.0.0.1:2181(CONNECTED) 1] create -s /xianglin/s
  Created /xianglin/s0000000001
  
  # 创建临时节点
  [zk: 127.0.0.1:2181(CONNECTED) 2] create -e /xianglin/tmp 123
  Created /xianglin/tmp
  # 断开连接后重新连接
  [zk: 127.0.0.1:2181(CONNECTED) 2] stat /xianglin/tmp
  Node does not exist: /xianglin/tmp
  
  # 修改节点
  [zk: 127.0.0.1:2181(CONNECTED) 14] set /xianglin 456
  # 乐观锁的使用
  [zk: 127.0.0.1:2181(CONNECTED) 6] set -v 1 /xianglin 123
  version No is not valid : /xianglin
  
  # 删除节点
  [zk: 127.0.0.1:2181(CONNECTED) 22] ls /xianglin
  [s0000000001]
  [zk: 127.0.0.1:2181(CONNECTED) 23] delete /xianglin/s0000000001
  [zk: 127.0.0.1:2181(CONNECTED) 24] delete /xianglin
  [zk: 127.0.0.1:2181(CONNECTED) 25]
  ```

* 查看节点数据和状态`get`

  ```shell
  [zk: 127.0.0.1:2181(CONNECTED) 12] get /xianglin
  123
  ```

## Java 使用 Zookeeper

### 使用原生的 JavaAPI

* 引入 Zookeeper 依赖

  ```xml
  <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.6.2</version>
  </dependency>
  ```

* 简单的使用

  ```java
  
  ```

### 原生 Java API 的缺点

* 不支持连接超时后的自动重连
* Watcher 注册一次后会失效
* 不支持递归创建节点

### 使用 Apache Curator

* [ ] TODO
