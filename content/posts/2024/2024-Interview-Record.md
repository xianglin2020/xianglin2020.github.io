---
title: "2024面试记录"
date: 2024-09-20T08:23:36+08:00
categories:
  - interview
tags:
  - 面试
summary: "2024社招的几次笔试面试记录。"
description: ""
author: [ "XiangLin" ]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## 部分问题

### Java 各版本新特性

[Java各版本语言特性](https://blog.xianglin.uk/posts/2024/java-version-features/)

### MySQL 大数据量如何优化处理

https://javaguide.cn/high-performance/read-and-write-separation-and-library-subtable.html

* 读写分离和分库分表
* 数据冷热分离
* 深度分页优化
* SQL 优化

### MQ 如何保证消息不丢失

[Kafka 消费顺序、消息丢失和重复消费](https://javaguide.cn/high-performance/message-queue/kafka-questions-01.html#kafka-消费顺序、消息丢失和重复消费)

### 阅读 Spring、MyBatis 源码对工作有和帮助

### Spring Bean 生命周期，三级缓存

* [生命周期](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#bean-%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E4%BA%86%E8%A7%A3%E4%B9%88)

* [Spring 的循环依赖](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-的循环依赖)

### SpringBoot 自动装配

[SpringBoot 自动装配原理详解](https://javaguide.cn/system-design/framework/spring/spring-boot-auto-assembly-principles.html)

[Starter原理解析](https://blog.xianglin.uk/posts/2023/starter-source-code-learning/)

### Spring 事务，哪些场景事物不生效

[Spring 事务详解](https://javaguide.cn/system-design/framework/spring/spring-transaction.html)

### 如何设计对外接口

https://blog.csdn.net/huwei2003/article/details/123862132

* 安全性
    * 使用 appId、appKey 和 appSecret，使用 token，采用 IP 白名单
    * 调用失败警告、记录接口请求日志
    * 采用 HTTPS、数据加密
    * 接口限流、高可用、高并发
* 幂等性
* 数据规范
    * 选择合适的协议，ProtoBuf
    * 接口版本控制，提供接口文档
    * 状态码、数据格式规范

### Redis 使用场景

* 分布式锁（ZooKeeper）
    * SET lockKey uniqueValue EX 3 NX
    * Redisson: Redisson 做了哪些工作？
    * RedLock
* 队列
    * 使用 list 数据结构，rpush/lpush 和 lpop/rpop (blpop/brpop) 操作入队和出队
    * 使用 zset 实现延迟队列，Redisson
    * stream
* 简单限流
* 地理位置
    * Geo
    * PostGIS
    * MongoDB
* 搜索引擎：RedisSearch
* 布隆过滤器

### 定时任务的使用

[XXL-JOB](https://www.xuxueli.com/xxl-job/)

### Kafka 为什么快

[Kafka 性能篇：为何 Kafka 这么快](https://segmentfault.com/a/1190000039702782)

[Kafka吞吐量大的原因](https://blog.xianglin.uk/posts/2023/kafka-basics/#kafka%E5%90%9E%E5%90%90%E9%87%8F%E5%A4%A7%E7%9A%84%E5%8E%9F%E5%9B%A0)

### MySQL 死锁

https://blog.csdn.net/qq_43546721/article/details/140141830

https://blog.csdn.net/AlbenXie/article/details/118613349

### 项目中的 JVM 调优

### Redis Sentinel | Cluster

[多机 Redis](https://blog.xianglin.uk/posts/2021/multi-redis/)

### MySQL binlog | redo log | undo log

[MySQL三大日志](https://javaguide.cn/database/mysql/mysql-logs.html)

### 内存泄漏

Java 中的内存泄漏是指对象已不再被使用，但是在GCRoots有向图中仍可达，导致GC无法回收。

常见的内存泄漏场景：

* 静态集合类，静态集合类的生命周期与应用程序一致，如果集合中对象不再使用但没有被移除，就会导致内存泄漏；
* 未关闭的资源，如数据库连接、文件流、网络连接等未关闭，会导致资源无法释放；
* 使用缓存时未设置清理策略；
* 使用 ThreadLocal 时没有正确调用 remove 方法。

检测和解决内存泄漏：

* 在缓存场景可以使用 WeakReference 或 SoftReference，使得对象在内存不足时能被回收；
* 使用 try-with-resources 确保资源自动关闭；
* 通过代码审查发现问题；
* 使用 VisualVM、Eclipse MAT 等工具分析内存使用情况，找出内存泄漏根源。

## 面试记录

| 公司                       | 时间 | 类型     | 结果                |
| -------------------------- | ---- | -------- | ------------------- |
| 卓牛科技【互联网】         | 0903 | 两轮     | 职位已关闭          |
| 彩讯股份【互联网】         | 0905 | 两轮     | 已读不回            |
| 物格智能【计算机软件】     | 0913 | 一面     | 已读不回            |
| 科大讯飞【教育BG】         | 0913 | 技术面   | 通过                |
| SHEIN【仓储供应链】        | 0914 | 一面     | 已读不回            |
| 科大讯飞【教育BG】         | 0918 | HR 面    | 未通过              |
| 研途科技【游戏】           | 0920 | 两轮     | 通过                |
| 英特仿真【计算机软件】     | 0920 | 一面     | 未通过              |
| 汇智通信【通信/网络设备】  | 0924 | 两轮     | <s>通过</s>中止录用 |
| 阳普智慧医疗【计算机软件】 | 0924 | 记错时间 | 未参加              |
| 雪宝科技【计算机软件】     | 1010 | HR/老板  | 通过                |
| 雪宝科技【计算机软件】     | 1012 | 技术面   | 已读不回            |
| 如祺出行【慧博云通/外包】  | 1012 | 技术面   | 通过                |
| 先进数通【计算机软件】     | 1015 | 技术面   | 未通过              |
| 广州银行【中电金信/外包】  | 1018 | 技术面   | 已读不回            |
| 易方达【博彦科技/外包】    | 1025 | 一面     | 未通过              |
| 悦信科技【互联网】         | 1202 | 三轮     | 未反馈              |
| 启辰科技【计算机软件】     | 1227 | 技术面   | 通过                |
| 启辰科技【计算机软件】     | 0103 | 老板     | 未反馈              |
| 佳都数据【互联网】         | 0107 | 技术面   | 未反馈              |
| 贤能数科【互联网金融】     | 0115 | 两轮     | 未反馈              |
| 行稳网络【医疗健康】       | 0124 | 技术     | 通过                |
| 国创中心【软通动力/外包】  | 0210 | 技术     | 通过                |
| 南方电网【煜邦/外包】      | 0211 | 初试     | 通过                |
| 南方电网【煜邦/外包】      | 0212 | 复试     | 通过                |
| 行稳网络【医疗健康】       | 0212 | 复试     | 凉凉                |
| 广州领克科技【计算机软件】 | 0213 | 技术/HR  | 通过                |
| 翰智软件【计算机软件】     | 0213 | 一轮     | 凉凉                |
| 国创中心【软通动力/外包】  | 0214 | 复试     | 通过                |
| 国创中心【软通动力/外包】  | 0217 | HR       | 未参加              |
| 广州骏伯网络【互联网】     | 0217 | 初试     | 通过                |

总结：

1. 这两年基本没进步
2. 有几个好工作也没把握住
3. 一手好牌打得稀烂，工资骤减履历也毁了
4. 切实沉淀两年后再做打算
