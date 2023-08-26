---
title: MySQL 事务
date: 2020-09-07 21:41:58
categories: [learn]
tags: [MySQL]
---

# MySQL事务

## MySQL事务简介

* 数据库的事务是指一组SQL语句组成的数据库逻辑处理单元，在这组SQL操作中，要么全部执行成功，要么全部执行失败。简单又经典的例子就是转账，事务A中进行转账，那么转出的账号要扣钱，转入的账号要加钱。为了保证数据一致性，这两个操作都必需要同时执行成功。 
* MySQL 事务都是指在 InnoDB 引擎下，MyISAM 引擎是不支持事务的。
* 事务用来管理insert、update、delete语句。

## MySQL事务特性

### ACID简介

MySQL中事务的四大特性包括：**原子性（Atomicity）**、**一致性（Consistent）**、**隔离性（Isolation）**、**持久性（Durable）**,简称`ACID`。

* 原子性是指事务的原子操作，对数据的修改要么全部执行成功，要么全部失败。实现事务的原子性，是基于日志的`Redo/Undo`机制。
* 一致性是指执行事务前后的状态要一致，可以理解为数据一致性。
* 隔离性侧重指事务之间相互隔离，不受影响，隔离性与事务设置的隔离级别有密切的关系。
* 持久性是指一个事务提交后，这个事务的状态会被持久化到数据库中，也就是事务提交，对数据的新增、更新将会持久化到数据库中。

原子性、隔离性、持久性都是为了保障一致性而存在的，一致性也是最终目的。

### 事务的隔离级别

| 隔离级别           | 描述                                 |
| ------------------ | ------------------------------------ |
| `READ UNCOMMITTED` | 可以读到其它事务未提交的数据         |
| `READ COMMITTED`   | 事务只能读到其他事务已经提交过的数据 |
| `REPEATABLE READ`  | 事务不会读到其它事务对数据的更改     |
| `SERIALIZABLE`     | 将事务的执行变为顺序执行             |

* MySQL 使用事务

  开启事务：`begin / start transaction / set autocommit = 0`

  提交事务：`commit`

  回滚事务：`rollback`

  *在执行`begin/start transaction`命令时，它们并不是一个事务的起点，在执行完它们后的第一个 sql 语句，才表示事务真正的启动 。*

* 查看当前数据库的隔离级别

  `select @@transaction_isolation`

  `show variables like 'transaction_isolation'`

* 查看当前运行的事务

  `select * from information_schema.innodb_trx;`

* 设置隔离级别

  `set [session | global] transaction isolation level {read uncommitted | read committed | repeatable read | serializable}`

  *当设置完隔离级别后对于之前打开的会话，是无效的，要重新打开一个窗口设置隔离级别才生效。*

* 事务隔离级别演示

  创建一个User表并插入相关测试数据

  ```sql
  create table user
  (
      id   int(11) not null primary key auto_increment,
      name varchar(20),
      age  tinyint default 0
  ) engine = InnoDB
    default charset = utf8;
  ```

  1. 读未提交

     设置默认隔离级别为`READ UNCOMMITTED`

     `set global transaction isolation level read uncommitted;`

     <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/215305.png" alt="image-20200907215304899" style="zoom:50%;" />

     打开两个查询窗口，同时开启事务，一个窗口更新数据，另一个窗口在更新数据前后分别查询一次数据
  
     ```sql
     # console1
     select * from user;
     # console2
     update user set name = '非科班的科班' where id = 1;
     # console1
   select * from user;
     ```

     ![image-20200907215622686](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/215641.png)

     第二个session产生的未提交的事务状态会直接影响到第一个session，也就是脏读。

  2. 读已提交

     设置默认的事务隔离级别为`READ COMMITTED`

     `set global transaction isolation level read committed;`

     ![image-20200907220111041](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/220111.png)

     重新打开两个窗口同时开启事务A和事务B。在事务B中使用update语句更新行，在事务A中进行查询。在事务B修改数据未提交时，事务A查询到的数据没有改变，当事务B执行提交后，事务A查询到的数据就是改变后的数据

     ![image-20200907220353267](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/220353.png)

     事务B的查询结果会受到事务A的影响，这就是不可重复读。
  
  3. 可重复读
  
     可重复读是对比不可重复读而言的，不可重复读是指同一事务不同时刻读取到的数据值可能不一致。而可重复读是指，事务不会读到其它事务对已有数据的修改，即使其它事务已提交。也就是说，事务开始时读取到的已有数据是什么，在事务提交前的任意时刻，这些数值都是一样的。*<s>但是，对于其它事务新插入的数据是可以读到的，也就引发了幻读问题。</s>（MySQL的`repeatable read`解决了幻读问题）*
     
     ![image-20200907220740412](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/220740.png)
     
     事务 A 在事务 B 删除记录前查询了 ID为 1 的记录，在事务 B中删除记录，并且提交事务，在事务 A 中依旧能查询到，即事务 A 不会读到其它事务对数据的修改。
     
  4. 串行化
  
     串行化是4种事务隔离级别中隔离效果最好的，解决了脏读、可重复读、幻读的问题，但是效果最差，它将事务的执行变为顺序执行，与其他三个隔离级别相比，它就相当于单线程，后一个事务的执行必须等待前一个事务结束。

