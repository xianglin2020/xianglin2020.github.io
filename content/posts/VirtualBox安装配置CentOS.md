---
title: Virtual Box 安装配置 CentOS
date: 2022-05-10 22:14:44
tags: [virtual box, centos]
categories: [daily]
description: 使用 VirtualBox 安装 CentOS 服务器版，并安装增强功能。
summary: 使用 VirtualBox 安装 CentOS 服务器版，并安装增强功能。 
---

# Virtual Box 安装配置 CentOS7

## 下载安装

### 下载 CentOS7 镜像

通过[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/#)，下载 CentOS7 的安装镜像，如图所示：

![image-20220514143242131](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143242.png)

### 安装 CentOS7

在 VirtualBox 中创建虚拟机的过程比较简单，安装提示即可，如下图所示：

![image-20220514143503439](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143503.png)

![image-20220514143531487](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143531.png)

![image-20220514143559828](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143559.png)

![image-20220514143658028](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143658.png)

![image-20220514143712645](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143712.png)

![image-20220514143806993](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143807.png)

![image-20220514143906528](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143906.png)

进入安装界面后，首先需要选择安装过程中使用的语言：

![image-20220514144210010](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/144210.png)

在「安装信息摘要」中，「本地化」和「软件」两栏的内容保持不变：

![image-20220514144500930](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/144501.png)

在「安装位置」中选中虚拟磁盘：

![image-20220514144727906](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/144727.png)

在「网络和主机名」中打开网络：

![image-20220514145134825](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/145134.png)

设置完成后点击「开始安装」：

![image-20220514145448098](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/145448.png)

安装过程中需要设置「ROOT密码」，并按需创建用户：

![image-20220514145551250](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/145551.png)

如果是”弱密码“，需要点击两次「完成」按钮：

![image-20220514145657570](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/145657.png)

等待 CentOS7 安装完成即可：

![image-20220514145928372](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/145928.png)

## 基础配置

### 配置 `Host-Only` 网络并指定静态 IP

虚拟机内 CentOS 的默认终端很不好用，一般都是通过 SSH 从宿主机连接使用，这时可以开启 `Host-Only` 网络并指定一个静态 IP。

#### 为 CentOS7 增加 `Host-Only` 网络配置

在关机状态下，打开虚拟机设置，在「网络」栏中新增网卡：

![image-20220514150440632](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/150440.png)

如果在「界面名称」中只有「未指定」一个选项，并且下方有“发现无效设置”时，需要先添加网络配置：

![image-20220514150555641](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/150555.png)

在「管理」—「主机网络管理器」中新增一条 `Host-Only` 网络配置：

![image-20220514150846268](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/150846.png)

![image-20220514150928680](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/150928.png)

完成后，再到虚拟机「设置」—「网络」中选择即可：

![image-20220514151019716](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/151019.png)

设置完成后，进入虚拟机，使用 `ip addr` 命令查看当前网络信息，可以发现已存在 `Host-Only` 网络：

![image-20220514151218998](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/151219.png)

#### 为新增的 `Host-Only` 网络配置静态 IP

CentOS7 的网络配置文件位于 `/etc/sysconfig/network-scripts`，并与设备名称同名。即需要编辑 `/etc/sysconfig/network-scripts/ifcfg-enp0s8` 文件：

```properties
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
# 设置为静态 IP
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s8
UUID=76b90704-1ecd-4c8e-a411-9c776cb7834d
DEVICE=enp0s8
# 设置开启启动
ONBOOT=yes

# 添加静态 IP 项
IPADDR=192.168.56.106
NETMASK=255.255.255.0
GATEWAY=192.168.56.1
```

同时可以在 `/etc/sysconfig/network` 中添加 DNS 配置：

```properties
# Created by anaconda
DNS1=192.168.56.1
DNS2=8.8.8.8
```

最后重启网络配置即可：

```bash
service network restart
```

再次使用 `ip addr` 命令查看网络信息，发现配置已经生效：

![image-20220514152400747](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/152400.png)

### 为 CentOS7 安装增强功能

CentOS Server 无法从「设备」—「安装增强功能」菜单为其安装增强功能，可以按照以下步骤手动安装。

安装依赖包：

```shell
[root@localhost ~]# yum install -y gcc gcc-c++ make kernel kernel-headers kernel-devel bzip2 tar
[root@localhost ~]# reboot
```

从菜单中挂载 `VBoxGuestAdditions.iso` ：

![image-20220514152808852](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/152808.png)

使用如下命令挂载光驱：

```shell
# 创建挂载点
[root@localhost ~]# mkdir /media/cdrom
# 挂载光驱
[root@localhost ~]# mount /dev/sr0 /media/cdrom
mount: /dev/sr0 写保护，将以只读方式挂载

[root@localhost ~]# ll /media/cdrom/
总用量 46826
-r--r--r--. 1 root root      763 2月  20 2020 AUTORUN.INF
-r-xr-xr-x. 1 root root     6384 10月 15 22:42 autorun.sh
dr-xr-xr-x. 2 root root      792 10月 15 22:48 cert
dr-xr-xr-x. 2 root root     1824 10月 15 22:48 NT3x
dr-xr-xr-x. 2 root root     2652 10月 15 22:48 OS2
-r-xr-xr-x. 1 root root     4821 10月 15 22:42 runasroot.sh
-r--r--r--. 1 root root      547 10月 15 22:48 TRANS.TBL
-r--r--r--. 1 root root  3830063 10月 15 22:41 VBoxDarwinAdditions.pkg
-r-xr-xr-x. 1 root root     3949 10月 15 22:41 VBoxDarwinAdditionsUninstall.tool
-r-xr-xr-x. 1 root root  7413172 10月 15 22:42 VBoxLinuxAdditions.run
-r--r--r--. 1 root root  9401856 10月 15 23:41 VBoxSolarisAdditions.pkg
-r-xr-xr-x. 1 root root 16950792 10月 15 22:45 VBoxWindowsAdditions-amd64.exe
-r-xr-xr-x. 1 root root   270616 10月 15 22:42 VBoxWindowsAdditions.exe
-r-xr-xr-x. 1 root root 10057608 10月 15 22:43 VBoxWindowsAdditions-x86.exe
```

![image-20220514152949269](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/152949.png)

执行安装命令，安装完成后重启即可：

```shell
[root@localhost cdrom]# sh ./VBoxLinuxAdditions.run
Verifying archive integrity... All good.
Uncompressing VirtualBox 6.1.16 Guest Additions for Linux........
VirtualBox Guest Additions installer
Removing installed version 6.1.16 of VirtualBox Guest Additions...
Copying additional installer modules ...
Installing additional modules ...
VirtualBox Guest Additions: Starting.
VirtualBox Guest Additions: Building the VirtualBox Guest Additions kernel
modules.  This may take a while.
VirtualBox Guest Additions: To build modules for other installed kernels, run
VirtualBox Guest Additions:   /sbin/rcvboxadd quicksetup <version>
VirtualBox Guest Additions: or
VirtualBox Guest Additions:   /sbin/rcvboxadd quicksetup all
VirtualBox Guest Additions: Building the modules for kernel
3.10.0-1160.6.1.el7.x86_64.
VirtualBox Guest Additions: Running kernel modules will not be replaced until
the system is restarted

[root@localhost ~]# reboot
```

使用 「共享文件夹」验证增强功能安装成功，在「共享文件夹」中新增一条配置，将宿主机的下载目录挂载到虚拟机的 `Downloads` 下，然后进入虚拟机查看到对应路径存在，代表安装成功：

![image-20220514155854979](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/155855.png)

![image-20220514160045458](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/160045.png)
