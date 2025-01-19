---
title: 使用 Docker 安装GitLab、Jenkins 配置自动化构建
date: 2022-04-16 17:20:05
tags: [docker]
categories: [daily]
description: 使用 Docker 安装配置 GitLab，搭建代码托管平台，安装配置 Jenkins，实现自动化构建。
summary: 使用 Docker 安装配置 GitLab，搭建代码托管平台，安装配置 Jenkins，实现自动化构建。
---

# 使用 Docker 配置自动化构建

## 安装 GitLab

官方文档中文：https://docs.gitlab.cn/jh/install/docker.html

官方文档英文：https://docs.gitlab.com/ee/install/docker.html

GitLab 镜像地址：https://hub.docker.com/r/gitlab/gitlab-ce

### 创建目录存储持久数据

```bash
mkdir -p ~/.docker/data/gitlab
```

### 安装运行镜像

```bash
docker run --detach \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume ~/.docker/data/gitlab/config:/etc/gitlab \
  --volume ~/.docker/data/gitlab/logs:/var/log/gitlab \
  --volume ~/.docker/data/gitlab/data:/var/opt/gitlab \
  --shm-size 256m \
  gitlab/gitlab-ee:latest
```

![image-20220416173723138](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/163723.png)

安装需要一段时间，可以使用如下命令查看安装日志：

```bash
docker logs -f gitlab
```

当出现如下内容时安装完成，从 `/etc/gitlab/initial_root_password` 查找 `root` 账户的密码。

![image-20220416174703755](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/164703.png)

```bash
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

![image-20220416174833209](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/164833.png)

## 安装 Jenkins

### 创建目录存储持久数据

```bash
mkdir -p ~/.docker/data/jenkins
```

### 安装运行镜像

```bash
docker run -d -p 8080:8080 -p 50000:50000 \
  -v ~/.docker/data/jenkins:/var/jenkins_home \
  --restart always \
  jenkins/jenkins:lts-jdk11
```

![image-20220416181421427](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/161421.png)

安装完成后，访问 `http://localhost:8080` 访问 Jenkins 首页。

### 初始化 Jenkins

解锁 Jenkins，使用如下命令从日志中查找密码：

```bash
docker logs containerId
```

![image-20220416182210768](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/162210.png)

![image-20220416182012943](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/162013.png)

安装推荐插件

![image-20220416182735636](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/162735.png)

![image-20220416182809271](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/162809.png)

创建用户

![image-20220502140448072](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/140448.png)

完成安装

![image-20220502140544617](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/140544.png)

## 准备项目

准备两个项目用于配置自动构建：一个 SpringBoot 项目，直接使用内置 tomcat 容器启动；另一个 Vue 项目，使用 nginx 提供服务。

### 创建 SpringBoot 项目

使用 IDEA 的 `Spring Initializr` 创建 Spring Boot 项目： 

![image-20220502141400561](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/141400.png)

根据项目需要选择所需的依赖项：

![image-20220502141603535](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/141603.png)

添加一个用于测试的请求 `hello`，打印当前日期：

```java
package store.xianglin.cicd.background.controller;

import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * @author xianglin
 */
@RestController
@CrossOrigin(originPatterns = "*")
public class HelloController {

    @GetMapping("/hello")
    public String hello(@RequestParam(value = "name") String name) {
        String now = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        return "hello " + name + "， now is " + now;
    }
}

```

在浏览器中测试结果如下：

![image-20220502142817685](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/142817.png)

### 创建 Vue 项目

使用 IDEA 创建 Vue 项目可能需要先安装 Vue.js 插件：

![image-20220502143042588](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143042.png)

使用 IDEA 创建 Vue 项目：

![image-20220502143208246](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143208.png)

在 Vue 中使用 axios 向 SpringBoot 发送请求并展示。首先添加 axios 的依赖：

```bash
npm install axios 
```

然后修改 `src/components/HelloWorld.vue`：

```vue
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
  </div>
</template>

<script>

export default {
  name: 'HelloWorld',
  data() {
    return {
      msg: 'Vue'
    }
  },
  mounted() {
    setInterval(() => {
      const axios = require('axios');
      const a = this
      axios.get('http://localhost:9509/hello?name=Vue')
          .then(function (response) {
            a.msg = response.data
          }).catch(function (error) {
        console.log(error);
      })
    }, 1000)
  }
}
</script>
```

从 IDEA 中启动项目，在浏览器中测试结果：

![image-20220502153716482](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/153716.png)

### 将项目提交到 GitLab 上

首先需要在 GitLab 上创建项目对应的仓库，选择 “create a project” —— “创建空白项目”：

![image-20220502143747293](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143747.png)

![image-20220502143853500](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/143853.png)

填写项目基本信息后，点击“新建项目”：

![image-20220502144002812](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/144003.png)

使用如下命令将本地仓库推送到 GitLab 上（以 Spring Boot 项目为例）：

