---
title: Docker基础
description: Docker 是一个用来装程序及其环境的容器，属于Linux 容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。
date: 2020-12-01 22:38:26
categories: [learn]
tags: [docker]
featuredImagePreview: https://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/Docker_%28container_engine%29_logo.svg/220px-Docker_%28container_engine%29_logo.svg.png
---

# Docker

## Docker 基础概念

Docker 是一个用来装程序及其环境的容器，属于Linux 容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。

### Docker 的用途

* 提供统一的环境
* 提供快速拓展、弹性伸缩的云服务
* 防止其他用户的进程把服务器资源占用过多

### Docker 的特点

* 标准化
  1. 运输方式：把程序和环境从一个机器运到另一个机器
  2. 存储方法：程序和环境的存储
  3. API 接口
* 灵活：即使是最复杂的应用也可以集装箱化
* 轻量级：容器利用并共享主机内核
* 便携式：可以在本地构造，部署到云，并可以在任何地方运行

## Docker 组成

![image-20201202214527746](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/214528.png)

## 镜像

![image-20201202214925288](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/214925.png)

![image-20201202215018108](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/215018.png)

镜像存储：联合文件系统，UnionFS



## 容器

* 镜像类似于 Java 中的类，而容器类似于实例
* 容器层可以修改，镜像是不可以修改的
* 同一个镜像可以生成多个独立的容器，容器之间互不干扰

## 仓库

* hub.docker.com
* 私有、共有仓库

## Client、Deamon

* client 提供给用户一个终端，用户输入 Docker 提供的命令来管理本地或远程的服务器
* deamon：服务端守护进程，接收 client 发送的命令并执行相应的操作

### Docker 的网络模式

* Bridge
* Host
* None

## Docker 安装

### Docker 在 CentOS7 的安装

* 查看 CentOS版本

  ```shell
  [root@localhost ~]# cat /etc/redhat-release
  CentOS Linux release 7.9.2009 (Core)
  ```

* 设置阿里云的 yum 源

  ```shell
  [root@localhost ~]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
  --2020-12-02 22:36:00--  http://mirrors.aliyun.com/repo/Centos-7.repo
  正在解析主机 mirrors.aliyun.com (mirrors.aliyun.com)... 157.148.73.246, 157.148.73.241, 157.148.73.242, ...
  正在连接 mirrors.aliyun.com (mirrors.aliyun.com)|157.148.73.246|:80... 已连接。
  已发出 HTTP 请求，正在等待回应... 200 OK
  长度：2523 (2.5K) [application/octet-stream]
  正在保存至: “/etc/yum.repos.d/CentOS-Base.repo”
  
  100%[======================================>] 2,523       --.-K/s 用时 0s
  
  2020-12-02 22:36:00 (499 MB/s) - 已保存 “/etc/yum.repos.d/CentOS-Base.repo” [2523/2523])
  ```

* 更新 yum

  ```shell
  [root@localhost ~]# yum clean all
  已加载插件：fastestmirror
  正在清理软件源： base docker-ce-stable extras updates
  Cleaning up list of fastest mirrors
  
  [root@localhost ~]# yum makecache
  已加载插件：fastestmirror
  Determining fastest mirrors
   * base: mirrors.aliyun.com
   * extras: mirrors.aliyun.com
   * updates: mirrors.aliyun.com
  base                                                     | 3.6 kB     00:00
  docker-ce-stable                                         | 3.5 kB     00:00
  extras                                                   | 2.9 kB     00:00
  updates                                                  | 2.9 kB     00:00
  (1/14): base/7/x86_64/group_gz                             | 153 kB   00:00
  (2/14): base/7/x86_64/filelists_db                         | 7.2 MB   00:00
  # .....
  (14/14): updates/7/x86_64/other_db                         | 225 kB   00:00
  元数据缓存已建立
  
  [root@localhost ~]# yum check-update
  
  [root@localhost ~]# yum update
  ```

* 安装所需的软件包

  ```shell
  [root@localhost ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
  ```

* 为 Docker 指定稳定的存储库

  ```shell
  [root@localhost ~]# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  已加载插件：fastestmirror
  adding repo from: http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  grabbing file http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
  repo saved to /etc/yum.repos.d/docker-ce.repo
  ```

