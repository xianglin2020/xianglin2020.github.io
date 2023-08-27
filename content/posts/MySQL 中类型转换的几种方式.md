---
title: MySQL 中类型转换的几种方式
date: 2022-02-11 18:53:36
tags: [MySQL]
categories: [daily]
summary: MySQL 中类型转换的几种方式
description: MySQL 中类型转换的几种方式
---

# MySQL 中类型转换的几种方式

### 表达式中的隐式转换

```mysql
# 在数学运算中隐式转换
select '1', '1' + 0; # double
```

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202202/124318.png" alt="image-20220212104318664" style="zoom:50%;" />

### 使用 `CAST()` 函数

`CAST`函数签名为：`CAST(ANY:any AS datatype:cast_datatype):TypeOf(2)`，作用是将任何类型的值转换为具有指定类型的值。目标类型可以是：`BINARY`、`CHAR`、`DATE`、`DATETIME`、`TIME`、`DECIMAL`、`SIGNED`、`UNSIGNED`。

```mysql
# 使用 CAST 函数
select cast('5' AS BINARY); # varbinary
select cast('5' AS CHAR); # varchar(1)
select cast('2022-02-12' AS DATE), cast(20220212 AS DATE); # 2022-02-12 date(yyyy-MM-dd)
select cast('2022-02-12' AS DATETIME); # 2022-02-12 00:00:00 datetime(yyyy-MM-dd HH:mm:ss)
select cast('111500' AS TIME), cast('11:15:00' AS TIME); # 11:15:00 time(HH:mm:ss)
select cast('5.45' AS DECIMAL), cast('5.45' AS DECIMAL(10, 3)); # decimal(10, 0)
select cast('5' AS SIGNED), cast('5' AS UNSIGNED); # bigint
```

需要注意的是：

* `DECIMAL(M, D)` 默认 `M` 为 10、`D` 为 0；
* `DATE`、`DATETIME`、`TIME` 类型的表达式不符合格式要求时返回 `NULL`。

### 使用 `CONVERT()` 函数

`CONVERT` 函数签名有两种：

* `CONVERT(str:varchar, datatype:cast_datatype):TypeOf(2)`，作用是将字符串转换为具有指定类型的值，此时和 `CAST()` 函数的目标类型和使用方式基本一致；
* `CONVERT(str:varchar USING ref:reference/CHARSET):varchar`，用于字符串在不同的字符集间转换。

```mysql
# 使用 CONVERT 函数
select convert(20220212, DATE);
select convert(_latin1'Müller' USING utf8);
```



