---
title: MyBatis 拦截器
date: 2020-10-31 22:53:47
categories: [learn]
tags: [mybatis]
---

# MyBatis 插件原理及 PageHelper 分页插件介绍


## 从 JDBC 开始

### JDBC 介绍

* JDBC（Java Database Connectivity）API 是 Java 语言与广泛数据库产品交互的基础，JDBC 技术体现了 Java `"write Once,Run AnyWhere"`的思想，即Java 为数据库访问提供一套纯 Java API，同时提供一个驱动管理器（DriverManager）。数据库供应商提供自己的数据库驱动，并注册到 Java 提供的DriverManager 中。

* 普通的 Java 开发人员只需要基于 JDBC API 开发应用程序即可，不需要关注驱动的实现和驱动与数据库交互的逻辑。只有在程序运行时，才需要加载和使用相应的驱动。这也是基于 Maven 的项目在声明依赖的时候，可以将其依赖范围声明为`runtime`的原因。


### JDBC 使用步骤

* 使用 JBDC 完成对数据库操作的基本步骤如下

	1. 注册驱动数据库类；
	2. 通过`DriverManager#getConnection()`建立到指定数据库的连接；
	3. 通过`Connection`对象创建`Statement`；
	4. 通过`Statement`中相关方法执行 SQL，按业务处理其返回值；
	5. 释放资源，关闭数据库连接等。
	
	```java
	// 获取数据库连接
	connection = DriverManager.getConnection(
    properties.getProperty("jdbc.url"),
	  properties.getProperty("jdbc.username"),
	  properties.getProperty("jdbc.password"));
	
	// 获取执行 SQL 的 Statement 对象
	String listAllSql = "select * from products";
	preparedStatement = connection.prepareStatement(listAllSql);
	
	// 执行查询
	resultSet = preparedStatement.executeQuery();
	
	// 按业务要求处理查询结果
	
	// 释放资源
	resultSet.close();
	preparedStatement.close();
	connection.close();
	```
	

### 基于 JDBC 的思考

* 使用 JDBC API 开发的不足之处比较明显，仅针对上面的查询场景而言，有以下几点

  1. 创建数据库连接、关闭连接释放资源等语句繁琐且冗余；
  2. SQL 语句和业务代码糅合在一起，不便于维护；
  3. 需要自己解析查询结果。

* `Commons DbUtils`和`Spring JDBC`都对 JDBC 做了封装，隐藏了创建数据库连接及释放资源等繁琐操作的处理过程，也提供了映射查询结果的方法，可以方便的将查询结果映射为 Map 或对应的 JavaBean。

  ```java
  // 使用 DbUtils 的 QueryRunner 完成查询及结果映射
  QueryRunner queryRunner = new QueryRunner(dataSource);
  List<Map> mapList = queryRunner.query(sql,new BeanListHandler<>(Map.class));
  
  // 使用 Spring JdbcTemplate 完成查询并将结果映射为 JavaBean
  List<Employee> list = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(Employee.class));
  ```


## 使用 MyBatis