* 查看 Docker 版本

  ```shell
  [root@localhost ~]# yum list docker-ce --showduplicates | sort -r
  ```

* 安装（可指定版本）Docker

  ```shell
  [root@localhost ~]# yum install docker-ce docker-ce-cli containerd.io
  ```

* 启动 Docker

  ```shell
  [root@localhost ~]# systemctl start docker
  
  # 设置开机自启
  [root@localhost ~]# systemctl enable docker
  ```

* 验证是否安装成功

  ```shell
  [root@localhost ~]# docker version
  [root@localhost ~]# docker info
  ```

### 运行第一个 Docker 容器

* 下载镜像

  `docker pull [OPTIONS] NAME[:TAG|@DIGEST]`

  ```shell
  [root@localhost ~]# docker pull hello-world
  Using default tag: latest
  latest: Pulling from library/hello-world
  0e03bdcc26d7: Pull complete
  Digest: sha256:e7c70bb24b462baa86c102610182e3efcb12a04854e8c582838d92970a09f323
  Status: Downloaded newer image for hello-world:latest
  docker.io/library/hello-world:latest
  ```

* 查看镜像

  `docker images [OPTIONS] [REPOSITORY[:TAG]]`
  
  ```shell
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  hello-world         latest              bf756fb1ae65        11 months ago       13.3kB
  ```
  
