---
title: VirtualBox_install_Win11
description: 使用 VirtualBox 安装 Windows11。
summary: 使用 VirtualBox 安装 Windows11。
date: 2021-11-17 14:46:04
categories: [daily]
tags: [virtual box]
cover:
  image: "https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202111/290945.jpeg"
  caption: ""
  alt: ""
  relative: false
---

# 使用 VirtualBox 安装 Windows11

## 链接

1. Window11 下载链接：[itellyou](https://next.itellyou.cn/Original/Index#cbp=Product?ID=42e87ac8-9cd6-eb11-bdf8-e0d4e850c9c6)
2. VirtualBox 下载链接：[virtualbox](https://www.virtualbox.org/wiki/Downloads)
3. [How to install Microsoft Windows 11 on VirtualBox!](https://blogs.oracle.com/virtualization/post/install-microsoft-windows-11-on-virtualbox)

## 安装步骤

### 推荐设置

* 系统

  ![image-20211129093107126](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202111/290931.png)

  ![image-20211129093128343](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202111/290947.png)

* 显示

  ![image-20211129093022869](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202111/290930.png)

* 存储

  ![image-20211129093201147](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202111/290932.png)

### 安装设置

1. 在安装界面，使用`Shift+F10`调出控制台，在控制台中输入`regedit`调出注册表编辑器；

2. 在左侧依次找到如下路径：`HKEY_LOCAL_MACHINE\SYSTEM\Setup`；

3. 在`Setup`中新增配置项`LabConfig`，新建三个**`DWORD（32-位）值`**，值都设置为 `1`：

   ![image-20211129094157895](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202111/290942.png)

   1. **`BypassTPMCheck`**
   2. **`BypassRAMCheck`**
   3. **`BypassSecureBootCheck`**

   结果如下：

   ![Windows 11 - bypass checks](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202111/290942.jpeg)

4. 完成后关闭注册表和控制台，即可正常安装。