### `Redo/Undo`机制

#### redo log

`redo log`叫做重做日志，用来实现事务的持久性。该日志文件由两个部分组成：重做日志缓冲（redo log buffer）、重做日志文件（redo log），前者在内存中，后者在磁盘中。当事务提交后会把所有修改信息保存到该日志中。

MySQL 为了提升性能不会把每次的修改都实时同步到磁盘，而是先存到缓冲池中，然后使用后台线程去做缓冲池和磁盘间的同步。引入 redo log 来记录已成功提交事务的修改信息，并且把 redo log 同步到磁盘，用于恢复数据，保障数据库的一致性。

#### undo log

`undo log`叫做回滚日志，用于记录数据被修改前的信息。为了回滚之前的状态，需要将之前的操作都记录下来。用于保障事务的原子性。

### MySQL锁机制

MySQL中的锁可以分为：**共享锁（Shared Locks）**、**排它锁（Exclusive Locks）**、**行锁（Record Locks）**、**表锁**、间隙锁。

* 共享锁是针对同一份数据，多个读操作可以同时进行，即读加锁，不能写并且可以并行读。
* 排它锁针对写操作，假设当前写操作没有完成，它会阻断其它的写锁和读锁，即写加锁，其它读写都阻塞。
* 行锁和表锁是从锁的粒度上划分的，行锁锁定当前数据行，锁的粒度小，加锁慢，发生锁冲突的概率小，并发度高。
* 表锁锁的粒度大，加锁快，发生冲突的概率大，并发度低。
* 间隙锁

数据库只有增、删、改才会加上排它锁，只是查询时并不会加锁。只能通过在 select 语句后显示的加`lock in share mode`或者`for update `来加共享锁和排它锁。

* READ UNCOMMITTED：没有加任何锁，所以它没有隔离效果，性能最好。
* SERIALIZABLE：读的时候加共享锁，也就是其他事务可以并发读，但是不能写。写的时候加排它锁，其他事务不能并发写也不能并发读。
* REPEATABLE READ：在事务开始的时候创建一致性视图，也叫做快照，之后的查询都公用这一个一致性视图，后续事务对数据的更改是对当前事务不可见的。
* READ COMMITTED：每次执行语句的时候都重新生成一次快照。

### 快照遵循的规则

1. 当前事务内的更新都是可见的；
2. 版本未提交的，都是不可见的；
3. 版本已提交，但是在创建快照之后提交的，是不可见的；
4. 版本已提交，但是在创建快照之前提交的，是可见的。

### MVCC

READ COMMITTED、REPEATABLE READ 底层实现采用 MVCC（多版本并发控制）实现。

InnoDB 的 MVCC，是通过在每行记录后面保存两个隐藏的列来实现的。这两个列，一个保存了行创建时间，一个保存了行过期时间，存储的是系统版本号。

### 并发写问题

假如事务1和事务2都要执行update操作，事务1先update数据行的时候，先回获取行锁，锁定数据，当事务2要进行update操作的时候，也会取获取该数据行的行锁，但是已经被事务1占有，事务2只能wait。若是事务1长时间没有释放锁，事务2就会出现超时异常。

加锁的过程要分有索引和无索引两种情况：

1. 有索引的情况，那么 MySQL 直接就在索引数中找到了这行数据，然后加上行锁；
2. 若是没有索引的条件下，就获取所有行，都加上行锁，然后 MySQL 会再次过滤符合条件的的行并释放锁，只有符合条件的行才会继续持有锁。





