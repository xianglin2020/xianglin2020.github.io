---
title: "VirtualBox_Debian12_修复引导"
date: 2024-11-10T11:35:53+08:00
categories:
  - daily
tags:
  - virtual box
  - linux
summary: "使用 live cd 修复 Debian12 引导。"
description: ""
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## 虚拟机迁移

需要将 Debian 虚拟机从一台主机迁移到另一台主机上。使用 VirtualBox 自带的【导出虚拟电脑】和【导入虚拟电脑】功能即可完成。

## 启动失败

在新主机上导入 Debian 虚拟机启动时，提示启动失败。

![image-20241110120333382](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202411101203420.png)

经查询，需要修复Debian的引导记录。

[记一次修复 Debian 12 Grub 莫名奇妙消失的经历](https://blog.lzc256.com/posts/debian-12-grub-rescue/)

 [GrubEFIReinstall](https://wiki.debian.org/GrubEFIReinstall)

## 修复虚拟机引导

下载 [Debian Live CD](https://mirrors.ustc.edu.cn/)。

![image-20241110120811833](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202411101208857.png)

虚拟机启动时选择下载的 Live CD。

![image-20241110120847016](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202411101208036.png)

进入 Live CD 后按步骤操作。

```bash
sudo -i
# 查看分区，我的 EFI 分区是 /dev/sda1，根分区是 /dev/sda2
fdisk -l
# 挂载分区
mount /dev/sda2 /mnt
mount /dev/sda1 /mnt/boot/efi
for i in /dev /dev/pts /proc /sys /sys/firmware/efi/efivars /run; do mount -B $i /mnt/$i; done
# 进入挂载的 Debian 系统
chroot /mnt
# 执行修复命令
grub-install /dev/sda
update-grub
# 退出 chroot，重启
<CTRL>-D
reboot
```

![image-20241110122011343](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202411101220373.png)
