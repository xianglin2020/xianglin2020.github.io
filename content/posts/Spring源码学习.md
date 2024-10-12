---
title: Spring 源码学习
date: 2023-01-06 13:04:14
tags: [spring]
categories: [learn]
description: Spring v5.2.0.RELEASE 源码学习。
summary: Spring v5.2.0.RELEASE 源码学习。
---

# 源码下载编译

## Spring 源码下载

在 GitHub 上打开 [spring-framework 项目](https://github.com/spring-projects/spring-framework)，从源码分支中找到 `v5.2.0.RELEASE` 标签，点击 `Code` 按钮下载源码压缩包。

![image-20230106130844454](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/06131500.png)

![image-20230106130946905](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/06131511.png)

## 导入 IDEA

参[照官网步](https://github.com/spring-projects/spring-framework/blob/main/import-into-idea.md)骤将源码导入 IDEA 中。

* 预编译 `spring-oxm` 模块，打开 spring 源码目录，在命名行中执行 `./gradlew :spring-oxm:compileTestJava` 命令完成预编译。

* 在 IDEA 中导入项目。

  ![image-20230106132121166](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/06132121.png)

* 排除 `spring-aspects` 模块

  ![image-20230106133723159](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/06133723.png)

# Spring IOC 容器

## 源码解析核心接口和类

### Bean 和 BeanDefinition

![BeanDefinition](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08143129.png)

Bean 的本质就是 Java 对象，只是这个对象的生命周期由容器管理。根据配置，生成用来描述 Bean 的 BeanDefinition，常用属性：

* 作用范围：`scope`（`@Scope`）
* 懒加载 `lazy-init`（`@Lazy`）
* 首选 `primary`（`@Primary`）：设置为 true 的 bean 会是优先的实现类
* `factory-bean` 和 `factory-method`（`@Configuration` 和 `@Bean`）

### 容器初始化主要做的事情

![Spring 容器初始化主要脉络.drawio](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/10200953.png)

解析配置

定位与注册组件

### BeanFactory

![BeanFactory](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08145334.png)

* 组件扫描：Spring 自动发现应用容器中需要创建的 Bean。
* 自动装配：自动满足 Bean 之间的依赖。

### ApplicationContext

![ApplicationContext](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08151639.png)

基于 XML 配置的经典容器

* FileSystemXmlApplicationContext：从文件系统加载配置。

* ClassPathXmlApplicationContext：从 `classpath` 加载配置。

* XmlWebApplicationContext：用于 WEB 应用程序的容器。

基于注解的容器

* AnnotationConfigServletWebServerApplicationContext

* AnnotationConfigReactiveWebServerApplicationContext

* AnnotationConfigApplicationContext：基于注解的应用程序容器。

refresh() 功能

### Resource

![ByteArrayResource.java](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08152657.png)

### ResourceLoader

![ResourceLoader](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08153704.png)

### BeanDefinitionReader

![BeanDefinitionReader](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08175136.png)

### BeanDefinitionRegistry

![BeanDefinitionRegistry](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/11211047.png)

## 容器初始化 `refresh`

`org.springframework.context.support.AbstractApplicationContext#refresh`

### 后置处理器 `PostProcessor`

![PostProcessor](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/11220102.png)

本身也是一种需要注册到容器里的 Bean，其里面的方法会在特定时机被容器调用，实现不改变容器或者 Bean 核心逻辑的情况下对 Bean 进行扩展：对 Bean 进行包装，影响其行为、修改 Bean 的内容。

容器级别的后置处理器：`BeanFactoryPostProcessor`、`BeanDefinitionRegistryPostProcessor`；

Bean 级别的后置处理器：`BeanPostProcessor`。

### `Aware`

![Aware](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202302/06210641.png)

### 事件监听器

# Spring 依赖注入

## Spring 循环依赖

### Spring 循环依赖支持情况

Spring 仅支持属性注入的单例循环依赖。



# Spring AOP

## AOP 概念

* 切面 Aspect：将横切关注点逻辑进行模块化封装的实体对象。
* 通知 Advice：好比是 Class 里面的方法，还定义了织入逻辑的时机。
* 连接点 Joinpoint：允许使用 Advice 的地方（Spring 只支持方法级别的 Joinpoint）。
* 切入点 Pointcut：定义一些列规则对 Joinpoint 进行筛选。
* 目标对象 Target：符合 Pointcut 条件，要被织入横切逻辑的对象。
* 织入：将 Aspect 模块化的横切关注点集成到 OOP 里。
* 织入器：完成织入过程的执行者，如 ajc。

## Advice 的种类

* BeforeAdvice
* AfterAdvice：好比 try……catch……finally 里面的 finally
* AfterReturning
* AfterThrowingAdvice
* AroundAdvice

## Aspect 执行顺序

单个 Aspect 的执行顺序

![单个 Aspect 执行顺序](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021655325.png)

多个 Aspect 的执行顺序

![多个 Aspect 执行顺序](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021655326.png)

## 引入型 Advice Introduction

为目标类引入新接口，不需要目标类做任何实现。新建一个接口以及实现类。

```java
public interface ILittleUniverse {
    void burningUp();
}
public class LittleUniverseImpl implements ILittleUniverse {
    @Override
    public void burningUp() {
        System.out.println("burningUp");
    }
}
```

在切面类中配置 introduction。

```java
@Aspect
@Component
public class ServeAspect {
    @DeclareParents(value = "store.xianglin.controller..*", defaultImpl = LittleUniverseImpl.class)
    public ILittleUniverse littleUniverse;
}
```

使用 introduction。

```java
Object bean = applicationContext.getBean(WelcomeController.class);
ILittleUniverse littleUniverse = (ILittleUniverse) bean;
littleUniverse.burningUp();
```



## Spring MVC 核心流程

1. 建立请求和 Controller 方法的映射集合的流程；
2. 根据请求查找对应的 Controller 方法的流程；
3. 请求参数绑定到方法形参，执行方法处理请求，渲染视图的流程。

![SpringMVC核心流程](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021655327.png)
