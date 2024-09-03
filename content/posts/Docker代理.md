---
title: "Docker代理"
date: 2024-09-01T11:58:07+08:00
categories:
  - learn
tags:
  - docker
summary: ""
description: ""
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## 前提

*需要一个梯子*

## 设置代理

### 用于拉取镜像

> [Configure the daemon to use a proxy](https://docs.docker.com/engine/daemon/proxy/)

编辑 `/etc/docker/daemon.json` 文件，添加代理配置

```json
{
  "proxies": {
    "http-proxy": "http://192.168.31.113:7897",
    "https-proxy": "http://192.168.31.113:7897",
    "no-proxy": "*.test.example.com,.example.org,127.0.0.0/8"
  }
}
```

### 用于构建镜像

> [Proxy arguments](https://docs.docker.com/build/building/variables/#proxy-arguments)
 
在 build 命令中使用 --build-arg 参数

```shell
docker image build -t web:latest  --build-arg HTTP_PROXY=http://192.168.31.113:7897 --build-arg HTTPS_PROXY=http://192.168.31.113:7897 .
```

### 为容器设置代理

> [Automatic proxy configuration for containers](https://docs.docker.com/reference/cli/docker/#automatic-proxy-configuration-for-containers)

编辑 `~/.docker/config.json` 文件，添加代理配置

```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://192.168.31.113:7897",
      "httpsProxy": "http://192.168.31.113:7897",
      "noProxy": "localhost,127.0.0.1,.example.com"
    }
  }
}
```

## 效果

*全配置了，下载 maven 项目依赖也是 Read timed out！！！*