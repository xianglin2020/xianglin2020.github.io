---
title: Spring 源码学习
date: 2023-01-06 13:04:14
tags: [ spring ]
categories: [ learn ]
description: Spring v5.2.0.RELEASE 源码学习。
summary: Spring v5.2.0.RELEASE 源码学习。
---

## 源码下载编译

### Spring 源码下载

在 GitHub 上打开 [spring-framework 项目](https://github.com/spring-projects/spring-framework)，从源码分支中找到
`v5.2.0.RELEASE` 标签，点击 `Code` 按钮下载源码压缩包。

![image-20230106130844454](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/06131500.png)

![image-20230106130946905](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/06131511.png)

### 导入 IDEA

参[照官网步](https://github.com/spring-projects/spring-framework/blob/main/import-into-idea.md)骤将源码导入 IDEA 中。

* 预编译 `spring-oxm` 模块，打开 spring 源码目录，在命名行中执行 `./gradlew :spring-oxm:compileTestJava` 命令完成预编译。

* 在 IDEA 中导入项目。

  ![image-20230106132121166](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/06132121.png)

* 排除 `spring-aspects` 模块

  ![image-20230106133723159](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/06133723.png)

## Spring IOC 容器

#### 容器初始化主要做的事情

![Spring 容器初始化主要脉络.drawio](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/10200953.png)

解析配置

定位与注册组件

### 源码解析核心接口和类

#### Bean 和 BeanDefinition

![BeanDefinition](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08143129.png)

Bean 的本质就是 Java 对象，只是这个对象的生命周期由容器管理。根据配置，生成用来描述 Bean 的 BeanDefinition，常用属性：

* 作用范围：`scope`（`@Scope`）
* 懒加载 `lazy-init`（`@Lazy`）
* 首选 `primary`（`@Primary`）：设置为 true 的 bean 会是优先的实现类
* `factory-bean` 和 `factory-method`（`@Configuration` 和 `@Bean`）

#### BeanFactory

![BeanFactory](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08145334.png)

* 组件扫描：Spring 自动发现应用容器中需要创建的 Bean。
* 自动装配：自动满足 Bean 之间的依赖。

#### ApplicationContext

![ApplicationContext](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08151639.png)

基于 XML 配置的经典容器

* FileSystemXmlApplicationContext：从文件系统加载配置。

* ClassPathXmlApplicationContext：从 `classpath` 加载配置。

* XmlWebApplicationContext：用于 WEB 应用程序的容器。

基于注解的容器

* AnnotationConfigServletWebServerApplicationContext

* AnnotationConfigReactiveWebServerApplicationContext

* AnnotationConfigApplicationContext：基于注解的应用程序容器。

#### Resource

![ByteArrayResource.java](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08152657.png)

#### ResourceLoader

![ResourceLoader](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08153704.png)

#### BeanDefinitionReader

![BeanDefinitionReader](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/08175136.png)

#### BeanDefinitionRegistry

![BeanDefinitionRegistry](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/11211047.png)

## 容器初始化

### 后置处理器 `PostProcessor`

![PostProcessor](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/11220102.png)

本身也是一种需要注册到容器里的 Bean，其里面的方法会在特定时机被容器调用。

实现不改变容器或者 Bean 核心逻辑的情况下对 Bean 进行扩展：对 Bean 进行包装，影响其行为、修改 Bean 的内容。

容器级别的后置处理器：`BeanFactoryPostProcessor`、`BeanDefinitionRegistryPostProcessor`；

Bean 级别的后置处理器：`BeanPostProcessor`。

### `Aware`

![Aware](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202302/06210641.png)

一个标记接口，指示 Bean 实现接口后能通过对应的回调方法获取所需的容器对象。

### 事件监听器

* 事件：ApplicationEvent 抽象类

  ![image-20241013114159934](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410131608414.png)

* 事件监听器：ApplicationListener

  ![image-20241013114411957](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410131608516.png)

* 事件发布器：Publisher 以及 Multicaster

  ![image-20241013115423391](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410131608217.png)

### `refresh`

org.springframework.context.support.AbstractApplicationContext#refresh

* prepareRefresh：刷新前的准备工作，设置状态、设置启动时间
* obtainFreshBeanFactory：获取子类刷新后的 BeanFactory 实例
* prepareBeanFactory：为容器注册必要的系统级别的 bean
* postProcessBeanFactory：允许容器的子类去注册 PostProcessor
* invokeBeanFactoryPostProcessors：实例化并调用所有已注册的的 BeanFactoryPostProcessor
* registerBeanPostProcessor：注册Bean级别的后置处理器
* initMessageSource：初始化国际化配置
* initApplicationEventMulticaster：初始化事件发布者
* onRefresh：预留给子类用于初始化其它特殊的 bean
* registerListeners：向事件发布者注册事件监听器
* finishBeanFactoryInitialization：完成 BeanFactory 初始化，初始化所有非延迟加载的单例
* finishRefresh：触发初始化完成的回调方法，发送容器刷新完毕事件
* resetCommonCaches：重置Spring中的共用的缓存

## Bean 实例创建

* AbstractBeanFactory#doGetBean：获取 Bean 实例
* DefaultSingletonBeanRegistry#getSingleton：获取单例实例（三级缓存）
* AbstractAutowireCapableBeanFactory#createBean：创建 Bean 实例
    * doCreateBean：创建 Bean 实例
    * applyMergedBeanDefinitionPostProcessors：处理 @Autowired 和 @Value 注解
    * populateBean：为 Bean 实例注入属性值
* AutowiredAnnotationBeanPostProcessor#postProcessProperties：依赖注入逻辑
* DefaultListableBeanFactory#doResolveDependency：依赖解析
* DependencyDescriptor (InjectionPoint) ：创建依赖实例

### `doGetBean`

* getSingleton：尝试从缓存获取 Bean
* 判断循环依赖
* getParentBeanFactory：递归去父容器获取 Bean 实例
* getMergedLocalBeanDefinition：从当前容器获取 BeanDefinition 实例
* getDependsOn：递归实例化显示依赖的 Bean depends-on
* 根据不同的 Scope 采用不同的策略创建 Bean 实例
* 对 Bean 进行类型检查

### `createBean`

* resolveBeanClass：Bean 类型解析
* prepareMethodOverrides：处理方法覆盖
* resolveBeforeInstantiation：应用 Bean 实例化前的后置处理器，如果生成对象则直接返回
* doCreateBean：创建Bean的入口

### `doCreateBean`

* createBeanInstance：创建 Bean 实例（工厂方法、含参构造器、无参构造器）
* applyMergedBeanDefinitionPostProcessors：记录下被 @Autowired 或者 @Value 标记上的方法和成员变量
* addSingletonFactory：如果允许提前暴露，向三级缓存添加记录
* populateBean：填充 Bean 属性
* initializeBean：初始化 Bean
* registerDisposableBeanIfNecessary：注册 Bean 的销毁逻辑

### Spring 单例循环依赖

![image-20241014103722774](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202410141037893.png)

## Spring AOP

### AOP 概念

* 切面 Aspect：将横切关注点逻辑进行模块化封装的实体对象。
* 通知 Advice：好比是 Class 里面的方法，还定义了织入逻辑的时机。
* 连接点 Joinpoint：允许使用 Advice 的地方（Spring 只支持方法级别的 Joinpoint）。
* 切入点 Pointcut：定义一些列规则对 Joinpoint 进行筛选。
* 目标对象 Target：符合 Pointcut 条件，要被织入横切逻辑的对象。
* 织入：将 Aspect 模块化的横切关注点集成到 OOP 里。
* 织入器：完成织入过程的执行者，如 ajc。

### Advice 的种类

* BeforeAdvice
* AfterAdvice：好比 try……catch……finally 里面的 finally
* AfterReturning
* AfterThrowingAdvice
* AroundAdvice

### Aspect 执行顺序

单个 Aspect 的执行顺序

![单个 Aspect 执行顺序](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021655325.png)

多个 Aspect 的执行顺序

![多个 Aspect 执行顺序](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021655326.png)

### 引入型 Advice Introduction

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
littleUniverse.

burningUp();
```

## Spring MVC 核心流程

1. 建立请求和 Controller 方法的映射集合的流程；
2. 根据请求查找对应的 Controller 方法的流程；
3. 请求参数绑定到方法形参，执行方法处理请求，渲染视图的流程。

![SpringMVC核心流程](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202307021655327.png)
