---
title: Docker 安装 Oracle-11g
date: 2021-04-06 10:18:12
tags: [docker]
categories: [learn]
---

# Docker 安装 Oracle-11g

## 参考链接

* DockerHub：[jaspeen/oracle-11g (docker.com)](https://hub.docker.com/r/jaspeen/oracle-11g)
* Oracle Database：[适用于 Linux x86-64 的 Oracle Database 11g 第 2 版 | Oracle 中国](https://www.oracle.com/cn/database/enterprise-edition/downloads/oracle-db11g-linux.html)
* Docker CE 清华源：[docker-ce | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)

## 安装步骤

### Docker 安装步骤

1. 删除旧版 Docker

   ```shell
   sudo apt-get remove docker docker-engine docker.io
   ```

2. 安装依赖

   ```shell
   sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common
   ```

3. 导入 Docker GPG 公钥

   ```shell
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

4. 添加软件仓库

   ```shell
   sudo add-apt-repository \
      "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
   ```

5. 安装 Docker

   ```shell
   sudo apt-get update
   sudo apt-get install docker-ce
   ```

6. 替换国内源，编辑`/etc/docker/daemon.json`

   ```json
   {
     "registry-mirrors": [
       "http://registry.docker-cn.com",
       "http://docker.mirrors.ustc.edu.cn",
       "http://hub-mirror.c.163.com"
     ],
     "insecure-registries": [
       "registry.docker-cn.com",
       "docker.mirrors.ustc.edu.cn"
     ],
     "debug": true,
     "experimental": true
   }
   ```

7. 将当前用户加入 docker 组

   ```shell
   sudo gpasswd -a username docker
   newgrp docker
   ```

### Oracle-11g 安装步骤

1. 下载 `linux.x64_11gR2_database_1of2.zip`、`linux.x64_11gR2_database_2of2.zip`两个文件，将其解压。

   ![image-20210406103137806](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/20210406103144.png)

   ![image-20210406103230234](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/20210406103230.png)

2. 拉取 Oracle-11g 的 docker 镜像

   ```shell
   docker pull jaspeen/oracle-11g
   ```

3. 安装启动 Oracle Database

   ```shell
   docker run --privileged --name oracle11g -p 1521:1521 -v /home/xianglin/oracle:/install jaspeen/oracle-11g
   ```

   安装过程较长，查看输出，有 `100% complete` 时，表示安装成功。

   ![image-20210406103420985](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/20210406103421.png)

### 解除账户限制

1. 连接到容器

   ```shell
   docker exec -it oracle11g /bin/bash
   ```

2. 切换到 oracle 用户，然后链接到 sql 控制台

   ```shell
   [root@9b255eb5dd63 /]# su - oracle
   Last login: Tue Apr  6 02:14:23 UTC 2021 on pts/0
   [oracle@9b255eb5dd63 ~]$ sqlplus / as sysdba
   
   SQL*Plus: Release 11.2.0.1.0 Production on Tue Apr 6 02:38:35 2021
   
   Copyright (c) 1982, 2009, Oracle.  All rights reserved.
   
   
   Connected to:
   Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
   With the Partitioning, OLAP, Data Mining and Real Application Testing options
   
   SQL>
   ```

3. 解锁账户

   ```shell
   SQL> alter user scott account unlock;
   
   User altered.
   
   SQL> conn scott/tiger
   ERROR:
   ORA-28001: the password has expired
   
   
   Changing password for scott
   New password:
   Retype new password:
   Password changed
   Connected.
   SQL>
   ```

4. 使用可视化工具登录

   ![image-20210406104925250](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/20210406104925.png)

