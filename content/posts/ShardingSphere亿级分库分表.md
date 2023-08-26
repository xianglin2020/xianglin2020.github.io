---
title: ShardingSphere亿级分库分表
date: 2023-06-12 19:56:23
tags: [MySQL]
categories: [learn]
---

# 数据初始化

## 实验环境

`Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz`

`5.15.0-73-generic #80~20.04.1-Ubuntu`

`mysql  Ver 8.0.33-0ubuntu0.20.04.2 for Linux on x86_64 ((Ubuntu))`

`openjdk version "19.0.2" 2023-01-17`

## 初始化

`post` 表结构见 `post.sql`，初始化程序见 `gen.GenExecutor`。

初始磁盘使用空间 `18G`。

![image-20230612202601520](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021650748.png)

数据初始化完成后磁盘使用空间 `76G`。

![image-20230613202754885](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021650749.png)

写入 `post` 表记录总数 `100000000` 条。

![image-20230613214549378](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021650750.png)

## 测试查询

从 1000000 开始取 2 条记录，耗时 `1.66s`。

![image-20230615195326122](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021650751.png)

查询 `user_id` 为 `7009336` 的记录，耗时 `2'43s`。

```mysql
select * from post where user_id = 7009336;
```

![image-20230615195632151](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021650752.png)

## 建立索引

尝试在 `user_id` 上创建索引，耗时 `12'55s`。

```mysql
create index `idx_user_id` on post(`user_id`);
```

![image-20230615201253102](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021650753.png)

再次查询 `user_id` 为 `7009336` 的记录，耗时降低到毫秒级。

![image-20230615201426583](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021650754.png)

查看磁盘空间已使用 `78G`，索引大致占 `2G`。

![image-20230615201517152](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021650755.png)

# 垂直拆分

## 使用 `sql` 复制数据

`post` 表中的 `info` 为 `text` 类型的字段，将其拆分为 `post_info` 表，将 `post` 表的数据复制到 `post_info` 中，语句见 `post_info.sql`。

使用 `iostat` 工具查看磁盘读写速度。

```bash
# 监控 nvme0n1，间隔 1s，打印 1000 次
iostat nvme0n1 1 1000
```

![image-20230615202813237](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021650756.png)

## 多线程复制

直接使用上述语句因为是单个事务操作，耗时过长，通过 Java 程序将其改为多线程复制，每次复制 `1000000` 条数据，磁盘的读写速度将近提升一倍，代码见 `migration.CopyData`。

![image-20230618155040348](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021650757.png)
