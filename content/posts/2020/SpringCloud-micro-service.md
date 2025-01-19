---
title: SpringCloud 和微服务
date: 2020-12-06 12:15:12
categories: [learn]
tags: [spring cloud, 微服务]
---

# 微服务

## 微服务的基础概念

### 单体应用的不足

* 部署效率低下：依赖众多、应用体积较大、启动时间较长
* 团队协作开发成本高
* 系统可用性差

### 服务化

* 把传统的单机应用中的本地方法调用，改造成通过 RPC、HTTP 产生的远程方法调用
* 把模块从单体应用中拆分出来，独立成一个服务部署

### 微服务概念

* 一种架构风格
* 开发单个应用作为一系列小型服务的套件，其中每个服务都运行在自己的进程中，并且通过轻量级的机制实现彼此间的通信，这通常是 HTTP 资源 API
* 这些服务是围绕着业务功能构建的，并且可以通过完全自动化的部署机制进行独立部署
* 这些服务的集中化管理做到了最小化（例如 docker 相关技术），每一种服务都可以通过不同的编程语言进行编写，并且可以使用不同的数据存储技术

### 微服务的特点

* 组件以服务形式来提供
* 微服务是产品而不是项目
* 轻量级通信、独立进程
* 分散治理、去中心化治理
* 容错性设计
* 会带来团队组织架构的调整

### 微服务的优缺点

* 服务简单、便于学习和上手、相对易于维护
* 独立部署、灵活扩展
* 技术栈丰富

* 运维成本过高
* 接口可能不匹配
* 代码可能重复
* 架构复杂度提高

## 微服的两大门派

* Spring Cloud：众多子项目
* dubbo：高性能、轻量级的开源 JavaRPC 框架，它提供了三大核心能力：面向接口的远程方法调用、智能容错和负载均衡、服务自动注册和发现

### 通信协议：RPC、HTTP

* RPC 的整体效率较高
* 服务提供方与调用方接口依赖方式太强
* 服务对平台敏感，难以简单复用

## 微服务重要模块

* 服务描述
* 注册中心
* 服务框架
* 负载均衡
* 熔断和降级
* 网关

# Spring Cloud

* SpringCloud 是一个成熟的微服务框架，定位为开发人员提供工具，以快速构建分布式系统

![image-20201206134046945](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/134047.png)

## Spring Cloud Netflix

### Netflix Eureka

* 服务中心，云端服务发现，一个基于 REST 的服务，用于定位服务。

### Netflix Hystrix

* 熔断器，容错管理工具，旨在通过熔断机制控制服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。

### Netflix Zuul

* Zuul 是提供动态路由、监控、弹性、安全等边缘服务的框架。

## 基础项目结构搭建

### 基础功能

* 实现一个课程列表和课程价格的接口服务
* 创建两个项目：`course-list`和`course-price`
* 实现两个接口：

  * 返回课程列表：`/courses`

  * 返回单个课程价格：`/price`


### 表设计如下

```sql
  CREATE TABLE `course` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
    `course_id` int(20) DEFAULT NULL COMMENT '课程 ID',
    `course_name` varchar(50) DEFAULT NULL COMMENT '课程名',
    `valid` int(1) DEFAULT NULL COMMENT '是否上架',
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  INSERT INTO `course` (`id`, `course_id`, `course_name`, `valid`)
  VALUES
  	(1,362,'Java 并发编程精讲',1),
  	(2,409,'玩转 Java 并发工具',1),
  	(3,345,'Nginx 教程',0);
  	
  
  CREATE TABLE `course_price` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
    `course_id` int(50) DEFAULT NULL COMMENT '课程 ID',
    `price` int(20) DEFAULT NULL COMMENT '课程价格',
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  INSERT INTO `course_price` (`id`, `course_id`, `price`)
  VALUES
  	(1,362,348),
  	(2,409,399);
```

### 项目搭建

* 搭建一个空的 springboot 项目`spring-cloud-exercise-course`，作为整个项目的 `parent`，删掉无用的`src`目录

  ![image-20201206181615615](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/190928.png)

* 新建一个空的 `maven `项目`course-service`作为`course-list`和`course-price`的`parent`，删掉无用的`src`目录。

  ![image-20201206181920496](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/181920.png)

  引入一些公共依赖，比如 `jdbc`、`mybatis`等。

  ```xml
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>2.1.3</version>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.12</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
  ```

* 新建`course-list`和`course-price`两个 `module`，开发`/courses`和`/price`接口。因为需要的依赖已经在`course-service`中引入，这两个 `module` 无需引入依赖。

  ![image-20201206182430587](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/182519.png)
  
