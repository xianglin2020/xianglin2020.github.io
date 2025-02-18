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

[Java各版本语言特性](https://blog.xianglin.store/posts/java%E5%90%84%E7%89%88%E6%9C%AC%E7%89%B9%E6%80%A7/)

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

[Starter原理解析](https://blog.xianglin.store/posts/starter%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90-springboot%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0/)

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
    * Redisson
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

### 定时任务的使用

[XXL-JOB](https://www.xuxueli.com/xxl-job/)

### Kafka 为什么快

[Kafka 性能篇：为何 Kafka 这么快](https://segmentfault.com/a/1190000039702782)

[Kafka吞吐量大的原因](https://blog.xianglin.store/posts/kafka%E5%9F%BA%E7%A1%80/#kafka%e5%90%9e%e5%90%90%e9%87%8f%e5%a4%a7%e7%9a%84%e5%8e%9f%e5%9b%a0)

### MySQL 死锁

https://blog.csdn.net/qq_43546721/article/details/140141830

https://blog.csdn.net/AlbenXie/article/details/118613349

### 项目中的 JVM 调优

## 面试记录

| 公司            | 时间   | 类型    | 结果            |
|---------------|------|-------|---------------|
| 卓牛科技【互联网】     | 0903 | 两轮    | 职位已关闭         |
| 彩讯股份【互联网】     | 0905 | 两轮    | 已读不回          |
| 物格智能【计算机软件】   | 0913 | 一面    | 已读不回          |
| 科大讯飞【教育BG】    | 0913 | 技术面   | 通过            |
| SHEIN【仓储供应链】  | 0914 | 一面    | 已读不回          |
| 科大讯飞【教育BG】    | 0918 | HR 面  | 未通过           |
| 研途科技【游戏】      | 0920 | 两轮    | 通过            |
| 英特仿真【计算机软件】   | 0920 | 一面    | 未通过           |
| 汇智通信【通信/网络设备】 | 0924 | 两轮    | <s>通过</s>中止录用 |
| 阳普智慧医疗【计算机软件】 | 0924 | 记错时间  | 未参加           |
| 雪宝科技【计算机软件】   | 1010 | HR/老板 | 通过            |
| 雪宝科技【计算机软件】   | 1012 | 技术面   | 已读不回          |
| 如祺出行【慧博云通/外包】 | 1012 | 技术面   | 通过            |
| 先进数通【计算机软件】   | 1015 | 技术面   | 未通过           |
| 广州银行【中电金信/外包】 | 1018 | 技术面   | 已读不回          |
| 易方达【博彦科技/外包】  | 1025 | 一面    | 未通过           |
| 悦信科技【互联网】     | 1202 | 三轮    | 未反馈           |
| 启辰科技【计算机软件】   | 1227 | 技术面   | 通过            |
| 启辰科技【计算机软件】   | 0103 | 老板    | 未反馈           |
| 佳都数据【互联网】     | 0107 | 技术面   | 未反馈           |
| 贤能数科【互联网金融】   | 0115 | 两轮    | 未反馈           |

总结：

1. 这两年基本没进步
2. 有几个好工作也没把握住
3. 一手好牌打得稀烂，工资骤减履历也毁了
4. 切实沉淀两年后再做打算
