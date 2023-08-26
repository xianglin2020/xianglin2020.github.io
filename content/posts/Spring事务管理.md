---
title: Spring 事务管理
date: 2020-11-29 10:57:54
categories: [learn]
tags: [spring]
---

# Spring 事务管理

## 事务的基础概念

### 什么是事务

* 事务是指逻辑上的一组操作，要么一起成功、要么一起失败。

### 事务的特性

* 原子性：原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
* 一致性：一致性指事务前后数据的完整性必须保持一致。
* 隔离性：隔离性是指多个用户并发访问数据库时，一个用户的事务不能被其他用户的事务所干扰，多个并发事务之间数据要相互隔离。
* 持久性：持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，即使数据库发生故障也不应该对其有影响。

## 事务隔离级别

* `READ_UNCOMMITED`：允许读取还未提交的数据，可能导致脏读、不可重复读、幻读。

1. `READ_COMMITTED`：允许读取并发事务已提交的数据，可防止脏读，但不可重复读、幻读仍可能发生。
2. `REPEATABLE_READ`： 在统一事务中读取的数据保持一致，即使其他事务对其修改并提交。
3. `SERIALIZABLE`：将并发事务串行化，解决一切问题。

## Spring 接口介绍

### 三个抽象接口

* 事务管理器：`PlatformTransactionManager`
* 事务定义信息（隔离级别、传播行为、超时、只读）：`TransactionDefinition`
* 事务运行状态：`TransactionStatus`

### `FlatformTransactionManager`

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202011/112045.png" alt="PlatformTransactionManager" style="zoom:50%;" />

### `TranscationDefinition`

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202011/111550.png" alt="transactiondefinition" style="zoom:50%;" />

#### 事务的传播行为

| 事务传播行为                | 描述                                     |
| --------------------------- | ---------------------------------------- |
| `PROPAGATION_REQUIRED`      | 支持当前事务，如果不存在就新建一个       |
| `PROPAGATION_SUPPORTS`      | 支持当前事务，如果不存在就不使用事务     |
| `PROPAGATION_MANDATORY`     | 支持当前事务，如果不存在则抛出异常       |
| `PROPAGATION_REQUIRES_NEW`  | 创建一个新事务，如果有事务存在则挂起     |
| `PROPAGATION_NOT_SUPPORTED` | 以非事务方式运行，如果有事务存在则挂起   |
| `PROPAGATION_NEVER`         | 以非事务方式运行，如果存在事务则抛出异常 |
| `PROPAGATION_NESTED`        | 如果当前事务存在，则嵌套事务执行         |

### `TransactionStatus`

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202011/122018.png" alt="TransactionStatus" style="zoom:33%;" />

## Spring 事务管理操作

### 准备转账案例

#### 表设计

```sql
CREATE TABLE `account`
(
    `id`    int(11)     NOT NULL,
    `name`  varchar(50) NOT NULL,
    `money` decimal(10, 2) DEFAULT NULL,
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
  
INSERT INTO imooc_demo.account (id, name, money) VALUES (1, 'a', 1000.00);
INSERT INTO imooc_demo.account (id, name, money) VALUES (2, 'b', 1000.00);
INSERT INTO imooc_demo.account (id, name, money) VALUES (3, 'c', 1000.00);  
```

#### 实现 DAO 层

```java
public interface AccountDao {

    void accountAdd(String name, BigDecimal decimal);

    void accountSub(String name, BigDecimal decimal);
}

public class AccountDaoImpl implements AccountDao {
    private JdbcTemplate jdbcTemplate;

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public void accountAdd(String name, BigDecimal decimal) {
        String sql = "update account set money = money + ? where name = ?";
        this.jdbcTemplate.update(sql, decimal, name);
    }

    @Override
    public void accountSub(String name, BigDecimal decimal) {
        String sql = "update account set money = money - ? where name = ?";
        this.jdbcTemplate.update(sql, decimal, name);
    }
}
```

#### 实现 Service 层

```java
public interface AccountService {
    void transfer(String from, String to, BigDecimal money);
}

public class AccountServiceImpl implements AccountService {
    private AccountDao accountDao;
    
    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    @Override
    public void transfer(String from, String to, BigDecimal money) {
        accountDao.accountSub(from, money);
        accountDao.accountAdd(to, money);
    }
}
```

#### 使用 XML 方式声明对象

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
  <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
  <property name="url" value="jdbc:mysql://hostip:port/dbname"/>
  <property name="username" value="name"/>
  <property name="password" value="password"/>
</bean>

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
  <property name="dataSource" ref="dataSource"/>
</bean>

<bean id="accountDao" class="dao.AccountDaoImpl">
  <property name="jdbcTemplate" ref="jdbcTemplate"/>
</bean>

<bean id="accountService" class="service.AccountServiceImpl">
  <property name="accountDao" ref="accountDao"/>
</bean>

   
```

#### 测试转账效果

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class AccountServiceImplTest {

    @Resource
    private AccountService accountService;

    @Test
    public void transfer() {
        accountService.transfer("a", "b", BigDecimal.valueOf(200));
    }
}
```

### 编程式的事务管理

配置事务管理器

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource"/>
</bean>

<!-- 用于简化事务管理的代码 -->
<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
  <property name="transactionManager" ref="transactionManager"/>
</bean>
```

#### 在业务层中注入事务管理模板

```java
public class AccountServiceImpl implements AccountService {
  private TransactionTemplate transactionTemplate;
  private AccountDao accountDao;
  // ...
  public void setTransactionTemplate(TransactionTemplate transactionTemplate) {
    this.transactionTemplate = transactionTemplate;
  }
  // ...
}

```

```xml
<bean id="accountService" class="person.xiangln.transaction.service.AccountServiceImpl">
  <property name="accountDao" ref="accountDao"/>
  <property name="transactionTemplate" ref="transactionTemplate"/>
</bean>
```

#### 使用编程式事务

```java
public void transfer(String from, String to, BigDecimal money) {
  transactionTemplate.executeWithoutResult(transactionStatus -> {
    accountDao.accountSub(from, money);
    int a = 1 / 0;
    accountDao.accountAdd(to, money);
  });
}
```

### 声明式事务管理

#### 基于 Aspectj 的 XML 配置

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource"/>
</bean>

<tx:advice id="txAdvice">
  <tx:attributes>
    <tx:method name="transfer"/>
  </tx:attributes>
</tx:advice>

<aop:config >
  <aop:pointcut id="pointcut1" expression="execution(* person.xiangln.transaction.service.AccountService+.*(..))"/>
  <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut1"/>
</aop:config>
```

#### 基于注解的事务管理

```xml
<!-- 基于注解的事务配置 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource"/>
</bean>

<!--开启注解事务-->
<tx:annotation-driven/>
```

```java
@Override
@Transactional
public void transfer(String from, String to, BigDecimal money) {
  accountDao.accountSub(from, money);
  int a = 1 / 0;
  accountDao.accountAdd(to, money);
}
```