* 开发`/courses`接口
  
  ![image-20201206185335187](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/185335.png)
  
* 开发`/price`接口
  
  ![image-20201206185846958](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/185847.png)
  
* 简单测试这两个接口

  ![image-20201206190055492](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/190055.png)

  ![image-20201206190108465](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/190108.png)

* 准备工作已经完成，基于 springboot 使用 MyBatis 的查询项目很简单。

### 版本搭配

* SpringBoot 与 SpringCloud 的版本选择，可以参见`https://start.spring.io/actuator/info`的返回

  ![image-20201206190657723](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/190657.png)

* SpringBoot 和 MyBatis 的版本选择，可以参见`http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/`

  ![image-20201206190811467](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/190811.png)

## 服务注册与发现 Eureka

* 服务中心又称为注册中心，管理各种服务功能包括服务的注册、发现、熔断、负载、降级等。

### Eureka 介绍

* Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务注册和发现。
* Eureka 采用了 C-S 的设计架构，Eureka Server 作为服务注册的服务器，它是服务注册中心。
* 系统中的其它微服务，使用 Eureka 的客户端连接到 Eureka Server，并维持心跳连接。
* 系统维护人员可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。
* Spring Cloud 的其它模块可以通过 Eureka Server 来发现系统中的其它微服务，并执行性格逻辑。

![image-20201209214640027](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/214640.png)

* 如图所示：
  * Eureka Server：提供服务注册于发现
  * Service Provider：服务提供方，将自身服务注册到 Eureka，使得消费者能到找到
  * Service Consumer：服务消费方，从 Eureka 获取注册服务列表，从而消费服务

## 使用 Eureka Server

* 基于上面的案例，`course-price`模块需要使用`course-list`模块提供的服务，可以启动一个 Eureka Server，将两个模块作为 Eureka Client 注册到服务中心，`course-list`作为服务提供者，`course-price`作为服务消费者。

### 配置 EurekaServer

* 新建一个模块`eureka-server` 作为注册中心

* 引入 Eureka Server 的依赖

  首先在整个项目的顶层 `POM`文件中加入对 Spring Cloud 版本的定义

  ```xml
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Hoxton.SR9</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  ```

  然后在`eureka-server`模块引入Eureka Server 的依赖

  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  </dependency>
  ```

  此时整个项目的情况是

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/220327.png" alt="image-20201209220327274" style="zoom:50%;" />

* 配置 Eureka Server

  新建配置文件`application.yml`，对 Eureka Server 进行配置

  ```yaml
  spring:
    application:
      name: eureka-server
  server:
    port: 8000
  eureka:
    instance:
      hostname: localhost
    client:
      fetch-registry: off
      register-with-eureka: false
      service-url:
        defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  ```

  * `eureka.client.fetch-registry`：是否需要从注册中心获取其它服务的地址，因为当前模块是注册中心，设置为 `false`
  * `eureka.client.register-with-eureka`：是否将自己作为服务注册到服务中心，设置为`false`，取消默认的注册行为
  * `eureka.client.service-url.defaultZone`：设置与 Eureka Server 交互地址，查询服务和注册服务都依赖这个地址，默认是`http://localhost:8761/eureka/`。

* 为`eureka-server`模块编写启动类

  ```java
  @SpringBootApplication
  @EnableEurekaServer
  public class EurekaServerApplication {
      public static void main(String[] args) {
          SpringApplication.run(EurekaServerApplication.class, args);
      }
  }
  ```

* 启动服务后，访问配置的地址

  ![image-20201209222016692](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/222016.png)

  此时没有任务服务注册到 Eureka Server，如果将上面的`register-with-eureka`设置为`true`。Eureka Server 会将自己注册到服务中心。

  ![image-20201209223104463](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/223104.png)

### 将`course-list`和`course-price`两个模块注册到服务中心

* 为 Eureka Client 添加依赖

  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>
  ```

* 在配置文件中增加 Eureka Server 的配置

  ```properties
  eureka.client.service-url.defaultZone=http://localhost:8000/eureka/
  ```

* 启动服务后，访问`http://localhost:8000`，发现`course-list`和`course-price`已经正确注册到服务中心。

  ![image-20201209223813993](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/223814.png)

### 多节点注册中心

* Eureka 可以通过互相注册的方式实现集群部署，只需要将 Eureka Server 的`service-url.defaultZone`配置为其它节点的地址即可。