```bash
# 设置帐号信息
git config --global user.email "you@example.com"
git config --global user.name "Your Name"

# 提交修改
git add .
git commit -m 'init'

# 推送远程仓库
git remote add origin http://localhost/xianglin/background.git
git push -u origin --all
```

刷新 GitLab 仓库，即可看到提交信息：

![image-20220502154508544](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/154508.png)

在 IDEA 中也可以通过以下步骤提交并推送（以 Vue 项目为例）：

![image-20220502154744439](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/154744.png)

![image-20220502154929053](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/154929.png)

![2022-05-02 15-46-38屏幕截图](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/155034.png)

![image-20220502155107812](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/155108.png)

![image-20220502155223686](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/155223.png)

## 配置 Jenkins 自动构建及部署

### 全局工具配置

依次点击 “系统管理” —— “插件管理”、 “全局工具配置”：

![image-20220502161714979](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/161715.png)

![image-20220502161805008](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/161805.png)

在插件管理中，选择“可选插件”，选择对应插件勾选后点击 “Install without restart” 安装插件：

![image-20220502171059565](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/171059.png)

![image-20220502170238457](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/170238.png)

![image-20220502170535199](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/170535.png)

安装 NodeJS：

![image-20220502173944063](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/173944.png)

安装 JDK，这里选择使用 Jenkins 已有的 JDK 11：

![image-20220502162802181](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/162802.png)

安装 Maven：

![image-20220502162433577](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/162433.png)

修改 Maven 配置，使用阿里云的 Maven 镜像（根据创建 jenkins 使用的 `-v ~/.docker/data/jenkins:/var/jenkins_home` 可以将 `settings.xml` 文件复制到对应目录）：

![image-20220502163229489](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/163229.png)

### 配置 Spring Boot 自动构建

选择“新建任务”：

![image-20220502163615902](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/163615.png)

输入任务名称，选择”构建一个自由风格的软件项目“，点击“确定”：

![image-20220502163742410](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/163742.png)

在“源码管理”中选择“Git”，输入GitLab仓库地址，并添加访问的用户名和密码，其中 IP 地址可以通过命令 `docker inspect --format '{{ .NetworkSettings.IPAddress }}' <container-ID>` 获取：

![image-20220502165129270](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/165129.png)

![image-20220502164009167](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/164009.png)

在“构建”中选择“调用顶层 Maven 目标”，在“构建后操作”中选择“归档成品”：

![image-20220502173134591](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/173134.png)

保存后点击“立即构建”，查看构建结果：

![image-20220502173240736](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/173240.png)

### 配置 Vue 自动构建

“源码管理”和上述一样，在“构建触发器”中选择“Provide Node & npm bin/folder to PATH”：

![image-20220502183321578](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/183321.png)

在“构建”中选择“执行shell”，在“构建后操作”中选择“归档成品”：

```shell
cnpm install
cnpm  run build
# 打包，方便下载和上传到服务器部署
tar -cf dist.tar dist
```

![image-20220502183524271](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/183524.png)

保存后点击“立即构建”，查看构建结果：

![image-20220502183634074](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/183634.png)

### 自动部署到服务器

自动构建完成后，可以手动将构建结果下载到本地执行，也可以通过 Jenkins 自动部署到指定服务器上，借助“Publish over SSH”插件完成。安装插件后在“系统管理”——“系统配置”中配置“Publish over SSH”，如图：

![image-20220502211327631](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/211327.png)

配置完成后可以通过“Test Configuration”验证配置是否正确可用：

![image-20220502211420139](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/211420.png)

#### 部署 Spring Boot 应用

直接通过 `jar` 命令启动 Spring Boot 应用，计划将 jar 文件放在 `/root/deploy/background`，编写启动脚本 `deploy.sh`：

```bash
#!/bin/bash

# deploy spring boot app

# stop server
if [ -f "/var/run/springboot.pid" ]; then
	kill -15 $(cat /var/run/springboot.pid)
fi

# start server
nohup /root/jdk-11.0.15+10/bin/java -jar /root/deploy/background/backgroud-0.0.1-SNAPSHOT.jar > /dev/null 2>&1 &
```

在”构建后操作“——”Send build artifacts over SSH“中配置如图所示：

![image-20220502220639362](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/220639.png)

最总效果如下：

![image-20220502220802535](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/220802.png)

#### 部署 Vue 应用

Vue 上传到服务器到配置与上述相同，静态文件使用 Nginx 作为服务器，将 Jenkins 构建到 dist.tar 解压到对应目录即可，命令如下：

```bash
#!/bin/bash

# deploy vue app

# extract file
tar -xf /root/deploy/frontend/dist.tar -C /usr/share/nginx/html
```

修改 Nginx 到配置 `/etc/nginx/conf.d/default.conf` ，如下：

```nginx
location / {
    root   /usr/share/nginx/html/dist;
    index  index.html index.htm;
}
```

启动 Nginx 即可。
