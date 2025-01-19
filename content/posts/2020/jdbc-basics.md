---
title: jdbc 基础
description: jdbc 基础知识学习。
summary: jdbc 基础知识学习。
date: 2020-09-06 22:55:29
categories: [learn]
tags: [jdbc]
---

# JDBC 基础

## IDEA

* IDEA 的[下载地址](https://www.jetbrains.com/idea/)：https://www.jetbrains.com/idea/
* IDEA 使用视频教程：[IDEA 神器使用技巧](https://www.imooc.com/learn/924)
* IDEA 使用教程：[IntelliJ-IDEA-Tutorial](https://github.com/judasn/IntelliJ-IDEA-Tutorial)

## JDBC

### JDBC 简介

* Java DataBase Connectivity

* JDBC 能让 Java 操作数据库

  ![image-20200908215723835](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/215724.png)

* JDBC 的优点

  1. 统一的 API，提供一致的开发过程
  2. 抑郁学习，代码结构稳定
  3. 功能强大，执行效率高，能用于处理海量的数据

### JDBC 使用步骤

1. <s>加载并注册 JDBC 驱动</s>
2. 创建数据库连接(`Connection`)
3. 创建 Statement 对象
4. 执行查询操作(`ResultSet`)
5. 关闭连接对象

```java
public static void main(String[] args) throws Exception {
  // 1. 加载并注册 JDBC 驱动
  //        Class.forName("com.mysql.cj.jdbc.Driver");

  // 2. 获取数据库连接
  Connection connection = DriverManager.getConnection(
    "jdbc:mysql://122.51.48.52:3306/imooc_demo",
    "xianglin",
    "950915"
  );
  // 3. 创建 Statement 对象，执行查询操作
  Statement statement = connection.createStatement();
  ResultSet resultSet = statement.executeQuery("select * from employee");

  // 4. 遍历查询结果集
  while (resultSet.next()) {
    int eno = resultSet.getInt(1);
    String ename = resultSet.getString("ename");
    float salary = resultSet.getFloat("salary");
    String dname = resultSet.getString("dname");
    System.out.println(dname + "--" + eno + "--" + ename + "--" + salary);
  }
  // 5. 关闭连接，释放资源
  resultSet.close();
  statement.close();
  connection.close();
}
```

### JDBC 细节

* `DriverManager`

  1. 用于注册、管理 JDBC 驱动程序
  2. `DriverManager.getConnection()`返回数据库连接
  3. 返回值`Connection`对象，对应数据库的物理网络连接

* `Connection`

  1. 用于 JDBC 与数据库的网络通信对象
  2. `java.sql.Connection`是一个接口，具体由驱动厂商实现
  3. 所有数据库操作都建立在 Connection对象基础上

* MySQL 连接字符串

  `jdbc:mysql://[hostip][:port]/databaseName?param`

  参数列表采用 url 编码格式

  | 参数名                  | 建议值        | 说明                         |
  | ----------------------- | ------------- | ---------------------------- |
  | `useSSL`                |               | 是否启用 ssl                 |
  | `useUnicode`            | true          | 启用 Unicode 编码传输数据    |
  | `characterEncoding`     | UTF-8         | 使用 UTF-8 编码传输数据      |
  | `serverTimeZone`        | Asia/Shanghai | 使用东 8 时区                |
  | allowPublicKeyRetrieval | true          | 运行从客户端获取公钥加密传输 |

### PreparedStatement

* SQL 注入攻击是未对原始 SQL 中的铭感字符做特殊处理
* `PreparedStatement`是`Statement`的子接口
* `PreparedStatement`对SQL 进行参数化，防止 SQL 注入

## 数据库连接池

### 阿里巴巴 Druid连接池

* Druid 是阿里巴巴开源的数据库连接池组件。

* Druid 能对数据库连接进行有效的管理与重用，最大化程序执行效率。

* 连接池负责创建管理连接，应用程序只管使用和归还。

* Druid 使用方式

  1. 获取 jar 文件

     ```xml
     <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>druid</artifactId>
       <version>1.1.23</version>
     </dependency>
     ```

  2. 创建配置文件：`db.properties`

     ```properties
     driverClass=com.mysql.cj.jdbc.Driver
     url=jdbc:mysql://122.51.48.52:3306/imooc_demo?useSSL=false
     username=xianglin
     password=950915
     initialSize=10
     maxActive=20
     ```
  
  3. 使用数据库连接池

     ```java
     // 1. 加载属性文件
     InputStream inputStream = DruidSample.class.getResourceAsStream("/druid.properties");
     Properties properties = new Properties();
     properties.load(inputStream);
     // 2. 获取 DateSource 数据源对象
     DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);
     // 3. 创建数据库连接
     Connection connection = dataSource.getConnection();
     ```
  

### C3P0

* [C3P0 官网](https://www.mchange.com/projects/c3p0/)

* C3P0 使用方式

  1. 获取 jar 文件

     ```xml
     <dependency>
       <groupId>com.mchange</groupId>
       <artifactId>c3p0</artifactId>
       <version>0.9.5.5</version>
     </dependency>
     ```

  2. `c3p0-config.xml`

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <c3p0-config>
         <default-config>
             <property name="driverClass">com.mysql.cj.jdbc.Driver</property>
             <property name="jdbcUrl">jdbc:mysql://122.51.48.52:3306/imooc_demo?useSSL=false</property>
             <property name="user">xianglin</property>
             <property name="password">950915</property>
             <property name="initialPoolSize">10</property>
             <property name="maxPoolSize">10</property>
         </default-config>
     </c3p0-config>
     ```
  
  3. 使用数据库连接池

     ```java
     // 1. 加载配置文件
     // 2. 创建 DataSource 数据源
     DataSource dataSource = new ComboPooledDataSource();
     // 3. 获取数据库连接
     Connection connection = dataSource.getConnection();
     ```

## Apache Commons DBUtils

* `Apache Commons DBUtils`是 Apache 提供的开源JDBC 工具类库

* 它对 JDBC简单封装，学习成本低

* 使用 DBUtils 能简化编码工作量

* [commons-dbutil 官网](https://commons.apache.org/proper/commons-dbutils/)

* DBUtils 的依赖坐标

  ```xml
  <dependency>
    <groupId>commons-dbutils</groupId>
    <artifactId>commons-dbutils</artifactId>
    <version>1.7</version>
  </dependency>
  ```

## Statement 和 PreparedStatement

* `Statement`用于执行静态SQL 并返回结果
* `PreparedStatement`用于执行预编译的 SQL 语句。
* `PreparedStatement`由于预编译使得效率较高
* `PreparedStatement`可以有效的防止 SQL 注入风险