* 保证多个节点的应用名一致，`register-with-eureka`和`fetch-registry`配置为`true`。如下所示：

  ```yaml
  eureka:
    client:
      register-with-eureka: true
      fetch-registry: true
  spring:
    application:
      name: eureka-server
  ---
  spring:
    profiles: eureka-server1
  server:
    port: 7000
  eureka:
    instance:
      hostname: xianglin1
    client:
      service-url:
        defaultZone: http://xianglin2:8000/eureka/,http://xianglin3:9000/eureka/
  ---
  spring:
    profiles: eureka-server2
  server:
    port: 8000
  eureka:
    instance:
      hostname: xianglin2
    client:
      service-url:
        defaultZone: http://xianglin1:7000/eureka/,http://xianglin3:9000/eureka/
  ---
  spring:
    profiles: eureka-server3
  server:
    port: 9000
  eureka:
    instance:
      hostname: xianglin3
    client:
      service-url:
        defaultZone: http://xianglin1:7000/eureka/,http://xianglin2:9000/eureka/
  ```

  在本机上启动三个 Eureka Server 的实例，相互同步注册信息。

* 为三个 hostname 添加解析

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/150929.png" alt="image-20201212150928999" style="zoom:50%;" />

* 将 spring boot 应用打包后使用：`--spring.profiles.active=`参数分别启动三个应用

  ```shell
  mvn clean package -Dmaven.test.skip=true
  
  java -jar eureka-server-0.0.1-SNAPSHOT.jar --spring.profiles.active=eureka-server
  ```

* 打开浏览器查看 Eureka Server 的启动情况

  ![image-20201212151208691](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/151208.png)

  ![image-20201212151229920](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/151230.png)

* 再次启动`course-list`和`course-price`两个模块，可以发现，三个注册中心都同步了应用的注册信息

  ![image-20201212151446100](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/151446.png)

## 服务间调用 Feign

* Feign 是一个声明式的webservice 调用客户端。
* 只需要创建接口和使用 Feign 提供的注解。
* Spring Cloud 为 Feign 提供了基于 Spring MVC 注解的支持，并且使用`HttpMessageConverters`作为默认的序列化工具。
* Spring Cloud 整合了Ribbon 和 Eureka ，使得 Feign 具有负载均衡的作用。

### 负载均衡

### 负载均衡策略

* RandomRule：随机策略
* RoundRobinRule：轮询策略
* WeightedResponseTimeRule：加权，根据平均响应时间动态加权
* 配置负载均衡策略：`course-list.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RoundRobinRule`

## 使用 Feign 调用服务

### `course-price`使用 Feign 调用`course-list`提供的`/courses`服务

* 在`course-price`模块中添加 Feign 的依赖

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
  ```

* 在`course-price`的启动类上添加`@EnableFeignClients`注解

  ```java
  @SpringBootApplication
  @EnableFeignClients
  public class CoursePriceApplication {
      public static void main(String[] args) {
          SpringApplication.run(CoursePriceApplication.class, args);
      }
  }
  ```

* 使用 Feign Client 调用`course-list`的服务

  ```java
  @FeignClient(value = "course-list")
  public interface CourseListClient {
  
      /**
       * 获取所有的课程信息
       *
       * @return 所有的课程信息
       */
      @GetMapping(value = "/courses")
      List<Course> courseList();
  }
  ```

* 在 `course-price`新增一个 REST 请求，用于验证服务间调用是否正常

  ```java
  @RestController
  public class CoursePriceController {
      @Resource
      private CourseListClient courseListClient;
  
      @GetMapping(value = "/feign/courses")
      public List<Course> courseList() {
          return courseListClient.courseList();
      }
  
  }
  ```

* 启动Eureka Server 、`course-list`和`course-price`三个模块，请求`http://localhost:8082/feign/courses`接口

  ![image-20201212154955865](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/154956.png)

### 负载均衡测试

* 在`course-list`模块新增一个方法，用于输出应用启动的端口，如下：

  ```java
  @Value("${server.port}")
  private String port;
  
  @GetMapping(value = "/rule")
  public String rule() {
    return "course-list instance port : " + port;
  }
  ```

* 在`course-price`中使用 Feign 进行调用，如下：

  ```java
  @FeignClient(value = "course-list")
  public interface CourseListClient {
      /**
       * 测试负载均衡
       *
       * @return 端口信息
       */
      @GetMapping(value = "/courses")
      String rule();
  }
  ```

  ```java
  @GetMapping(value = "/feign/port")
  public String port() {
    return courseListClient.rule();
  }
  ```

