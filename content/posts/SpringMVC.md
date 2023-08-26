---
title: SpringMVC
date: 2020-10-06 13:16:26
categories: [learn]
tags: [spring mvc]
---

# Spring MVC

## Spring MVC 环境搭建

1. 引入 spring-mvc 相关依赖

   ```xml
   <dependencies>
     <dependency>
       <groupId>javax.servlet</groupId>
       <artifactId>servlet-api</artifactId>
       <version>2.5</version>
       <scope>provided</scope>
     </dependency>
     <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-webmvc</artifactId>
     </dependency>
   </dependencies>
   ```

2. 在`web.xml`中配置`DispatcherServlet`

   ```xml
   <servlet>
     <servlet-name>springmvc</servlet-name>
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
     <init-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath:applicationContext.xml</param-value>
     </init-param>
     <load-on-startup>0</load-on-startup>
   </servlet>
   <servlet-mapping>
     <servlet-name>springmvc</servlet-name>
     <url-pattern>/</url-pattern>
   </servlet-mapping>
   ```

3. 创建并配置`applicationContext.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="http://www.springframework.org/schema/beans 
          http://www.springframework.org/schema/beans/spring-beans.xsd 
          http://www.springframework.org/schema/context 
          https://www.springframework.org/schema/context/spring-context.xsd 
          http://www.springframework.org/schema/mvc 
          https://www.springframework.org/schema/mvc/spring-mvc.xsd">
   
       <context:component-scan base-package="person.xianglin.springmvc"/>
       <mvc:annotation-driven/>
       <mvc:default-servlet-handler/>
   </beans>
   ```

4. 开发业务代码

## SpringMVC 基础及数据绑定

### URL 绑定

`@RequestMapping`

### 接收请求参数

* 方法参数接收请求数据

  `@RequestParam`

* JavaBean 接收请求参数

* 日期类型转换

  `@DateTimeFormat`

  实现`org.springframework.core.convert.converter.Converter`自定义转换

  ```java
  public class DateConverter implements Converter<String, Date> {
      @Override
      public Date convert(String s) {
          DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
          Date parse = null;
          try {
              parse = dateFormat.parse(s);
              return parse;
          } catch (ParseException e) {
              return null;
          }
      }
  }
  ```

  ```xml
  <context:component-scan base-package="person.xianglin.springmvc"/>
  <mvc:default-servlet-handler/>
  <mvc:annotation-driven conversion-service="conversionService"/>
  
  <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="converters">
      <set>
        <bean class="person.xianglin.springmvc.convert.DateConverter"/>
      </set>
    </property>
  </bean>
  ```

### Spring MVC 中文乱码

* Get 请求：`server.xml`中增加`URIEncoding`属性（Tomcat8.0 之前）

  ```xml
  <Connector port="8080" protocol="HTTP/1.1"
                 connectionTimeout="20000"
                 redirectPort="8443" URIEncoding="UTF-8"/>
  ```

