---
title: 接口打包、安装到本地、上传至仓库
description: 接口打包、安装到本地、上传至仓库。
summary: 接口打包、安装到本地、上传至仓库。
date: 2021-04-23 09:48:19
tags: [java, maven]
categories: [work record]
---

## 使用`jar`命令将 class 文件打包成 jar 文件

```shell
jar cvf test.jar .\store\xianglin\mvn\test\
```

## 将生成的 jar 文件安装至本地仓库

```shell
mvn install:install-file -Dfile=C:\Users\xianglin\Documents\mvn\target\classes\test-1.0.jar -DgroupId=store.xianglin -DartifactId=mvn -Dversion=1.0 -Dpackaging=jar
```

## 将生成的 jar 文件上传至 Maven 私服

```shell
mvn deploy:deploy-file -DgroupId=store.xinaglin -DartifactId=test  -Dversion=1.0   -Dfile=test-1.0.jar   -Durl=http://127.0.0.1/nexus/content/repositories/mvn/ -DrepositoryId=mvn