* 在调用方`course-price`中为服务配置负载均衡策略

  ```properties
  # 配置负载均衡
  course-list.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule
  ```

* 启动多个`course-list`实例，调用`/feign/port`接口，观察输出

  ![image-20201212162254699](/Users/xianglin/Library/Application Support/typora-user-images/image-20201212162254699.png)
  
  ```tex
  course-list instance port : 8091
  course-list instance port : 8071
  ```
  

### Feign 调用携带 Session

服务间的 Feign 调用是基于 HTTP 请求的，也就是说，服务间调用是无状态的，类似于 Session 信息无法在服务间传递，此时可以使用`FeignRequestInterceptor`拦截器对请求增加指定的请求头。

```java
/**
 * Feign 请求的拦截器，为请求增加请求头
 *
 * @author xianglin
 */
@EnableFeignClients
@Configuration
public class FeignRequestInterceptor implements RequestInterceptor {
    @Override
    public void apply(RequestTemplate requestTemplate) {
        // 获取到请求
        RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
        if (requestAttributes == null) {
            return;
        }
        HttpServletRequest request = ((ServletRequestAttributes) requestAttributes).getRequest();
        // 将网关的请求头附加到 Feign 调用的请求中
        Enumeration<String> headerNames = request.getHeaderNames();
        if (headerNames != null) {
            while (headerNames.hasMoreElements()) {
                String name = headerNames.nextElement();
                Enumeration<String> values = request.getHeaders(name);
                while (values.hasMoreElements()) {
                    String value = values.nextElement();
                    requestTemplate.header(name, value);
                }
            }
        }
    }
}
```



## 熔断器 Hystrix

### 雪崩效应

微服务架构中基本是多个服务层相互调用，基础服务的故障可能会导致级联故障，进而造成整个系统不可用的情况，这种情况被称为服务雪崩效应。服务雪崩效应是一种因“服务提供者”不可用导致“服务消费者”不可用，并将不可用逐渐放大的过程。

### Hystrix 特性

#### 断路器机制

当Hystrix Command 请求后端服务失败超过一定比例时，断路器会自动切换到开路（OPEN）状态，这时候所有请求会直接失败而不是发送到后端服务。断路器保持在开路状态一段时间后，自动切换到半开状态（HALF-OPEN），这时会判断下一次请求的返回状况。如果请求成功，断路器切回关闭（CLOSE）状态， 否则重新切换到开路状态。

#### FallBack

fallback 相当于降级操作。对于查询操作，我们可以实现一个 fallback 方法，当请求后端服务异常时，可以使用 fallback 的返回值。fallback 的返回值一般是设置的默认值或者缓存。

#### 资源隔离

## 熔断器使用

### 在服务调用方配置熔断器

* 在`course-price`模块中配置使用熔断器

  ```properties
  # 熔断器使用
  feign.hystrix.enabled=true
  ```

* 为`CourseListClient`创建回调类

  ```java
  /**
   * CourseList 的 回调类 调用
   *
   * @author xianglin
   */
  public class CourseListClientHystrix implements CourseListClient {
  
      @Override
      @GetMapping(value = "/courses")
      public List<Course> courseList() {
          return Collections.emptyList();
      }
  
  }
  ```

* 在`@FeignClient`注解上指定回调类

  ```java
  @FeignClient(value = "course-list", fallback = CourseListClientHystrix.class)
  ```

* 启动项目，验证熔断器

  先验证服务是否正常

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/171230.png" alt="image-20201212171230361" style="zoom:50%;" />

  将`course-list`服务停止后再次访问

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/171330.png" alt="image-20201212171330624" style="zoom:50%;" />

  后再次启动`course-list`服务后访问，发现服务恢复正常。

  


## 服务网关 Zuul

Spring Cloud Zuul路由是微服务架构的不可或缺的一部分，提供动态路由，监控，弹性，安全等的边缘服务。Zuul是Netflix出品的一个基于JVM路由和服务端的负载均衡器。

### Zuul 简单使用

* 新建一个网关项目`course-zuul`

* 添加依赖

  ```xml
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
  </dependencies>
  ```

* 配置网关相关内容

  ```yaml
  server:
    port: 8080
  spring:
    application:
      name: course-zuul
  eureka:
    client:
      service-url:
        defaultZone: http://localhost:8000/eureka/
  zuul:
    #  定义网关前缀
    prefix: /course
    #  为服务指定前缀
    routes:
      course-list:
        path: /list/*
        service-id: course-list
      course-price:
        path: /price/*
        service-id: course-price
  ```

