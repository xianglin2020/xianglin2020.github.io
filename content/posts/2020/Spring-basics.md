---
title: Spring 基础
date: 2020-10-04 18:08:53
categories: [learn]
tags: [spring]
---

# Spring

![image-20201005094800988](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/153710.png)

## Spring IOC

### IOC：控制反转

* IOC 控制反转，全称 Inverse of Control，是一种设计理念。
* 由代理人来创建与管理对象，消费者通过代理人获取对象。
* IoC 的目的是降低对象之间的直接耦合。

### DI：依赖注入

* IOC 是设计理念，是现代程序设计遵循的标准，是宏观目标。
* DI（Dependency Injection）是具体技术实现。
* DI 在 Java 中利用反射技术实现对象注入。

### Bean 配置方式

1. 基于 XML 配置 JavaBean：`applicationContext.xml`
2. 基于注解配置 JavaBean
3. 基于 Java 代码配置 JavaBean

路径表达式：

![image-20201005104352459](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/104352.png)

### 实例化 Bean 的三种方式

1. 基于对象构造方法实例化：使用`constructor-arg`标签指定构造器参数

   ```xml
   <bean id="sourApple" class="org.example.entity.Apple">
     <constructor-arg name="title" value="红富士"/>
     <constructor-arg name="color" value="红色"/>
     <constructor-arg name="origin" value="欧洲"/>
     <constructor-arg name="price" value="19.8"/>
   </bean>
   <bean id="softApple" class="org.example.entity.Apple">
     <constructor-arg index="0" value="金帅"/>
     <constructor-arg index="1" value="中国"/>
     <constructor-arg index="2" value="黄色"/>
     <constructor-arg index="3" value="10.90"/>
   </bean>
   ```

2. 基于静态工厂实例化对象

   ```java
   public class AppleStaticFactory {
       public static Apple createSweetApple() {
           return new Apple("红富士", "红色", "欧洲");
       }
   }
   ```

   ```xml
   <bean id="apple4" class="org.example.factory.AppleStaticFactory" factory-method="createSweetApple"/>
   ```

3. 基于工厂实例方法实例化对象

   ```java
   public class AppleFactoryInstance {
       public Apple createSweetApple() {
           return new Apple("红富士", "红色", "欧洲");
       }
   }
   ```

   ```xml
   <bean id="factoryInstance" class="org.example.factory.AppleFactoryInstance"/>
   <bean id="apple5" factory-bean="factoryInstance" factory-method="createSweetApple"/>
   ```

### 从 IoC 容器中获取 Bean

```java
Object getBean(String var1) throws BeansException;

<T> T getBean(String var1, Class<T> var2) throws BeansException;

Object getBean(String var1, Object... var2) throws BeansException;

<T> T getBean(Class<T> var1) throws BeansException;

<T> T getBean(Class<T> var1, Object... var2) throws BeansException;
```

### 对象依赖注入

1. 基于 setter 方法注入对象

   1. 注入静态数值

      `<property name="" value=""/>`

   2. 注入对象

      `<peoperty name="" ref=""/>`

2. 基于构造器方法注入对象

   ```xml
   <constructor-arg name="name" value="lili"/>
   <constructor-arg name="apple" ref="softApple"/>
   ```

3. 使用注解方式注入属性

   配置 Bean 扫描包

   ```xml
   <context:component-scan base-package=""/>
   ```

   使用组件注解标识类

   ```java
   @Repository 
   @Service
   @Controller
   @Component
   ```

   配置`@Resource`或`@Autowired`注入属性

   ```java
   @Resource
   @Autowired
   ```

4. 使用 JavaConfig 注入依赖

   使用`@Configuration`注解标识此为配置类

   使用`@Bean`注解创建对象交由容器管理