* 运行镜像

  `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

  ```shell
  [root@localhost ~]# docker run hello-world
  
  Hello from Docker!
  This message shows that your installation appears to be working correctly.
  
  To generate this message, Docker took the following steps:
   1. The Docker client contacted the Docker daemon.
   2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
      (amd64)
   3. The Docker daemon created a new container from that image which runs the
      executable that produces the output you are currently reading.
   4. The Docker daemon streamed that output to the Docker client, which sent it
      to your terminal.
  
  To try something more ambitious, you can run an Ubuntu container with:
   $ docker run -it ubuntu bash
  
  Share images, automate workflows, and more with a free Docker ID:
   https://hub.docker.com/
  
  For more examples and ideas, visit:
   https://docs.docker.com/get-started/
  ```

### 运行 Nginx 容器

* 拉取 Nginx 镜像

  ```shell
  [root@localhost ~]# docker pull hub.c.163.com/library/nginx:latest
  latest: Pulling from library/nginx
  5de4b4d551f8: Pull complete
  d4b36a5e9443: Pull complete
  0af1f0713557: Pull complete
  Digest: sha256:f84932f738583e0169f94af9b2d5201be2dbacc1578de73b09a6dfaaa07801d6
  Status: Downloaded newer image for hub.c.163.com/library/nginx:latest
  hub.c.163.com/library/nginx:latest
  ```

* 运行 Nginx 容器

  ```shell
  [root@localhost ~]# docker run -d hub.c.163.com/library/nginx:latest
  0708d1119dfedcd40371935d901fc460682d6eb8cc0177b510b47eaa8ffeb506
  ```

* 登入容器

  ```shell
  [root@localhost ~]# docker exec -it 0708 bash
  root@0708d1119dfe:/#
  
  
  root@0708d1119dfe:/# which nginx
  /usr/sbin/nginx
  ```

* 停止容器

  `docker stop [OPTIONS] CONTAINER [CONTAINER...]`

  ```shell
  [root@localhost ~]# docker stop 0708
  0708
  ```

* 使用端口映射

  ```shell
  [root@localhost ~]# docker run -d -p 8080:80 hub.c.163.com/library/nginx:latest
  8dc5cef54fcf00dc4f5e428d1de14c8efe6c867b9e2f4870078d99bf3c0e0cef
  ```

  `-p, --publish list 	Publish a container's port(s) to the host
   -P, --publish-all		Publish all exposed ports to random ports`

* 查看进行端口映射的容器信息

  ```shell
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE                                COMMAND                  CREATED              STATUS              PORTS                  NAMES
  8dc5cef54fcf        hub.c.163.com/library/nginx:latest   "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp   dreamy_ritchie
  ```

  ```shell
  [root@localhost ~]# netstat -na| grep 8080
  tcp6       0      0 :::8080                 :::*                    LISTEN
  ```

* 在浏览器访问 Nginx

  ![image-20201202232056733](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/232056.png)

## Dockerfile

### 简单的 Dockerfile

* 编写 Dockerfile

  ```dockerfile
  FROM alpine:latest
  MAINTAINER xianglin
  CMD echo 'hello my dockerfile'
  ```

* 构建镜像

  ```shell
  [root@localhost ~]# docker build -t hello-docker .
  Sending build context to Docker daemon  15.36kB
  Step 1/3 : FROM alpine:latest
   ---> d6e46aa2470d
  Step 2/3 : MAINTAINER xianglin
   ---> Using cache
   ---> 5b2571b3eae2
  Step 3/3 : CMD echo 'hello my dockerfile'
   ---> Using cache
   ---> 465286d0513b
  Successfully built 465286d0513b
  Successfully tagged hello-docker:latest
  ```

* 查看镜像、运行容器

  ```shell
  [root@localhost ~]# docker images
  REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
  hello-docker                  latest              465286d0513b        41 seconds ago      5.57MB
  alpine                        latest              d6e46aa2470d        5 weeks ago         5.57MB
  hello-world                   latest              bf756fb1ae65        11 months ago       13.3kB
  hub.c.163.com/library/nginx   latest              46102226f2fd        3 years ago         109MB
  # hello-docker 基于 alpine 构建
  [root@localhost ~]# docker run hello-docker
  hello my dockerfile
  ```


## 常用软件工具的 docker 命令

https://hub.docker.com/

* 在 Docker 实例中安装软件包

  ```shell
  # 登入容器
  docker exec -it 7ec9e6cf98a9 bash
  # 替换源
  sed -i s@/deb.debian.org/@/mirrors.163.com/@g /etc/apt/sources.list
  # 更新源、安装软件
  apt update
  apt install vim
  ```

* [MySQL](https://hub.docker.com/_/mysql)

  ```shell
  docker run --name mysql-dev -e MYSQL_ROOT_PASSWORD=12345678 -e MYSQL_DATABASE=dev -e MYSQL_USER=dev -e MYSQL_PASSWORD=password -p 3306:3306 -d mysql:latest
  ```

* [禅道](https://hub.docker.com/r/idoop/zentao)

  ```shell
  mkdir -p /data/zbox && \
  docker run -d -p 80:80 -p 3306:3306 \
          -e ADMINER_USER="root" -e ADMINER_PASSWD="password" \
          -e BIND_ADDRESS="false" \
          -v /data/zbox/:/opt/zbox/ \
          --name zentao-server \
          idoop/zentao:latest
  ```
  
* [Jenkins](https://hub.docker.com/r/jenkins/jenkins)

  ```shell
  mkdir -p /data/jenkins && \
  docker run -d -p 8080:8080 -p 50000:50000 \ 
  				-v /data/jenkins:/var/jenkins_home \ 
  				--restart always \ 
  				jenkins/jenkins:lts-jdk11
  ```

* [GitLab](https://hub.docker.com/r/gitlab/gitlab-ce)

  [中文教程](https://docs.gitlab.cn/jh/install/docker.html)

  ```shell
  docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 80:80 --publish 22:22 \
    --name gitlab \
    --restart always \
    --volume /data/gitlab/config:/etc/gitlab \
    --volume /data/gitlab/logs:/var/log/gitlab \
    --volume /data/gitlab/data:/var/opt/gitlab \
    --shm-size 256m \
    gitlab/gitlab-ee:latest
  ```

* [MongoDB](https://hub.docker.com/_/mongo)

  参见：[Docker版MongoDB的安装](https://www.jianshu.com/p/2181b2e27021)

  ```bash
  mkdir -p .docker/data/mongodb && \
  docker run -d -p 27017:27017 -v ~/.docker/data/mongodb:/data/db -e MONGO_INITDB_ROOT_PASSWORD=mongomongo -e MONGO_INITDB_ROOT_USERNAME=mongo --name mongo mongo 
  
  # 进入容器中的 MongoDB 命令行
  docker exec -it mongo mongo admin
  # 创建管理员账户
  db.createUser({ user: 'mongo', pwd: 'mongomongo', roles: [ { role: "root", db: "admin" } ] });
  ```
  
* [Redis](https://hub.docker.com/_/redis/)

  ```bash
  docker run --name some-redis -d -p 6379:6379 redis 
  ```

  