* Post 请求：`web.xml`中配置`CharacterEncodingFilter`

  ```xml
  <filter>
    <filter-name>characterFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

* Response 响应：Spring 配置`StringHttpMessageConverter`

  ```xml
  <mvc:annotation-driven>
    <mvc:message-converters>
      <bean class="org.springframework.http.converter.StringHttpMessageConverter">
        <property name="supportedMediaTypes">
          <list>
            <value>application/json;charset=utf-8</value>
            <value>text/html;charset=utf-8</value>
          </list>
        </property>
      </bean>
    </mvc:message-converters>
  </mvc:annotation-driven>
  ```

### 响应输出结果

* `@ResponseBody`

* `ModelAndView`

  属性保存在当前请求 request 对象中

  modelAndView 使用请求转发完成页面跳转

  使用`redirect:index.html`完成重定向

* Spring 与模板引擎整合

  1. 引入 freemaker 依赖

     ```xml
     <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context-support</artifactId>
     </dependency>
     <dependency>
       <groupId>org.freemarker</groupId>
       <artifactId>freemarker</artifactId>
       <version>2.3.30</version>
     </dependency>
     ```

  2. 配置`FreeMarkerViewResolver`

     ```xml
     <bean id="viewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
       <property name="contentType" value="text/html;charset=UTF-8"/>
       <property name="suffix" value=".ftl"/>
     </bean>
     ```

  3. 配置`FreeMarkerConfig`

     ```xml
     <!-- NoSuchBeanDefinitionException: No qualifying bean of type 'org.springframework.web.servlet.view.freemarker.FreeMarkerConfig' available -->
     <bean id="freeMarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
       <property name="templateLoaderPath" value="/WEB-INF/ftl/"/>
       <property name="freemarkerSettings">
         <props>
           <prop key="defaultEncoding">UTF-8</prop>
         </props>
       </property>
     </bean>
     ```
  

## Restful风格的应用

* REST：表现层状态转换，资源在网络中以某种形式进行状态转换
* RESTFul 开发规范：
  1. 使用 URL 作为用户交互入口
  2. 明确的语义规范，使用不同的 HTTP 方法完成对应的操作
  3. 只返回数据，不包含展示

### SpringMVC RESTFul 使用

* `@RestController` 创建 RestFul 风格控制器对象

* `@PathVariable` 路径变量注解

* 非简单请求JavaBean无法获取参数问题

  在`web.xml`中配置`FormContentFilter`过滤器

  ```xml
  <filter>
    <filter-name>formContextFilter</filter-name>
    <filter-class>org.springframework.web.filter.FormContentFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>formContextFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

* jackson 中对日期格式的注解`@JsonFormat`

  ```java
  @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss SSS",timezone = "Asia/Shanghai")
  private Date date;
  ```

### 跨域问题

* 同源策略阻止从一个域加载脚本去获取另一个域上的资源
* 只要协议、域名、端口有一个不同，都视为不同的域
* `Access-Control-Allow-Origin`

#### HTML 中允许跨域的标签

* `<img>` 图片
* `<srcipt>` JS 脚本
* `link` CSS 样式

#### SpringMVC 解决跨域访问问题

* `@CrosOrigin` Controller 跨域注解

  | 字段             | 描述                       |值|
  | ---------------- | -------------------------- | -------------------------- |
  | origins          | 被允许跨区请求的源列表     ||
  | allowedHeaders   | 被允许在实际请求中的请求头 |`Access-Control-Allow-Headers`|
  | exposedHeaders   |                            |`Access-Control-Expose-Headers`|
  | methods          | 支持的HTTP请求方法的列表   ||
  | allowCredentials |                            |`Access-Control-Allow-Credentials`|
  | maxAge           | 预检响应缓存持续时间       |`Access-Control-Max-Age`|

* `<mvc:cros>` 全局跨域设置

  ```xml
  <mvc:cors>
      <mvc:mapping path="/restful/**" allowed-origins="http://127.0.0.1,http://example.com" max-age="3600"/>
  </mvc:cors>
  ```

## Spring 拦截器

* 拦截器（Interceptor）用于对请求进行前置、后置过滤
* 拦截器基于 Spring AOP 实现

### Spring 拦截器开发流程

1. 引入`servlet-api`

   ```xml
   <dependency>
     <groupId>javax.servlet</groupId>
     <artifactId>javax.servlet-api</artifactId>
     <version>3.1.0</version>
     <scope>provided</scope>
   </dependency>
   ```

2. 继承`org.springframework.web.servlet.HandlerInterceptor`实现拦截器逻辑

3. 配置拦截器

   ```xml
   <mvc:interceptors>
     <mvc:interceptor>
       <mvc:mapping path="/**"/>
       <bean class="person.xianglin.springrestful.interceptor.MyInterceptor"/>
     </mvc:interceptor>
   </mvc:interceptors>
   ```


### HandlerInterceptor 接口

| 方法            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| preHandle       | 前置执行处理：返回 true 允许执行后续操作，返回 false 则立即返回 |
| postHandle      | 目标资源已被 SpringMVC框架处理                               |
| afterCompletion | 响应文本已经产生                                             |

### 拦截器处理 URL

* 资源过滤

  ```xml
  <mvc:exclude-mapping path="/**.ico"/>
  <mvc:exclude-mapping path="/resources/**"/>
  ```