* 创建启动类

  ```java
  /**
   * Zuul 网关启动类
   *
   * @author xianglin
   */
  @SpringBootApplication
  @EnableZuulProxy
  public class ZuulApplication {
      public static void main(String[] args) {
          SpringApplication.run(ZuulApplication.class, args);
      }
  }
  ```

* 通过网关访问`/courses`接口

  ![image-20201212174409807](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/174409.png)

### Zuul 的核心应用

Filter是Zuul的核心，用来实现对外服务的控制。Filter的生命周期有4个，分别是“PRE”、“ROUTING”、“POST”、“ERROR”，整个生命周期可以用下图来表示。

![image-20201212175248087](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/175248.png)

Zuul 中大部分功能都是基于过滤器实现的，这些过滤器对应于请求的典型生命周期：

* `PRE`：在请求被路由之前调用，可以用于身份校验，在集群中选择请求的微服务等。
* `ROUTING`：将请求路由到微服务，用于构建发送给微服务的请求，使用 Apache HTTPClient 或者Netflix Ribbon请求微服务。
* `POST`：在请求从微服务返回后执行，可以用于设置统一的响应头，收集统计信息等。
* `ERROR`：在其他阶段发生错误时执行该过滤器。
* 自定义过滤器

#### Zuul 中默认实现的过滤器

| 类型    | 顺序 | 过滤器                    | 功能                       |
| :------ | :--- | :------------------------ | :------------------------- |
| `pre`   | -3   | `ServletDetectionFilter`  | 标记处理Servlet的类型      |
| `pre`   | -2   | `Servlet30WrapperFilter`  | 包装HttpServletRequest请求 |
| `pre`   | -1   | `FormBodyWrapperFilter`   | 包装请求体                 |
| `route` | 1    | `DebugFilter`             | 标记调试标志               |
| `route` | 5    | `PreDecorationFilter`     | 处理请求上下文供后续使用   |
| `route` | 10   | `RibbonRoutingFilter`     | serviceId请求转发          |
| `route` | 100  | `SimpleHostRoutingFilter` | url请求转发                |
| `route` | 500  | `SendForwardFilter`       | forward请求转发            |
| `post`  | 0    | `SendErrorFilter`         | 处理有错误的请求响应       |
| `post`  | 1000 | `SendResponseFilter`      | 处理正常的请求响应         |

可以在配置文件中配置需要禁用的过滤器

```yaml
zuul:
  FormBodyWrapperFilter:
    pre:
      disable: true
```

#### 自定义 Filter

自定义 Filter 需要继承`com.netflix.zuul.ZuulFilter`，并实现四个方法

```java
/**
 * 自定义过滤器
 * @author xianglin
 */
@Component
public class CustomerZuulFilter extends ZuulFilter {
    @Override
    public String filterType() {
        // 定义过滤器的类型
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        // 定义同类过滤器的执行顺序，数字越小，优先级越高
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        // 判断是否需要执行此过滤器
        return false;
    }

    @Override
    public Object run() throws ZuulException {
        // 过滤器要执行的具体操作
        return null;
    }
}

```



### Zuul 的其它应用

#### 路由熔断

当后端服务出现异常时，不希望直接将异常抛出给最外层，而是希望服务可以自动进行降级。此时可以实现`org.springframework.cloud.netflix.zuul.filters.route.FallbackProvider`。其中`getRoute`方法告诉 Zuul 它负责哪个 `route`定义的熔断，`fallbackResponse`用于处理请求并返回。

```java
public interface FallbackProvider {
    String getRoute();

    ClientHttpResponse fallbackResponse(String route, Throwable cause);
}

```

为`course-price`定制熔断内容

```java
/**
 * 为 course-price 服务定制熔断内容
 *
 * @author xianglin
 */
@Component
public class CoursePriceFallbackProvider implements FallbackProvider {
    @Override
    public String getRoute() {
        return "course-price";
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        String message = "";
        if (cause != null && cause.getCause() != null) {
            message = cause.getCause().getMessage();
        }
        String finalMessage = message;
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return HttpStatus.OK.value();
            }

            @Override
            public String getStatusText() throws IOException {
                return HttpStatus.OK.name();
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream(("服务异常，请稍后！" + finalMessage).getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                return new HttpHeaders() {{
                    setContentType(MediaType.APPLICATION_JSON);
                }};
            }
        };
    }
}
```

当`course-price`服务停止后，再次调用则会返回“服务异常”的提示

![image-20201212183544655](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202012/183544.png)

>  Zuul 目前只支持服务级别的熔断，不支持具体到某个URL进行熔断。