5. 注入 list、set、map、properties

   ```xml
   <bean id="company" class="org.example.entity.Company">
     <property name="rooms">
       <!-- ArrayList -->
       <list>
         <value>room1</value>
         <value>room2</value>
         <value>room3</value>
       </list>
     </property>
     <property name="jobs">
       <!-- LinkedHashSet -->
       <set>
         <value>job1</value>
         <value>job1</value>
         <value>job2</value>
       </set>
     </property>
     <property name="computers">
       <!-- LinkedHashMap -->
       <map>
         <entry key="lenovo" value-ref="computer"/>
         <entry key="sumsun">
           <bean class="org.example.entity.Computer">
             <property name="band" value="sumsun"/>
           </bean>
         </entry>
       </map>
     </property>
     <property name="info">
       <props>
         <prop key="address">北京市</prop>
         <prop key="website">www.baidu.com</prop>
       </props>
     </property>
   </bean>
   ```


### Bean 作用域

![image-20201005125209619](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/151713.png)

* Bean 的 scope 属性默认是 singleton 单例模式，会在容器初始化时被创建
* prototype 表示多例，容器初始化时不会创建，在使用时才创建
* lazy-init 为 true 时表示将对象的创建延迟到使用时创建，主要用于延迟初始化单例对象

### Bean 声明周期

![image-20201005152142313](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/152142.png)

1. 使用 XML 定义的`init-method`和`destroy-method`方法

2. 实现`InitializingBean`和 `DisposableBean`中的回调方法

   ```java
   public class Order implements InitializingBean, DisposableBean {
       private Float price;
       private Integer quantity;
       private Float total;
   
       public Order() {
           System.out.println("创建 Order:" + this);
       }
   
       public void initMethod() {
           System.out.println("执行 initMethod");
           this.total = this.price * this.quantity;
       }
   
       public void destroyMethod() {
           System.out.println("执行 destroyMethod");
       }
   
       @Override
       public void afterPropertiesSet() throws Exception {
           System.out.println("执行 afterPropertiesSet");
       }
   
       @Override
       public void destroy() throws Exception {
           System.out.println("执行 destroy");
       }
   }
   ```

   ```xml
   <bean id="order1" class="org.example.entity.Order" init-method="initMethod" destroy-method="destroyMethod">
     <property name="price" value="19.3"/>
     <property name="quantity" value="1000"/>
   </bean>
   ```

### 基于注解配置 IoC 容器

Spring 从 2.5 开始支持注解来配置依赖注入

#### 四种组件类型注解

![image-20201005160441887](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/160442.png)

#### 两类自动装配注解

![image-20201005162052486](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/162052.png) 

* `@Autowried`注解在属性和 setter 方法的不同

* `@Autowried`注解在存在多个类型相同实例时指定注入实例的方法

  1. 在需要被注入实例上增加`@Primary`

  2. 在`@Autowired`注解的字段或方法上增加`@Qualifier`注解，指定需要注入的对象

     `@Qualifier`优先级大于`@Primary`

#### 元数据注解

![image-20201005165710646](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/165731.png)

### 基于Java 的配置

![image-20201005170901898](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/184249.png)

### Spring 与 JUnit 整合

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = Config.class)
public class UserServiceTest {
    @Resource
    private UserService userService;


    @Test
    public void test1() {
        userService.test();
    }
}
```

## Spring AOP

### AOP 关键概念

* AspectJ一种基于Java 平台的面向切面编程的语言
* Spring 使用aspectjweaver实现类与方法匹配
* SpringAOP 使用代理模式实现对象运行时扩展

![image-20201005182648553](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/182649.png)

### AOP 配置过程

1. 引入aspectj依赖

   ```xml
   <dependency>
     <groupId>org.aspectj</groupId>
     <artifactId>aspectjweaver</artifactId>
     <version>1.9.5</version>
   </dependency>
   ```

2. 编写切面类和切面方法

3. 配置Aspect Bean

   ```xml
   <bean id="methodChecker" class="org.example.aspect.MethodChecker"/>
   ```

4. 定义 pointcut

5. 定义切面类、配置 advice

   ```xml
   <aop:config>
     <aop:pointcut id="pointcut" expression="execution(* *(..))"/>
     <aop:aspect ref="methodChecker">
       <aop:around method="check" pointcut-ref="pointcut"/>
     </aop:aspect>
   </aop:config>
   ```


### JoinPoint核心方法

```java
Object getTarget();

