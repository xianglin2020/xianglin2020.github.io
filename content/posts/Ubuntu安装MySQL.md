---
title: Ubuntu 安装 MySQL
date: 2020-10-10 22:44:19
categories: [daily]
tags: [ubuntu, mysql]
featuredImagePreview: https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212642.png
---

# Ubuntu 安装 MySQL

## Ubuntu Server 和 MySQL

* [Ubuntu Server](https://ubuntu.com/download/alternative-downloads)
* [MySQL Server](https://downloads.mysql.com/archives/community/)

### 使用 APT 安装 MySQL

1. 使用命令安装 MySQL Server

   `sudo apt install mysql-server`

2. 查看 MySQL 运行情况

   `sudo service mysql status`

   或者

   `sudo /etc/init.d/mysql status`

   ```shell
   sudo /etc/init.d/mysql status
   ● mysql.service - MySQL Community Server
        Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
        Active: active (running) since Thu 2020-10-08 22:41:04 CST; 2 days ago
      Main PID: 55128 (mysqld)
        Status: "Server is operational"
         Tasks: 40 (limit: 2319)
        Memory: 332.3M
        CGroup: /system.slice/mysql.service
                └─55128 /usr/sbin/mysqld
   
   Oct 08 22:41:03 iZwz92js0u6m08arfj6jp8Z systemd[1]: Starting MySQL Community…...
   Oct 08 22:41:04 iZwz92js0u6m08arfj6jp8Z systemd[1]: Started MySQL Community …er.
   Hint: Some lines were ellipsized, use -l to show in full.
   ```

   启动、停止、重启MySQL 的命令类似

   ```shell
   sudo [/etc/init.d/mysql | service mysql] [force-reload|restart|status|reload|start|stop]
   ```

   <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/230111.png" alt="image-20201010230111324" style="zoom:50%;" /><img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/230138.png" alt="image-20201010230138405" style="zoom:50%;" />

3. 在 Ubuntu 中安装 MySQL 并没有提示设置 root 密码，但可以在 `/etc/mysql` 路径下找到相关配置文件

   ```shell
   cd /etc/mysql
   sudo cat debian.cnf
   ```

   会得到如下信息：

   ![image-20201010225727578](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/225728.png)

   可以使用`debain.cnf`中的 `user` 和 `password` 登录 MySQL

   ![image-20201010225905777](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/000743.png)
   
4. 新建一个用于远程连接的用户，并分配权限

   新建用户

   `create user 'xianglin'@'119.130.215.117' identified by '950915';`

   分配权限

   `grant all privileges on *.* to 'xianglin'@'119.130.215.117';`

   `flush privileges;`

   ![image-20201010231312744](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/231313.png)

5. 找到 `/etc/mysql/mysql.conf.d/mysqld.cnf` 并注释 `bind-address=127.0.01`，后重启 MySQL 服务

6. 在阿里云的实例列表中找到如下菜单

   ![image-20201011001243793](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/000622.png)

   添加访问规则，允许 MySQL 默认端口 3306 访问

   ![image-20201011001500697](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/001501.png)

7. 使用 MySQL 客户端工具 datagrid 连接 MySQL

   ![image-20201011001532104](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/000652.png)

   
