---
title: Virtual Box 安装配置 Ubuntu
date: 2022-05-14 16:03:16
tags: [virtual box, ubuntu]
categories: [daily]
description: 使用 VirtualBox 安装 Ubuntu 桌面版，并更换软件源。 
---

# Ubuntu 安装 MySQL

## Ubuntu 安装

推荐从 [USTC](https://mirrors.ustc.edu.cn/) 下载镜像。

### 创建 VirtualBox 虚拟机

点击「新建」，名称输入 `Ubuntu`，类型和版本会自动选择 `Ubuntu(64-bit)`：

![Snipaste_2022-04-27_20-46-07](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213304.png)

![Snipaste_2022-04-27_20-46-38](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213314.png)

![Snipaste_2022-04-27_20-46-45](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213322.png)

使用 Ubuntu 桌面版时，建议调整「系统」栏中的「处理器数量」和「显示」栏中的「显存大小」：

![Snipaste_2022-04-27_20-47-18](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213419.png)

![Snipaste_2022-04-27_20-47-31](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213426.png)

双击启动，选择已下载的 Ubuntu 安装镜像，进入安装界面：

![Snipaste_2022-04-27_20-47-56](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213434.png)

![Snipaste_2022-04-27_20-48-07](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213443.png)

![Snipaste_2022-04-27_20-48-35](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213451.png)

### 安装 Ubuntu

![Snipaste_2022-04-27_20-49-08](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213509.png)

在左边下拉选择「中文（简体）」：

![Snipaste_2022-04-27_20-53-46](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213530.png)

![Snipaste_2022-04-27_20-53-55](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213546.png)

推荐选择「最小安装」：

![Snipaste_2022-04-27_20-54-14](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213622.png)

没有特殊情况不需要自己分区，选择「清除整个磁盘并安装 Ubuntu」即可：

![Snipaste_2022-04-27_20-54-28](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213634.png)

![Snipaste_2022-04-27_20-56-00](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213643.png)

![Snipaste_2022-04-27_20-56-20](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213657.png)

![Snipaste_2022-04-27_20-56-27](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213713.png)

![Snipaste_2022-04-27_21-00-39](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213722.png)

安装完成后，即可进入虚拟机：

![Snipaste_2022-04-27_21-04-31](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213733.png)

![Snipaste_2022-04-27_21-06-02](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213740.png)

## Ubuntu 基础配置

### 更换国内源

可以参考[Ubuntu 源使用帮助](https://mirrors.ustc.edu.cn/help/ubuntu.html)，首先打开「软件和更新」，在「下载自」选择「其它站点」然后在「中国」条目下选择合适的源：

![Snipaste_2022-04-27_21-08-42](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213803.png)

![Snipaste_2022-04-27_21-14-27](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213811.png)

点击「重新载入」刷新本地缓存即可：

![Snipaste_2022-04-27_21-14-51](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213820.png)

### 安装搜狗输入法

下载Ubuntu搜狗输入法：https://pinyin.sogou.com/linux/?r=pinyin

```shell
sudo dpkg -i sogoupinyin_2.3.2.07_amd64-831.deb

# 提示缺少依赖时，使用如下命令
sudo apt install -f
```