Object[] getArgs();

Signature getSignature();
```

### PointCut切点表达式

`execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)throws-pattern?)`

`execution(public * org.example..*.*(..))`

* `*` 通配符
* `..` 包通配符
* `(..)` 参数通配符

[aop-pointcuts-example](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-pointcuts-examples)

### 五种通知类型

![image-20201005220543321](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/220544.png)

### 使用注解配置 AOP

`@Around`、`@Before`、`@After`、`@AfterReturning`、`@AfterThrowing`

### Spring AOP 的实现原理

* Spring 基于代理模式实现功能扩展
  1. 目标类实现接口，通过 JDK动态代理实现功能扩展 `com.sun.proxy.$Proxy20`
  2. 目标类没有实现接口，通过 CGLib组件实现功能扩展 `EmployeeService$$EnhancerBySpringCGLIB$$ed94134b`

## Spring JDBC

### Spring JDBC

* Spring JDBC 是Spring 框架对用于处理关系型数据库的模块
* Spring JDBC对 JDBC API进行封装
* JDBCTemplate是 SpringJDBC 的核心类，提供数据的 CURD 操作

### Spring JDBC使用步骤

1. 引入`spring-jdbc`依赖

   ```xml
   <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-context</artifactId>
   </dependency>
   <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-jdbc</artifactId>
   </dependency>
   <dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
     <version>8.0.20</version>
   </dependency>
   ```

2. 在`applicationContext.xml`中配置数据源

   ```xml
   <!-- 数据源 -->
       <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
         <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
         <property name="url" value="jdbc:mysql://122.51.48.52:3306/imooc_demo?useSSL=false&amp;useUnicode=true"/>
         <property name="username" value=""/>
         <property name="password" value=""/>
   </bean>
   
   <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
     <property name="dataSource" ref="dataSource"/>
   </bean>
   
   <bean id="employeeDao" class="org.example.dao.EmployeeDao">
     <property name="jdbcTemplate" ref="jdbcTemplate"/>
   </bean>
   ```

3. 在 dao 对象中注入`JdbcTemplate`依赖

   ```java
   public class EmployeeDao {
       private JdbcTemplate jdbcTemplate;
   
       public JdbcTemplate getJdbcTemplate() {
           return jdbcTemplate;
       }
   
       public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
           this.jdbcTemplate = jdbcTemplate;
       }
   }
   
   ```

### Spring JDBC 事务管理

1. 编程式事务 `TransactionManger`

   ```xml
   <!-- 事务管理器 -->
   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
     <property name="dataSource" ref="dataSource"/>
   </bean>
   ```

   ```java
   // 事务标准配置
   TransactionDefinition definition = new DefaultTransactionDefinition();
   // 开始事务，返回事务状态
   TransactionStatus transaction = transactionManager.getTransaction(definition);
   // 提交事务
   transactionManager.commit(transaction);
   // 回滚事务
   transactionManager.rollback(transaction);
   ```

2. 声明式事务

   ```xml
   <tx:advice id="txAdvice" transaction-manager="transactionManager">
     <tx:attributes>
       <tx:method name="batchImport"/>
       <tx:method name="find*" propagation="NOT_SUPPORTED" read-only="true"/>
       <tx:method name="*" propagation="NOT_SUPPORTED" read-only="true"/>
     </tx:attributes>
   </tx:advice>
   
   <aop:config>
     <aop:pointcut id="pointcut" expression="execution(* org.example..*Service.*(..))"/>
     <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>
   </aop:config>
   ```

### 事务传播行为

* 事务传播行为是指多个拥有事务控制的方法在嵌套调用时的事物控制方式

  `<tx:method name="batchImport" propagation="REQUIRED"/>`

  `@Transactional(propagation = Propagation.REQUIRED)`

* 七种事务传播行为：`org.springframework.transaction.annotation.Propagation`

  ![image-20201006123215905](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/123216.png)
