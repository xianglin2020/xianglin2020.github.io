---
title: "启动加载器解析 SpringBoot源码学习"
date: 2023-11-18T20:27:43+08:00
categories:
  - SpringBoot源码学习
tags:
  - SpringBoot
summary: "SpringBoot 计时器介绍、启动加载器案例演示、启动加载器原理解析。"
description: "SpringBoot 计时器介绍、启动加载器案例演示、启动加载器原理解析。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## 计时器介绍

StopWatch 是 Spring 提供的秒表工具类，允许对多个任务进行计时，显示总运行时间和每个指定任务的运行时间，简单使用例子。

```java
public class StopWatchTest {
    @Test
    public void testStopWatch() throws InterruptedException {
        var stopWatch = new StopWatch("testWatch");
        stopWatch.start("task1");
        TimeUnit.SECONDS.sleep(2);
        stopWatch.stop();
        stopWatch.start("task2");
        TimeUnit.SECONDS.sleep(1);
        stopWatch.stop();
        stopWatch.start("task3");
        TimeUnit.SECONDS.sleep(3);
        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint());
        System.out.println(stopWatch);
    }
}
```

![image-20231118204128112](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700311288.png)

## 启动加载器案例

实现 CommandLineRunner 和 ApplicationRunner 接口的类称为启动加载器，用于在 SpringApplication 启动完成后立即执行其 run 方法，多个启动加载器通过使用 Ordered 接口或 @Order 注释进行排序，Order 值相同时 ApplicationRunner 优先执行。

```java
@Component
@Order(1)
public class FirstApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) {
        System.out.println("\u001B[32m >>> startup first ApplicationRunner <<<");
    }
}

@Component
@Order(1)
public class FirstCommandlineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) {
        System.out.println("\u001B[32m >>> startup first CommandlineRunner <<<");
    }
}
```

## 启动加载器原理解析

启动加载器调用的入口在 `org.springframework.boot.SpringApplication#run(java.lang.String...)` 方法的 callRunners 方法中。

```java
public ConfigurableApplicationContext run(String... args) {
  // 将启动参数包装为 ApplicationArguments
  ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
  // ...
  listeners.started(context);
  // SpringApplication 启动完成后执行
  callRunners(context, applicationArguments);
  // ...
  listeners.running(context);
}
```

callRunners 的逻辑比较简单，从容器中获取所有实现 ApplicationRunner 和 CommandLineRunner 接口的实例，按 Order 排序后依次调用。

```java
private void callRunners(ApplicationContext context, ApplicationArguments args) {
  List<Object> runners = new ArrayList<>();
  // 因为先加入 ApplicationRunner 后加入 CommandLineRunner，排序后 Order 值相同的 ApplicationRunner 靠前
  runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
  runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
  AnnotationAwareOrderComparator.sort(runners);
  for (Object runner : new LinkedHashSet<>(runners)) {
    if (runner instanceof ApplicationRunner) {
      callRunner((ApplicationRunner) runner, args);
    }
    if (runner instanceof CommandLineRunner) {
      callRunner((CommandLineRunner) runner, args);
    }
  }
}
```

ApplicationRunner 和 CommandLineRunner 唯一不同是 run 方法的入参不一样，ApplicationArguments 是对 `String[] args` 入参的包装，默认实现 DefaultApplicationArguments 会将 `String[] args` 解析成 CommandLineArgs。