* 多个拦截器执行顺序

  ![image-20201007184030414](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/184032.png)

### Spring MVC 处理流程

![image-20201007191121154](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/191124.png)

## SSM 整合过程

### Spring与 SpringMVC环境配置

1. 在`pom.xml`中引入相应依赖

   ```xml
   <dependencies>
     <!-- SpringMVC 依赖 -->
     <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-webmvc</artifactId>
     </dependency>
     <!-- Freemarker 依赖 -->
     <dependency>
       <groupId>org.freemarker</groupId>
       <artifactId>freemarker</artifactId>
       <version>2.3.30</version>
     </dependency>
     <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context-support</artifactId>
     </dependency>
     <!-- Jackson 依赖 -->
     <dependency>
       <groupId>com.fasterxml.jackson.core</groupId>
       <artifactId>jackson-core</artifactId>
       <version>2.11.1</version>
     </dependency>
     <dependency>
       <groupId>com.fasterxml.jackson.core</groupId>
       <artifactId>jackson-annotations</artifactId>
       <version>2.11.1</version>
     </dependency>
     <dependency>
       <groupId>com.fasterxml.jackson.core</groupId>
       <artifactId>jackson-databind</artifactId>
       <version>2.11.1</version>
     </dependency>
   </dependencies>
   ```

2. 在`web.xml`中声明`ContextLoaderListener`和`DispatcherServlet`

   `ContextLoaderListener`用于初始化 IOC 容器，`DispatcherServlet`用于初始化 SpringMVC 相关内容

   ```xml
   <!-- 将 Spring 的配置信息以 context-param 的方式注册，并让ContextLoaderListener 进行读取-->
   <context-param>
     <param-name>contextConfigLocation</param-name>
     <param-value>classpath:applicationContext.xml</param-value>
   </context-param>
   <listener>
     <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
   </listener>
   
   <servlet>
     <servlet-name>springmvc</servlet-name>
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
     <!-- 指定 spring-mvc 配置文件路径 -->
     <init-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath:spring-mvc.xml</param-value>
     </init-param>
     <!-- 让DispatcherServlet在应用启动时初始化 -->
     <load-on-startup>1</load-on-startup>
   </servlet>
   <servlet-mapping>
     <servlet-name>springmvc</servlet-name>
     <url-pattern>/</url-pattern>
   </servlet-mapping>
   ```

