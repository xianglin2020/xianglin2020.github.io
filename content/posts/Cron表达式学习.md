---
title: Cron表达式学习
description: Cron 表达式是一种使用简单字符串来表示时间规则的方式。
date: 2021-12-29 17:26:59
tags: [cron]
categories: [daily]
---

# Cron 表达式学习

## 基本结构

Cron 表达式是一种使用简单字符串来表示时间规则的方式，其基本结构如下：

**`Seconds Minutes Hours DayofMonth Month DayofWeak [Year]`**

其中 `Year` 是可选的，不写代表包含所有年份。

举个例子，每月1 号 0 点：`0 0 0 1 * ?`。

## 各个域的范围

`Seconds、Minutes`：0 ~ 59

`Hours`：0 ~ 23

`DayofMonth`：1 ~ L

`Month`：1 ~ 12 或者 `JAN FEB MAR APR MAY JUN JUL AUG SEP OCT NOV DEC`

`DayofWeak`：1 ~ 7 或者 `SUN MON TUE WED THU FRI SAT`

`Year`：1970 ~ 2099

## 部分特殊字符

`*`：表示匹配任意值，如 `Month` 使用`*`表示每个月都会触发；

`?`：只能用在 `DayofMonth` 和 `DayofWeak` 两个域，因为这两个域相互影响，比如每月 1 号，是这个周的第几天是不确定的，只能用 `?`；

`-`：表示范围，比如在 `Minutes` 中使用 `10-20`，表示从 10 分钟到 20 分钟每分钟触发一次；

`/`：表示从起始时间开始，每隔固定间隔时间触发一次，比如在 `Minutes` 中使用 `5/20` 表示5、25、45分别触发一次；

`,`：表示列出枚举值，比如在 `Minutes` 中使用 `5,20`，表示在第 5 分钟、第 20 分钟分别触发一次；

`L、W、#`：`L` 表示最后，使用在`DayofMonth` 和 `DayofWeak` 两个域，如果在 `DayofWeak` 中使用 `5L` 表示最后一个周四触发；`xW` 表示最接近 `x` 的工作日，只能用在 `DayofMonth` 中，`LW` 表示每个月最后一个工作日；`#` 表示每个月第几个星期几，比如 `2#1` 表示第二个星期日。

## 部分示例

每天凌晨一点触发一次：`0 0 1 * * ? *`

每 30 分钟触发一次：`0 0/30 * * * ?`

每天凌晨 2 ~ 6 点每 20 分钟触发一次：`0 0/20 2-6 * * ?`

工作日 8 点触发一次：`0 0 8 ? * MON-FRI`

每月最后一天 23 点执行一次：`0 0 23 L * ?`

[在线 CRON 表达式工具](https://qqe2.com/cron)



