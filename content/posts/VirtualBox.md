---
title: VirtualBox 的安装和使用
date: 2020-10-04 21:16:32
categories: [daily]
tags: [virtual box]
featuredImagePreview: https://www.virtualbox.org/graphics/button61.png
---

# VirtualBox

## VirtualBox 安装

VirtualBox 安装比较简单，参考其官方说明即可：[Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)。

## VBoxManage 基础命令

### `VBoxManage list`

* 列出已安装的虚拟机

  ```shell
  ➜  ~ VBoxManage list vms
  "fedora" {b9daec39-7e85-485f-969b-629fa52bd549}
  "centos" {49074fd5-386b-42ca-9e25-af29f5e42f1f}
  ```

* 列出正在运行的虚拟机

  ```shell
  ➜  ~ VBoxManage list runningvms
  "centos" {49074fd5-386b-42ca-9e25-af29f5e42f1f}
  ```

### `VBoxManage startvm`

* 命令格式

  ```shell
  ➜  ~ VBoxManage startvm
  Usage:
  
  VBoxManage startvm          <uuid|vmname>...
                              [--type gui|headless|separate]
                              [-E|--putenv <NAME>[=<VALUE>]]
  ```

* 以窗口模式启动虚拟机

  ```shell
  ➜  ~ VBoxManage startvm fedora --type  gui
  Waiting for VM "fedora" to power on...
  VM "fedora" has been successfully started.
  ```

* 以无窗口模式启动虚拟机

  `Starts a VM without a window for remote display only.`

  ```shell
  ➜  ~ VBoxManage startvm centos --type headless
  ```

### `VBoxManage controlvm`

* 命令格式

  ```shell
  ➜  ~ VBoxManage controlvm
  Usage:
  
  VBoxManage controlvm        <uuid|vmname>
                              pause|resume|reset|poweroff|savestate|
                              acpipowerbutton|acpisleepbutton|
  ```

* 暂停虚拟机

  ```shell
  ➜  ~ VBoxManage controlvm fedora pause
  ```

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/114348.png" alt="image-20201205114348345" style="zoom:50%;" />

* 恢复已暂停的虚拟机

  ```shell
  ➜  ~ VBoxManage controlvm fedora resume
  ```

* 关闭虚拟机：相当于拉闸断电

  ```shell
  ➜  ~ VBoxManage controlvm fedora poweroff
  0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
  ```

* 关闭虚拟机：正常关闭

  ```shell
  ➜  ~ VBoxManage controlvm fedora acpipowerbutton
  ```

* 休眠虚拟机

  ```shell
  ➜  ~ VBoxManage controlvm fedora savestate
  0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
  ```

## `VBoxManage guestproperty`

* 获取或者设置正在运行的虚拟机的属性

* 命令格式

  ```shell
  ➜  ~ VBoxManage guestproperty
  Usage:
  
  VBoxManage guestproperty    get <uuid|vmname>
                              <property> [--verbose]
  
  VBoxManage guestproperty    set <uuid|vmname>
                              <property> [<value> [--flags <flags>]]
  
  VBoxManage guestproperty    delete|unset <uuid|vmname>
                              <property>
  
  VBoxManage guestproperty    enumerate <uuid|vmname>
                              [--patterns <patterns>]
  
  VBoxManage guestproperty    wait <uuid|vmname> <patterns>
                              [--timeout <msec>] [--fail-on-timeout]
  ```

* 获取当前运行虚拟机的 IP 地址

  ```shell
  ➜  ~ VBoxManage guestproperty enumerate centos | grep "Net.*V4.*IP"
  Name: /VirtualBox/GuestInfo/Net/0/V4/IP, value: 10.0.2.15, timestamp: 1607138767607688000, flags:
  Name: /VirtualBox/GuestInfo/Net/1/V4/IP, value: 192.168.56.106, timestamp: 1607138767607908000, flags:
  
  ➜  ~ VBoxManage guestproperty get centos '/VirtualBox/GuestInfo/Net/1/V4/IP'
  Value: 192.168.56.106
  ```