3. 处理请求响应中的中文乱码问题

   在`web.xml`中声明`CharacterEncodingFilter`用于处理请求中的乱码问题

   ```xml
   <filter>
     <filter-name>characterFilter</filter-name>
     <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
     <init-param>
       <param-name>encoding</param-name>
       <param-value>UTF-8</param-value>
     </init-param>
   </filter>
   <filter-mapping>
     <filter-name>characterFilter</filter-name>
     <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

   在`spring-mvc.xml`中使用`<mvc:message-converters>`标签处理响应中的乱码问题

   ```xml
   <mvc:message-converters>
     <bean class="org.springframework.http.converter.StringHttpMessageConverter">
       <property name="supportedMediaTypes">
         <list>
           <value>text/html;charset=utf-8</value>
           <value>application/json;charset=utf-8</value>
         </list>
       </property>
     </bean>
   </mvc:message-converters>
   ```

4. 创建`spring-mvc.xml`配置 spring mvc相关配置

   ```xml
   <!-- 指定扫描 controller 包 -->
   <context:component-scan base-package="person.xianglin.reader.controller"/>
   <!-- 启用 mvc 注解开发 -->
   <mvc:annotation-driven>
     <mvc:message-converters>
       <bean class="org.springframework.http.converter.StringHttpMessageConverter">
         <property name="supportedMediaTypes">
           <list>
             <value>text/html;charset=utf-8</value>
             <value>application/json;charset=utf-8</value>
           </list>
         </property>
       </bean>
     </mvc:message-converters>
   </mvc:annotation-driven>
   <!-- 处理静态资源：将静态资源转发至 Tomcat 的默认Servlet 中 -->
   <mvc:default-servlet-handler/>
   ```

5. 在`spring-mvc.xml`中整合模板引擎（Freemarker），同时在创建指定目录`WEB-INF/ftl`

   ```xml
   <bean id="freeMarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
     <property name="templateLoaderPath" value="/WEB-INF/ftl/"/>
     <property name="freemarkerSettings">
       <props>
         <prop key="defaultEncoding">UTF-8</prop>
       </props>
     </property>
   </bean>
   <bean id="viewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
     <property name="contentType" value="text/html;charset=utf-8"/>
     <property name="suffix" value=".ftl"/>
   </bean>
   ```

   如果使用 JSP ，可配置`InternalResourceViewResolver`

   ```xml
   <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
     <property name="prefix" value="/WEB-INF/jsp/"/>
     <property name="suffix" value=".jsp"/>
   </bean>
   ```

6. 编写测试用的`TestController`

   ```java
   @Controller
   @RequestMapping(value = "/test")
   public class TestController {
       @GetMapping(value = "/t1")
       public ModelAndView test1() {
           ModelAndView modelAndView = new ModelAndView("test");
           modelAndView.addObject("test", "/test/t1");
           return modelAndView;
       }
   
       @GetMapping(value = "/t2")
       @ResponseBody
       public Map<String, Object> test2() {
           Map<String, Object> map = new HashMap<>(3);
           map.put("success", true);
           return map;
       }
   }
   ```

### Spring 与 MyBatis 整合

1. 在`pom.xml`中引入依赖

   ```xml
   <dependencies>
     <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-jdbc</artifactId>
     </dependency>
     <dependency>
       <groupId>org.mybatis</groupId>
       <artifactId>mybatis</artifactId>
       <version>3.5.5</version>
     </dependency>
     <dependency>
       <groupId>org.mybatis</groupId>
       <artifactId>mybatis-spring</artifactId>
       <version>2.0.5</version>
     </dependency>
     <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>8.0.20</version>
     </dependency>
     <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>druid</artifactId>
       <version>1.1.23</version>
     </dependency>
   </dependencies>
   ```

2. 在`applicationContext.xml`中配置数据源与连接池

   ```xml
   <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
     <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
     <property name="url" value="jdbc:mysql://122.51.48.52:3306/imooc_reader?useSSL=false&amp;useUnicode=true"/>
     <property name="username" value=""/>
     <property name="password" value=""/>
     <property name="initialSize" value="5"/>
     <property name="maxActive" value="20"/>
   </bean>
   ```

3. 在`applicationContext.xml`中配置`SqlSessionFactoryBean`和`MapperScannerConfigurer`，用于将 MyBatis 的 SQLSession 和 Mapper 交于 Spring 管理

   ```xml
   <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
     <property name="dataSource" ref="dataSource"/>
     <property name="mapperLocations" value="classpath:mappers/*.xml"/>
     <property name="configLocation" value="classpath:mybatis-config.xml"/>
   </bean>
   
   <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
     <property name="basePackage" value="person.xianglin.reader.mapper"/>
   </bean>
   ```

### 配置其他组件

* 配置logback日志输出

  引入logback依赖

  ```xml
  <dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
  </dependency>
  ```

  创建`logback.xml`配置文件

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <configuration>
      <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
          <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
              <pattern>%d{HH:mm:ss} %-5level [%thread] %logger{30} -%msg%n</pattern>
              <charset>UTF-8</charset>
          </encoder>
      </appender>
      <root level="debug">
          <appender-ref ref="console"/>
      </root>
  </configuration>
  ```

* 声明式事务配置

  在`applicationContext.xml`配置

  ```xml
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
  </bean>
  <tx:annotation-driven/>
  ```

