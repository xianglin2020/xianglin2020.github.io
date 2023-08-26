---
title: java 初识
date: 2020-09-06 22:47:54
categories: [learn]
tags: [java]
---

# Java 初识

## Java 简介

* Java 是一门面向对象的程序设计语言。
* 1995 年由 sun 公司发布。
* 2010 年 sun 公司被 Oracle 收购，Java 现在属于 Oracle 公司。

## JVM

* JVM（Java Virtual Machine），Java 虚拟机的简称。
* JVM 是 Java 语言平台无关性的关键。

## Java 程序执行的流程

![image-20200511220044340](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224832.png)

## JDK

* JDK（Java Development Kit），Java 语言的软件开发包。
* 两个主要的组件：
  * `javac`，将源程序编译成字节码
  * `java`，运行编译后的Java 程序

## JRE

* JRE（Java Runtime Environment）
* 包括 Java 虚拟机（JVM）、Java 核心类库和支持文件
* 如果只需要运行 Java 程序，下载并安装 JRE即可
* 如果要开发 Java 软件，需要下载 JDK
* 在 JDK 中包含 JRE

 ## JDK、JRE 和 JVM 三者的关系

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212948.png" alt="image-20200511220713080" style="zoom:25%;" />

* JRE = JVM + JavaSE标准类库
* JDK = JRE + 开发工具集

## Java 平台

* Java SE：Java 标准版，用于开发桌面程序
* Java EE：Java 企业版，用于开发 Web 程序（电商网站、门户网站）
* Java ME：Java 微型版，用于移动设备，现占用率已不高

## JDK 安装

* [Oracle官网](https://www.oracle.com/java/technologies/javase-downloads.html)下载各个版本的 JDK

* MAC 下使用 [HomeBrew](https://brew.sh/index_zh-cn) 安装各个版本的 [JDK](https://www.cnblogs.com/imzhizi/p/macos-jdk-installation-homebrew.html)

* 验证 JDK 安装：`java -version`

  ![image-20200512214446103](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212953.png)

## 使用 Eclipse 开发 Java 程序

* Eclipse [下载地址](https://www.eclipse.org/downloads/)

* 使用 HomeBrew 安装 Eclipse

  ![image-20200512215316576](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224833.png)

* 使用 Eclipse 创建 Java 项目

  1. 创建一个名为`FirstJavaProject`的项目

     <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/213002.png" alt="image-20200512215557878" style="zoom:50%;" />

  2. 创建一个名为`person.xianglin`的包

     ![截屏2020-05-12 21.57.24](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/213007.png)

  3. 创建第一个名为`HelloWorld`的 Java 类

     ![image-20200512215937592](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224834.png)

  4. 完成编程语言第一壮举：输出`Hello World!`

     ![image-20200512220419959](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224836.png)
