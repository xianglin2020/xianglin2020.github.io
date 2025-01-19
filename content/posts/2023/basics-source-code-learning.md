---
title: "全局流程解析 SpringBoot源码学习"
date: 2023-11-08T19:46:32+08:00
categories:
  - SpringBoot源码学习
tags:
  - SpringBoot
summary: "SpringBoot 项目环境准备、工程搭建、框架启动流程。"
description: "SpringBoot 项目环境准备、工程搭建、框架启动流程。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## 环境准备

Java 8+：https://jdk.java.net/archive/

MySQL 5.7+：https://www.mysql.com/cn/downloads/

Maven 3.3+：https://maven.apache.org/download.cgi

IDEA：https://www.jetbrains.com.cn/idea/

实例代码：https://github.com/xianglin2020/springboot2-source-study

## SpringBoot 启动介绍

一行代码完成启动。

```java
@SpringBootApplication
@MapperScan("store.xianglin.sb2.mapper")
public class SpringBoot2Application {
    public static void main(String[] args) {
        SpringApplication.run(SpringBoot2Application.class, args);
    }
}
```

### 框架初始化

`new SpringApplication` 主要步骤见：`org.springframework.boot.SpringApplication#SpringApplication(org.springframework.core.io.ResourceLoader, java.lang.Class<?>...)`。

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
  // 配置资源加载器
  this.resourceLoader = resourceLoader;
  Assert.notNull(primarySources, "PrimarySources must not be null");
  // 配置 primarySources
  this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
  // 应用环境检测
  this.webApplicationType = WebApplicationType.deduceFromClasspath();
  // 配置系统初始化器
  setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
  // 配置应用监听器
  setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
  // 配置 main 方法所在类
  this.mainApplicationClass = deduceMainApplicationClass();
}
```

### 框架启动

主要步骤见：`org.springframework.boot.SpringApplication#run(java.lang.String...)`。

```java
public ConfigurableApplicationContext run(String... args) {
  // 计时器开始计时
  StopWatch stopWatch = new StopWatch();
  stopWatch.start();
  ConfigurableApplicationContext context = null;
  Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
  // Headless 模式赋值
  configureHeadlessProperty();
  // 发送 starting 事件
  SpringApplicationRunListeners listeners = getRunListeners(args);
  listeners.starting();
  try {
    ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
    // 配置环境模块，发送 environmentPrepared 事件
    ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
    configureIgnoreBeanInfo(environment);
    // 打印 Banner
    Banner printedBanner = printBanner(environment);
    // 创建应用上下文
    context = createApplicationContext();
    // 初始化失败分析器
    exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
        new Class[] { ConfigurableApplicationContext.class }, context);
    // 关联 SpringBoot 组件与应用上下文
    // 1、发送 contextPrepared 事件 
    // 2、加载 source 到 context
    // 3、发送 contextLoaded 事件
    prepareContext(context, environment, listeners, applicationArguments, printedBanner);
    // 刷新上下文
    refreshContext(context);
    afterRefresh(context, applicationArguments);
    // 计时器停止
    stopWatch.stop();
    if (this.logStartupInfo) {
      new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
    }
    // 发送 started 事件
    listeners.started(context);
    // 调用框架启动扩展类：实现ApplicationRunner和CommandLineRunner
    callRunners(context, applicationArguments);
  }
  catch (Throwable ex) {
    handleRunFailure(context, ex, exceptionReporters, listeners);
    throw new IllegalStateException(ex);
  }

  try {
    // 发送 running 事件
    listeners.running(context);
  }
  catch (Throwable ex) {
    handleRunFailure(context, ex, exceptionReporters, null);
    throw new IllegalStateException(ex);
  }
  return context;
}
```

上述过程中的事件定义在 `org.springframework.boot.SpringApplicationRunListener` 中。

> 无头模式：[Selecting an AWT mode](https://www.ibm.com/docs/it/i/7.3?topic=toolkit-selecting-awt-mode)

### 自动化装配

1. 收集配置文件中的配置工厂类
2. 加载组件工厂
3. 注册组件内定义的 bean