* 整合 JUnit 单元测试

  引入依赖

  ```xml
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13</version>
    <scope>test</scope>
  </dependency>
  ```

  编写测试类

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(locations = {"classpath:applicationContext.xml"})
  public class TestServiceTest {
  
      @Resource
      private TestService testService;
  
      @Test
      public void insert() {
          testService.insert();
      }
  }
  ```

### 配置 MyBatis-Plus

1. 在`pom.xml`中引入依赖

   ```xml
   <dependency>
     <groupId>com.baomidou</groupId>
     <artifactId>mybatis-plus</artifactId>
     <version>3.4.0</version>
   </dependency>
   ```

2. 调整`applicationContext.xml`中的`SqlSessionFactory`

   ```xml
   <!--    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">-->
   <bean id="sqlSessionFactory" class="com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean">
     <property name="dataSource" ref="dataSource"/>
     <property name="mapperLocations" value="classpath:mappers/*.xml"/>
     <property name="configLocation" value="classpath:mybatis-config.xml"/>
   </bean>
   ```

3. 在`mybatis-config.xml`中增加分页插件

   ```xml
   <plugins>
     <plugin interceptor="com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor"/>
   </plugins>
   ```

### MyBatis-Plus 使用步骤

1. 创建实体类

   ```java
   @Data
   @TableName("test")
   public class Test {
       @TableId(type = IdType.AUTO)
       @TableField("id")
       private Integer id;
       @TableField("context")
       private String context;
   }
   ```

   * `@TableName`
   * `@TableId`
   * `@TableField`

2. 创建 Mapper 接口和MapperXML

   继承自`com.baomidou.mybatisplus.core.mapper.BaseMapper`

   ```java
   public interface TestMapper extends BaseMapper<Test> {
       void insertSample();
   }
   ```

3. 注入 Mapper对象，通过内置 API 完成 CURD 操作

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration(locations = {"classpath:applicationContext.xml"})
   public class TestMapperTest {
       @Resource
       private TestMapper testMapper;
   
       @org.junit.Test
       public void testInsert() {
           Test test = new Test();
           test.setContext("MyBatis-Plus 测试");
           testMapper.insert(test);
       }
   
       @org.junit.Test
       public void testUpdate() {
           Test test = testMapper.selectById(8);
           test.setContext("测试 1111");
           testMapper.updateById(test);
       }
   
       @org.junit.Test
       public void testSelectList() {
           QueryWrapper<Test> queryWrapper = new QueryWrapper<>();
           queryWrapper.ge("id", 8);
           List<Test> selectList = testMapper.selectList(queryWrapper);
           Assert.assertEquals(selectList.size(), 1);
       }
   }
   ```

   * `insert`、`updateById`、`deleteById`、`selectById`、`selectList`、`selectPage`

### Kaptcha 验证码

1. 引入依赖

   ```xml
   <dependency>
     <groupId>com.github.penggle</groupId>
     <artifactId>kaptcha</artifactId>
     <version>2.3.2</version>
   </dependency>
   ```

2. 配置`com.google.code.kaptcha.Producer`实现类

   ```xml
   <bean id="defaultKaptcha" class="com.google.code.kaptcha.impl.DefaultKaptcha">
     <property name="config">
       <bean class="com.google.code.kaptcha.util.Config">
         <constructor-arg name="properties">
           <props>
             <prop key="kaptcha.border">no</prop>
             <prop key="kaptcha.image.width">120</prop>
             <prop key="kaptcha.textproducer.font.color">blue</prop>
             <prop key="kaptcha.textproducer.font.size">40</prop>
             <prop key="kaptcha.textproducer.char.length">4</prop>
           </props>
         </constructor-arg>
       </bean>
     </property>
   </bean>
   ```

3. 在 controller 中使用

   ```java
   @Controller
   public class KaptchaController {
       @Resource
       private Producer defaultKaptcha;
   
       @GetMapping(value = "/verify_code")
       public void createVerifyCode(HttpServletRequest request, HttpServletResponse response) throws IOException {
           response.setDateHeader("Expires", 0);
           response.setHeader("Cache-Control", "no-store,no-cache,must-revalidate");
           response.setHeader("Cache-Control", "post-check=0;pre-check=0");
           response.setHeader("Pragma", "no-cache");
           response.setContentType("image/png");
   
           String verifyCode = defaultKaptcha.createText();
           request.getSession().setAttribute("kaptchaVerifyCode", verifyCode);
           BufferedImage kaptchaImage = defaultKaptcha.createImage(verifyCode);
           ServletOutputStream outputStream = response.getOutputStream();
           ImageIO.write(kaptchaImage, "png", outputStream);
           outputStream.flush();
           outputStream.close();
       }
   }
   ```

   
