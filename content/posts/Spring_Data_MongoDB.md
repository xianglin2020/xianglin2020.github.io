---
title: Spring Data MongoDB
date: 2022-08-13 11:52:27
tags: [spring, MongoDB]
categories: [learn]
description: 学习 Spring Data MongoDB 的基础用法。
---

# Spring Data MongoDB

按照 Spring [官网教程](https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/#repositories)学习 Spring Data MongoDB。

## 核心概念

Spring Data repository 抽象中的核心接口是 `org.springframework.data.repository.Repository`，作为一个标记接口，没有定义任何方法：

```java
// Central repository marker interface
@Indexed
public interface Repository<T, ID> {

}
```

接口 `org.springframework.data.repository.CrudRepository` 定义了复杂的 CRUD 操作方法：

```java
// Interface for generic CRUD operations on a repository for a specific type.
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
	// 保存实体
	<S extends T> S save(S entity);
	<S extends T> Iterable<S> saveAll(Iterable<S> entities);
	// 通过 ID 属性查询
	Optional<T> findById(ID id);
  Iterable<T> findAllById(Iterable<ID> ids);
	boolean existsById(ID id);
	// 返回所有记录
	Iterable<T> findAll();
	// 查询所有记录数
	long count();
  // 删除指定记录
	void deleteById(ID id);
	void delete(T entity);
	void deleteAllById(Iterable<? extends ID> ids);
	void deleteAll(Iterable<? extends T> entities);
	// 删除所有记录
	void deleteAll();
}
```

针对 MongoDB 提供了 `org.springframework.data.mongodb.repository.MongoRepository` 接口：

```java
// Mongo specific Repository interface.
@NoRepositoryBean
public interface MongoRepository<T, ID>
		extends ListCrudRepository<T, ID>, ListPagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
	<S extends T> S insert(S entity);
	<S extends T> List<S> insert(Iterable<S> entities);
	@Override
	<S extends T> List<S> findAll(Example<S> example);
	@Override
	<S extends T> List<S> findAll(Example<S> example, Sort sort);
}
```

`org.springframework.data.repository.PagingAndSortingRepository` 接口提供方法用于分页查询：

```java
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends Repository<T, ID> {
	Iterable<T> findAll(Sort sort);
	Page<T> findAll(Pageable pageable);
}
```

![NoMethodRepository](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202208/132729.png)

![Repository](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202208/132538.png)

## 使用方法

通过以下步骤使用 Spring Data MongoDB：

以查询 `Person` 为例

1. 创建一个接口继承自 `Repository` 或者它的子接口，并指定泛型参数 `T` 和 `ID`。

   ```java
   public interface PersonRepository extends Repository<Person, ObjectId> {
   }
   ```

2. 在接口中声明查询方法。

   ```java
   public interface PersonRepository extends Repository<Person, ObjectId> {
       List<Person> findByLastname(String lastname);
   }
   ```

3. 通过注解 `@EnableMongoRepositories` 让 Spring 管理查询接口生成接口代理实现类。

4. 通过依赖注入获取接口实例，使用查询方法。

   ```java
   @SpringBootTest
   class PersonRepositoryTest {
       @Resource
       private PersonRepository personRepository;
   
       @Test
       void findByLastname() {
           var personList = personRepository.findByLastname("Matthews");
           assertFalse(personList.isEmpty());
       }
   }
   ```

## 定义 `Repository` 接口

定义 Repository 接口可以通过继承 `Repository` 及其子接口，也可以在自定义接口上增加 `org.springframework.data.repository.RepositoryDefinition` 注解：

```java
// Annotating an interface with RepositoryDefinition will cause the same behaviour as extending Repository.
public @interface RepositoryDefinition {
	Class<?> domainClass();
	Class<?> idClass();
}
```

例如：

```java
@RepositoryDefinition(domainClass = Person.class, idClass = ObjectId.class)
public interface AnnotationPersonRepository {
    List<Person> findByLastname(String lastname);
}
```

### 微调 Repository 定义

如果只想暴露部分方法，比如自定义的 Repository 中不提供删除方法，可以从 `CrudRepository` 复制其它方法到接口中。

```java
/**
 * 自定义 Repository 只提供保存和查询方法
 *
 * @author linxiang
 */
@NoRepositoryBean
public interface CustomBaseRepository<T, ID> extends Repository<T, ID> {
    <S extends T> S save(S entity);

    Optional<T> findById(ID id);
}
```

增加 `org.springframework.data.repository.NoRepositoryBean` 注解避免 Spring 为接口创建代理对象。

### 使用多个 Spring Data 模块

可以在项目中同时使用多个 Spring Data 模块，比如 Spring Data Jpa 和 Spring Data MongoDB，有几种方式可以区分它们：

1. Repository 定义时继承特定接口，比如 `MongoRepository` 和 `JpaRepository`；

2. 通过实体类上的注解区分，比如 `@Entity` 用于 Jpa ，`@Document` 用于 MongoDB；

3. 通过 `@Enable${store}Repositories` 注解区分

   ```java
   @EnableJpaRepositories(basePackages = "com.acme.repositories.jpa")
   @EnableMongoRepositories(basePackages = "com.acme.repositories.mongo")
   class Configuration {}
   ```

## `MongoTemplate` 介绍

`org.springframework.data.mongodb.core.MongoTemplate` 是 Spring MongoDB 的核心类，该模板提供了增删改查方法以及 MongoDB `Document` 和 Java `POJO` 转换的方法。`MongoTemplate` 是线程安全的。

MongoDB `Document` 和 Java `pojo` 转换是委托 `org.springframework.data.mongodb.core.convert.MongoConverter` 接口实现的，Spring 的默认实现类是 `org.springframework.data.mongodb.core.convert.MappingMongoConverter`。

https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/#mongo-template