* MyBatis 是一款优秀的支持自定义SQL查询、存储过程和高级映射的持久层框架，消除了几乎所有的JDBC代码和参数的手动设置以及结果集的处理。MyBatis 可以使用XML 或注解进行配置和映射，MyBatis 通过将参数映射到配置的 SQL 形成最终执行的 SQL 语句，最后将执行SQL 的结果映射成 Java 对象返回。
* MyBatis 相关网站
  1. [mybatis]([mybatis – MyBatis 3 | 简介](https://mybatis.org/mybatis-3/zh/index.html))
  2. [mybatis-spring]([mybatis-spring – MyBatis-Spring | 简介](http://mybatis.org/spring/zh/))
  3. [mybatis-spring-boot]([mybatis-spring-boot – About](http://mybatis.org/spring-boot-starter/))

### MyBatis 配置文件`mybatis-config.xml`

* MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。

* MyBatis 的配置包含如下项：

  * configuration
    * properties（属性）
    * settings（设置）
    * typeAliases（类型别名）
    * typeHandlers（类型处理器）
    * objectFactory（对象工厂）
    * plugins（插件）
    * environments（环境配置）
      * environment（环境变量）
        * transactionManager（事务管理器）
        * dataSource（数据源）
    * databaseIdProvider（数据库厂商标识）
    * mappers（映射器）

* 每个配置项的具体内容可以参照MyBatis官网或者MyBatis jar包中提供的`mybatis-3-config.dtd`或`mybatis-config.xsd`。需要注意的是，并不要求每个配置项必需存在，但是需要按照上述顺序进行配置。

  ```dtd
  <!ELEMENT configuration (properties?, settings?, typeAliases?, typeHandlers?, objectFactory?, objectWrapperFactory?, reflectorFactory?, plugins?, environments?, databaseIdProvider?, mappers?)>
  ```

* 一个简单的配置示例如下：

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
      <properties resource="jdbc-mysql.properties"/>
    
      <settings>
          <setting name="mapUnderscoreToCamelCase" value="true"/>
      </settings>
  
      <environments default="dev">
          <environment id="dev">
              <transactionManager type="JDBC"/>
              <dataSource type="UNPOOLED">
                  <property name="driver" value="${jdbc.driver}"/>
                  <property name="url" value="${jdbc.url}"/>
                  <property name="username" value="${jdbc.username}"/>
                  <property name="password" value="${jdbc.password}"/>
              </dataSource>
          </environment>
      </environments>
  
      <mappers>
          <mapper resource="mapper/ProductMapper.xml"/>
      </mappers>
  </configuration>
  ```

### MyBatis 映射文件`Mapper.xml`

* MyBatis 的强大在于它的语句映射，在完成基本配置后，MyBatis 开发只需要将重点放在Mapper映射中。MyBatis的结果映射、动态SQL、缓存等配置都在`mapper.xml`文件中。

* `Mapper.xml`的部分配置项如下：

  * `cache` – 该命名空间的缓存配置。
  * `cache-ref` – 引用其它命名空间的缓存配置。
  * `resultMap` – 自定义结果集映射方式。
  * `sql` – 定义可重用的 sql 块。
  * `insert`、`update`、`delete`、`select` – 映射SQL语句。

* 同样的，每个标签的使用细节可以参照MyBatis官网或者MyBatis jar包中提供的`mybatis-3-mapper.dtd`或`mybatis-mapper.xsd`。

  ```dtd
  <!ELEMENT mapper (cache-ref | cache | resultMap* | parameterMap* | sql* | insert* | update* | delete* | select* )+>
  ```

* 一个简单的配置如下：

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="product">
      <select id="getProduct" resultType="site.xianglin.share.entity.Product">
          select prod_id, vend_id, prod_name, prod_price, prod_desc
          from products
          limit 1
      </select>
  
      <select id="pageProduct" resultType="Map">
          select *
          from products
          limit #{pageSize} offset #{offset}
      </select>
  
      <select id="pageProductRowBounds" resultType="Map">
          select *
          from products
      </select>
  </mapper>
	```


### MyBatis 示例

* 基于以上示例配置，完善其他内容，并编写一个简单的测试类，体会 MyBatis 给 JDBC 开发带来的便捷。 MyBatis 的基础使用一般分为以下几步：

  1. 在项目中引入MyBatis的依赖
  2. 通常创建基于XML的配置文件`mybatis-config.xml`
  3. 通常创建基于XML的映射文件`ProductMapper.xml`
  4. 通过配置文件创建`SqlSessionFactory`
  5. 从`SqlSessionFactory`中获取`SqlSession`，完成数据库操作
  6. 对分析、处理结果数据

  其中4、5两步的示例代码如下：

  ```java
  // 使用MyBatis提供的Resources工具类获取配置文件，并由SqlSessionFactoryBuilder解析创建SqlSessionFactory
  Reader resource = Resources.getResourceAsReader("mybatis-config.xml");
  SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resource);
  
  // 从SqlSessionFactory中获取SqlSession完成数据库操作，使用try-with-resources自动关闭资源
  try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
      Product product = sqlSession.selectOne("product.getProduct");
      Assert.assertNotNull(product);
  }
  ```


### MyBatis 执行过程

* 测试方法执行成功后，不妨借助 IDE 的调试功能，简单跟踪 MyBatis 初始化、SQL 执行以及结果映射的过程。

* 从`SqlSessionFactoryBuilder`的`build`方法入手，`build`方法大致有三种类型的重载：

  ```java
  SqlSessionFactory build(InputStream inputStream, String environment, Properties properties);
  SqlSessionFactory build(Reader reader, String environment, Properties properties)；
    
  SqlSessionFactory build(Configuration config)；
  ```

  且前两种方法都会调用`build(Configuration config)`。MyBatis 运行时的配置信息都保存在`Configuration`实例中，分析 MyBatis 基于配置文件初始化的步骤也就是分析配置信息从配置文件读取、解析并保存在到`Configuration`的过程。主要代码如下：

  ```java
  XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
  parser.parse()
  ```

  `parse`方法会调用`XMLConfigBuilder`中的`parseConfiguration`方法将 MyBatis 配置文件解析为`Configuration`实例。MyBatis 会读取配置文件中的各个配置项，做对应的处理，并将其赋值给`Configuration`中对应的属性。

  有了`Configuration`实例后，就会调用 `build(Configuration config)`方法，生成`SqlSessionFactoryBuilder`实例。

  ```java
  public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }
  ```

* 有了 `SqlSessionFactory`实例后，我们可以从中获得 `SqlSession` 的实例。`SqlSession` 提供了在数据库执行 SQL 命令所需的所有方法。通过 `SqlSession` 实例可以直接执行已映射的 SQL 语句，也可以获取接口对应的代理对象，并通过代理对象执行 SQL 语句。其过程比较简单，如下：

  ```java
  final Environment environment = configuration.getEnvironment();
  final TransactionFactory transactionFactory = environment.getTransactionFactory();
  tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
  final Executor executor = configuration.newExecutor(tx, execType);
  return new DefaultSqlSession(configuration, executor, autoCommit);
  ```

* 获取`SqlSession`实例后就可以调用其中执行 SQL 语句的方法，以`sqlSession.selectOne()`方法为例，跟踪分析其执行过程。`SqlSession`的默认实现是`DefaultSqlSession`，按照`selectOne`的调用链路，最终会调用同类中`selectList`方法，方法签名如下：

  ```java
  <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds)
  ```

  其中`statement`即为 Mapper 文件中配置的 SQL 语句的 `id` 属性，`parameter`为预编译 SQL 需要的参数，`rowBounds`是 MyBatis 提供的保存分页信息的对象。

  在`selectList`方法中获取 MyBatis 初始化时解析并保存在`Configuration`实例中的`MappedStatement`对象，一个`MappedStatement`即对应一个映射SQL语句的标签。然后调用`executor`的`query`方法。

  ```java
  MappedStatement ms = configuration.getMappedStatement(statement);
  return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
  ```

  在`query`方法中获取到`BoundSql`和`CacheKey`后继续调用抽象方法`doQuery`，该方法被`SimpleExecutor`重写。`BoundSql`负责保存SQL，而`CacheKey`与 MyBatis 的缓存机制相关。

  ```java
  BoundSql boundSql = ms.getBoundSql(parameterObject);
  CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
  return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
  ```

  忽略缓存相关内容，在`doQuery`方法中获取`StatementHandler`对象，同时`StatementHandler`会创建并持有`ParameterHandler`用于设置参数，和`ResultSetHandler`用于处理结果。

  ```java
  Configuration configuration = ms.getConfiguration();
  StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
  stmt = prepareStatement(handler, ms.getStatementLog());
  return handler.query(stmt, resultHandler);
  ```

  `prepareStatement`方法完成获取`Connection`对象、获取`Statement`和为预编译 SQL 占位符赋值等操作，至此一个基于 JDBC 的`Statement`对象创建完成。
  
  ```java
  Statement stmt;
  Connection connection = getConnection(statementLog);
  stmt = handler.prepare(connection, transaction.getTimeout());
  // 为SQL占位符赋值
  handler.parameterize(stmt);
  ```
  
* `Statement`实例执行的代码很简单

  ```java
  PreparedStatement ps = (PreparedStatement) statement;
  ps.execute();
  return resultSetHandler.handleResultSets(ps);
  ```

* 如果执行的是查询类 SQL，会调用`ResultSetHandler`的`handleResultSets`方法完成结果映射，假如使用了MyBatis提供的`RowBounds`参数，内存分页的过程也在这里完成。内存分页的实现很简单，调用`skipRows`将结果集的游标偏移 `offset` 位，同时遍历结果的时候只取 `limit` 条记录。

  ```java
  private void skipRows(ResultSet rs, RowBounds rowBounds) throws SQLException {
      // TYPE_FORWARD_ONLY 结果集的游标只能向下滚动
      if (rs.getType() != ResultSet.TYPE_FORWARD_ONLY) {
          if (rowBounds.getOffset() != RowBounds.NO_ROW_OFFSET) {
              rs.absolute(rowBounds.getOffset());
          }
      } else {
          for (int i = 0; i < rowBounds.getOffset(); i++) {
              if (!rs.next()) {
                  break;
              }
          }
      }
  }
  
  private boolean shouldProcessMoreRows(ResultContext<?> context, RowBounds rowBounds) {
      return !context.isStopped() && context.getResultCount() < rowBounds.getLimit();
  }
  ```

* MyBatis 完成一个简单查询的大致过程如上所诉，我将其分为：初始化 MyBatis 创建`SqlSessionFactory`实例、获取`SqlSession`实例、得到`preparedStatement`实例并执行SQL操作、处理映射查询结果等步骤，示意图大致如下所示。以上代码跟踪暂未涉及其中细节，对 MyBatis 执行流程有了大致认识后，可以尝试具体分析 MyBatis 的插件实现原理。

  ![MyBatis执行过程](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/215213.jpg)

## MyBatis 插件 `Plugin`

* MyBatis 允许在映射语句执行过程中的某一点进行拦截调用，允许使用插件来拦截的方法调用包括：

  - `Executor` (`update`,` query`,` flushStatements`,` commit`,` rollback`,` getTransaction`,` close`, `isClosed`)
  - `ParameterHandler` (`getParameterObject`, `setParameters`)
  - `ResultSetHandler` (`handleResultSets`,` handleOutputParameters`)
  - `StatementHandler `(`prepare`, `parameterize`, `batch`, `update`, `query`)
  
* MyBatis在`org.apache.ibatis.plugin`包下提供了插件相关的接口、类和注解，如下所示

  | 名称               | 描述 |
  | ------------------ | ---- |
  | *`Interceptor`*      | 拦截器接口，通过实现此接口，对拦截对象和方法进行处理 |
  | `InterceptorChain` | 保存所有`Interceptor`的实例 |
  | `Invocation`       | 保存被代理对象、被代理方法及运行参数 |
  | `Plugin`           | 实现`InvocationHandler`，是动态代理的默认实现类 |
  | `PluginException`  | 插件相关异常 |
  | `@Intercepts`      | 指定拦截目标的注解 |
  | `@Signature`       | 指定方法签名的注解，与`@Intercepts`配合使用 |


### MyBatis 插件示例

* 实现`org.apache.ibatis.plugin.Interceptor`接口，并使用`@Intercepts`和`@Signature`注解指定插件需要拦截的方法。

  ```java
  @Intercepts({
    @Signature(
      type = Executor.class,
      method = "query",
      args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})
  })
  public class ExamplePlugin implements Interceptor {
      private Properties properties;
  
      @Override
      public Object intercept(Invocation invocation) throws Throwable {
          String simpleName = invocation.getTarget().getClass().getSimpleName();
          String name = invocation.getMethod().getName();
          Object[] args = invocation.getArgs();
          System.out.println("ExamplePlugin 插件拦截了" + simpleName + "." + name + "方法，方法入参为：" + Arrays.toString(args));
          Object result = invocation.proceed();
          System.out.println("同插件一起配置的属性有：" + properties.toString());
        	return result;
      }
  
      @Override
      public void setProperties(Properties properties) {
          this.properties = properties;
      }
  
  }
  ```

* 在 MyBatis 配置文件中使用`plugins`标签注册插件

  ```xml
  <plugins>
    <plugin interceptor="site.xianglin.share.plugin.ExamplePlugin">
      <property name="pluginProperty" value="100"/>
    </plugin>
  </plugins>
  ```

* 执行`sqlSession.selectOne("product.getProduct");`方法观察输出

  ```console
  ExamplePlugin 插件拦截了SimpleExecutor.query方法，方法入参为：[org.apache.ibatis.mapping.MappedStatement@64485a47, {limit=1}, org.apache.ibatis.session.RowBounds@25bbf683, null]
  同插件一起配置的属性有：{pluginProperty=100}
  ```


### MyBatis 插件工作原理

#### 加载插件配置信息

* 上面提到，`XMLConfigBuilder`中的`parseConfiguration`方法将 MyBatis 配置文件解析为`Configuration`实例，插件的信息也在此处解析加载，如下所示（略去`mybatis-config.xml`中未配置的部分）。

  ```java
  private void parseConfiguration(XNode root) {
    try {
      propertiesElement(root.evalNode("properties"));
      Properties settings = settingsAsProperties(root.evalNode("settings"));
      // 插件信息在此加载
      pluginElement(root.evalNode("plugins"));
      settingsElement(settings);
      environmentsElement(root.evalNode("environments"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
  ```
  
* `pluginElement`方法的代码及主要完成的工作如下：

  ```java
  private void pluginElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        // 根据配置类信息创建 Interceptor 对象
        String interceptor = child.getStringAttribute("interceptor");
        Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).getDeclaredConstructor().newInstance();
        // 调用 setProperties 方法设置配置的属性
        Properties properties = child.getChildrenAsProperties();
        interceptorInstance.setProperties(properties);
        // 将插件实例添加到 configuration 的拦截器链中
        configuration.addInterceptor(interceptorInstance);
      }
    }
  }
  ```

* 拦截器链`InterceptorChain`维护了一个`Interceptor`列表保存所有的拦截器实例，定义了`pluginAll`方法用于为被拦截对象创建代理对象。

  ```java
  private final List<Interceptor> interceptors = new ArrayList<>();
  ```

#### 生成代理对象

* Mybatis 插件主要是基于动态代理实现的，MyBatis 的四大对象都是由 `Configuration` 负责创建的，查看对应的方法：

  ```java
  // Executor 代理对象
  public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor = new SimpleExecutor(this, transaction);
    // ...
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
  }
  
  // StatementHandler 代理对象
  public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;
  }
  
  // ParameterHandler 代理对象
  public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
    ParameterHandler parameterHandler = mappedStatement.getLang().createParameterHandler(mappedStatement, parameterObject, boundSql);
    
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
    return parameterHandler;
  }
  
  // ResultSetHandler代理对象
  public ResultSetHandler newResultSetHandler(Executor executor, MappedStatement mappedStatement, RowBounds rowBounds, ParameterHandler parameterHandler,
                                              ResultHandler resultHandler, BoundSql boundSql) {
    ResultSetHandler resultSetHandler = new DefaultResultSetHandler(executor, mappedStatement, parameterHandler, resultHandler, boundSql, rowBounds);
    
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    return resultSetHandler;
  }
  ```

* 这些代理对象都是通过`interceptorChain.pluginAll`方法生成的，`pluginAll`方法定义如下。

  ```java
  public Object pluginAll(Object target) {
    for (Interceptor interceptor : interceptors) {
      target = interceptor.plugin(target);
    }
    return target;
  }
  ```

  可以知道，当配置多个拦截器时，MyBatis 会遍历所有拦截器，按顺序执行拦截器的 `plugin` 方法，被拦截的对象就会被层层代理。

* MyBatis 为`plugin`方法提供了默认实现，那么`Plugin.wrap`就是典型的动态代理实现。

  ```java
  default Object plugin(Object target) {
    return Plugin.wrap(target, this);
  }
  
  public static Object wrap(Object target, Interceptor interceptor) {
    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
    Class<?> type = target.getClass();
    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
    if (interfaces.length > 0) {
      return Proxy.newProxyInstance(
        type.getClassLoader(),
        interfaces,
        new Plugin(target, interceptor, signatureMap));
    }
    return target;
  }
  ```

#### 拦截器执行逻辑

* 由于`Executor`、`ParameterHandler`、`ResultSetHandler`和`StatementHandler`均被代理，所以在执行方法时，首先执行`Plugin`中的`invoke`方法，`invoke`方法的实现如下：

  ```java
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      Set<Method> methods = signatureMap.get(method.getDeclaringClass());
      if (methods != null && methods.contains(method)) {
        return interceptor.intercept(new Invocation(target, method, args));
      }
      return method.invoke(target, args);
    } catch (Exception e) {
      throw ExceptionUtil.unwrapThrowable(e);
    }
  }
  ```

  判断当前方法是否需要被拦截，如果需要，则执行`interceptor`对象的`intercept`方法，不需要则直接执行原方法。

### 实现一个分页插件

* 在MyBatis 执行过程中分析到 MyBatis 默认的分页实现是内存分页，即查询所有记录，在内存中取分页数据。可以通过拦截`StatementHandler`的`prepare`方法，修改MyBatis 最终执行的SQL 来实现物理分页。具体实现如下所示：

  ```java
  /**
   * 简单的分页实现：
   * 1. 如果有RowBounds参数，使用其作为分页参数
   * 2. 如果接口方法类似 list(int pageSize, int pageNum)，使用其作为分页参数
   * 3. 使用ThreadLocal存储分页参数
   *
   * @author xianglin
   */
  @Intercepts({
          @Signature(
                  type = StatementHandler.class,
                  method = "prepare",
                  args = {
                          Connection.class,
                          Integer.class
                  }
          )
  })
  @SuppressWarnings("unchecked")
  public class SimplePageHelper implements Interceptor {
      private String pageSizeKey = "pageSize";
      private String pageNumKey = "pageNum";
      private Page page;
  
      @Override
      public Object intercept(Invocation invocation) throws Throwable {
          StatementHandler statementHandler = (StatementHandler) invocation.getTarget();
          MetaObject metaStatementHandler = SystemMetaObject.forObject(statementHandler);
          while (metaStatementHandler.hasGetter("h")) {
              Object object = metaStatementHandler.getValue("h");
              metaStatementHandler = SystemMetaObject.forObject(object);
          }
          // 分离最后一个代理对象的目标类
          while (metaStatementHandler.hasGetter("target")) {
              Object object = metaStatementHandler.getValue("target");
              metaStatementHandler = SystemMetaObject.forObject(object);
          }
          // 判断是否需要分页
          if (needPage(metaStatementHandler)) {
              // 拼接分页条件
              BoundSql boundSql = (BoundSql) metaStatementHandler.getValue("delegate.boundSql");
              String sql = boundSql.getSql();
              String newSql = "select * from (" + sql + ") t limit " + this.page.limit + " offset " + this.page.offset;
              metaStatementHandler.setValue("delegate.boundSql.sql", newSql);
          }
          return invocation.proceed();
      }
  
      private boolean needPage(MetaObject metaObject) {
          // 判断 RowBounds
          RowBounds rowBounds = (RowBounds) metaObject.getValue("delegate.rowBounds");
          if (rowBounds != null && rowBounds != RowBounds.DEFAULT) {
            	// 取消默认的内存分页
              metaObject.setValue("delegate.rowBounds", RowBounds.DEFAULT);
              this.page = new Page();
              this.page.limit = rowBounds.getLimit();
              this.page.offset = rowBounds.getOffset();
              return true;
          }
          // 判断方法参数
          Object parameterObject = metaObject.getValue("delegate.parameterHandler.parameterObject");
          SqlCommandType sqlCommandType = (SqlCommandType) metaObject.getValue("delegate.parameterHandler.mappedStatement.sqlCommandType");
          if (parameterObject instanceof Map && sqlCommandType == SqlCommandType.SELECT) {
              Map<String, Object> paraMap = (Map<String, Object>) parameterObject;
              Integer pageSize = (Integer) paraMap.get(this.pageSizeKey);
              Integer pageNum = (Integer) paraMap.get(this.pageNumKey);
              if (pageSize != null && pageNum != null) {
                  this.page = new Page();
                  this.page.limit = pageSize;
                  this.page.offset = (pageNum - 1) * pageSize;
                  return true;
              }
          }
  
          // 从ThreadLocal获取分页信息
          Page page = getPage();
          if (page != null) {
              this.page = new Page();
              this.page.limit = page.pageSize;
              this.page.offset = (page.pageNum - 1) * page.pageSize;
              return true;
          }
          return false;
      }
  
      @Override
      public void setProperties(Properties properties) {
          this.pageSizeKey = properties.getProperty("pageSizeKey", "pageSize");
          this.pageNumKey = properties.getProperty("pageNumKey", "pageNum");
      }
  
      public static void startPage(int pageSize, int pageNum) {
          Page page = new Page();
          page.pageSize = pageSize;
          page.pageNum = pageNum;
          PAGE_LOCAL.set(page);
      }
  
      private static Page getPage() {
          Page page = PAGE_LOCAL.get();
          PAGE_LOCAL.remove();
          return page;
      }
  
      private final static ThreadLocal<Page> PAGE_LOCAL = new ThreadLocal<>();
  
      static class Page {
          private int pageSize;
          private int pageNum;
  
          private int limit;
          private int offset;
  
      }
  }
  ```


## MyBatis 分页插件 `PageHelper`

### PageHelper 介绍

* 引入分页插件

  ```xml
  <dependency>
      <groupId>com.github.pagehelper</groupId>
      <artifactId>pagehelper</artifactId>
      <version>最新版本</version>
  </dependency>
  ```

* 配置拦截器插件`com.github.pagehelper.PageInterceptor`

  ```xml
  <plugins>
      <plugin interceptor="com.github.pagehelper.PageInterceptor">
          <property name="param1" value="value1"/>
  	</plugin>
  </plugins>
  ```

* 分页插件参数介绍：

  * `pageSizeZero`：默认值为 `false`，当该参数设置为 `true` 时，如果 `pageSize=0` 或者 `RowBounds.limit = 0` 就会查询出全部的结果（相当于没有执行分页查询，但是返回结果仍然是 `Page` 类型）。

  * `reasonable`：分页合理化参数，默认值为`false`。当该参数设置为 `true` 时，`pageNum<=0` 时会查询第一页， `pageNum>pages`（超过总数时），会查询最后一页。默认`false` 时，直接根据参数进行查询。

  * `supportMethodsArguments`：支持通过 Mapper 接口参数来传递分页参数，默认值`false`，分页插件会从查询方法的参数值中取值，查找到合适的值时就会自动分页。

#### 分页插件常见使用方式

* #### `RowBounds`方式的调用

  ```java
  List<Object> selectList = sqlSession.selectList("product.pageProductRowBounds", null, rowBounds);
  
  // 接口也可以增加RowBounds参数
  PageRowBounds pageRowBounds = new PageRowBounds(0, 10);
  List<Product> productList = mapper.pageProduct(pageRowBounds);
  Assert.assertNotNull(pageRowBounds.getTotal());
  ```

  *由于默认情况下的 `RowBounds` 无法获取查询总数，分页插件提供了一个继承自 `RowBounds` 的 `PageRowBounds`，这个对象中增加了 `total` 属性，执行分页查询后，可以从该属性得到查询总数。*

* #### `PageHelper.startPage` 静态方法调用

  ```java
  PageHelper.startPage(2, 5);
  List<Product> productList = productMapper.listAllProduct();
  
  // 分页时，实际返回的结果list类型是Page<E>，如果想取出分页信息，需要强制转换为Page<E>，
  // 或者使用PageInfo对结果进行包装
  Page<Product> page = (Page<Product>) productList;
  PageInfo<Product> pageInfo = new PageInfo<>(productList);
  ```

  *`PageHelper` 方法使用了静态的 `ThreadLocal` 参数，分页参数和线程是绑定的。*

* #### 使用参数方式

  ```java
  List<Product> pageProduct(int pageSize, int pageNum);
  
  List<Product> productList = productMapper.pageProduct(5, 2);
  ```

  *使用参数方式，需要配置 `supportMethodsArguments` 参数为 `true`*

### 在 Spring 中使用 PageHelper

*见[项目演示](https://github.com/xianglin2020/mybatis)*

### 在 SpringBoot 中使用 PageHelper

*见[项目演示](https://github.com/xianglin2020/mybatis)*

