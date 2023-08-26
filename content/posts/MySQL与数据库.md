---
title: MySQL 与数据库
date: 2020-09-06 22:55:14
categories: [learn]
tags: [MySQL]
---

# MySQL

## SQL 与 NoSQL

### 常见的关系型数据库：

* MySQL
* Oracle
* DB2
* SQL Server

### 常见的非关系型数据库

* Redis
* MongoDB

### MySQL 安装与用户管理

### MySQL 配置文件

`/etc/mysql/mysql.conf.d/mysqld.cnf`

## MySQL 数据库、表操作

### SQL

* DML：数据操作语言
* DCL：数据控制语言
* DDL：数据定义语言
* SQL 语句的注意事项
  1. SQL 语句不区分大小写
  2. SQL 语句必须以分号结尾
* SQL 语句的注释
  1. `# 这是一段注释文字`
  2. `/* 这是另一段注释文字 */`

### 数据库操作

```sql
# 数据库操作
# 创建数据库
create database demo;
# 查询存在的数据库
show databases;
# 删除数据库
drop database demo;
```

### 数据表操作

```sql
use demo;
# 创建数据表--student
create table student
(
    id       int unsigned primary key,
    name     varchar(20) not null,
    sex      char(1)     not null,
    birthday date        not null,
    tel      char(11)    not null,
    remark   varchar(200)
);

insert into student
values (1, '李强', '男', '1999-01-01', '17328222989', '备注');

# 查看表
show tables;
# 查看表结构
desc student;
# 查看建表语句
show create table student;
# 删除表
drop table student;
```

### 修改数据表结构

```sql
# 修改表结构
# 增加表字段
alter table student
    add address  VARCHAR(200) not null,
    add home_tel CHAR(11)     not null;

# 修改表字段属性
alter table student
    modify home_tel VARCHAR(20) not null;

# 修改字段名
alter table student
    change address home_address VARCHAR(200) NOT NULL;

# 删除字段
alter table student
    drop home_address,
    drop home_tel;

desc student;
```



## MySQL 的数据类型和约束

### 数据类型

* 数值型

  | 类型      | 大小（字节） | 范围（有符号）            | 范围（无符号）  | 说明                                     |
  | --------- | ------------ | ------------------------- | --------------- | ---------------------------------------- |
  | TINYINT   | 1            | [-128, 127]               | [0, 255]        | 小整型值                                 |
  | SMALLINT  | 2            | [-32768, 32767]           | [0, 65535]      |                                          |
  | MEDIUMINT | 3            |                           |                 |                                          |
  | INT       | 4            | [-2147483648, 2147483647] | [0, 4294967295] |                                          |
  | BIGINT    | 8            |                           |                 |                                          |
  | FLOAT     | 4            |                           |                 | 单精度浮点数                             |
  | DOUBLE    | 8            |                           |                 | 双精度浮点数                             |
  | DECIMAL   | DECIMAL(M,D) |                           |                 | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 |

* 字符串类型

  | 类型    | 大小（字符） | 说明                   |
  | ------- | ------------ | ---------------------- |
  | CHAR    | 1 - 255      | 定长字符串             |
  | VARCHAR | 1 - 65535    | 变长字符串             |
  | TEXT    | 1 - 65535    | 长文本数据             |
  | BLOB    | 1 - 65535    | 二进制形式的长文本数据 |

  *char(n) 和 varchar(n) 中括号中 n 代表字符的个数，并不代表字节个数，比如 CHAR(30) 就可以存储 30 个字符。*

* 日期/时间类型

  | 类型      | 大小（字节） | 范围                                      | 格式                | 说明             |
  | --------- | ------------ | ----------------------------------------- | ------------------- | ---------------- |
  | DATE      | 3            | 1000-01-01 / 9999-12-31                   | YYYY-MM-DD          | 日期值           |
  | TIME      | 3            | -838:59:59 / 838:59:59                    | HH:MM:SS            | 时间值或持续时间 |
  | YEAR      | 1            | 1901 / 2155                               | YYYY                | 年份值           |
  | DATETIME  | 8            | 1000-01-01 00:00:00 / 9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 日期时间         |
  | TIMESTAMP | 4            |                                           | YYYYMMDD HHMMSS     | 时间戳           |

### 关系型数据库三范式

1. 列不可以再分：数据表的每一列都是不可分割的基本数据项，同一列中不能有多个值，也不能存在重复的属性
2. 属性完全依赖于主键：数据表中的每条记录必须是唯一的。为了区分，通常要为表加上一个列存储唯一标识
3. 属性不依赖于其他非主属性：每列都与主键有直接关系，不存在传递依赖

### 字段约束

| 约束名称 | 关键字      | 描述                      |
| -------- | ----------- | ------------------------- |
| 主键约束 | PRIMARY KEY | 字段值唯一，且不能为 NULL |
| 非空约束 | NOT NULL    | 字段值不能为 NULL         |
| 唯一约束 | UNIQUE      | 字段值唯一，可以为 NULL   |
| 外键约束 | FOREIGN KEY |                           |

### 索引

* 数据量很大，而且经常被查询的数据可以设置索引

* 索引只添加在经常被用作检索条件的字段上

* 创建表时添加索引

  ```sql
  # 使用索引
  create table t_message
  (
      id          int unsigned primary key,
      content     varchar(200)            not null,
      type        enum ("公告","通报","个人通知") not null,
      create_time timestamp               not null,
      index idx_type (type)
  );
  ```
  
* 操作索引

  ```sql
  # 添加删除索引
  create index idx_type on t_message (type);
  alter table t_message
      add index idx_time (create_time);
  # 查看索引
  show index from t_message;
  # 删除索引
  drop index idx_type on t_message;
  ```

## 数据库的基本查询

### 数据的简单查询（别名、字段计算）

```sql
# 基本查询
select *
from t_emp;

select empno, ename
from imooc_demo.t_emp;

# 使用列别名
select empno, sal * 12 as income
from imooc_demo.t_emp;
```



### 数据的高级查询（排序、分页、去重）

* 分页

  ```sql
  # 分页查询
  select empno, ename
  from imooc_demo.t_emp
  limit 10,5;
  select *
  from imooc_demo.t_emp
  limit 10;
  ```

* 排序：ASC、DESC

  ```sql
  # 排序
  select empno, ename, sal, deptno
  from imooc_demo.t_emp
  order by sal desc;
  
  select empno, ename, sal, hiredate
  from imooc_demo.t_emp
  order by sal desc, hiredate;
  
  select empno,ename,sal
  from imooc_demo.t_emp
  order by sal desc
  limit 0,5;
  ```

* 去重

  ```sql
  # 去重
  select job
  from imooc_demo.t_emp;
  
  select distinct job
  from imooc_demo.t_emp;
  
  # 去重失效
  select distinct job, ename
  from imooc_demo.t_emp;
  ```

### 数据的有条件查询（表达式）

```sql
# 条件查询
select empno, ename, sal
from imooc_demo.t_emp
where deptno = 10
  and sal >= 2000;
  
# 数字运算 
select empno, ename, sal, hiredate
from imooc_demo.t_emp
where deptno = 10
  and (sal + ifnull(comm, 0)) * 12 >= 15000
  and datediff(now(), hiredate) / 365 >= 20;
```

1. `IS NULL`
2. `IS NOT NULL`
3. `BETWEEN AND`
4. `LIKE`
5. `REGEXP`

## 数据库高级查询

### 聚合函数

| 函数             | 说明                     |
| ---------------- | ------------------------ |
| `avg()`、`sum()` | 返回某列的平均值、和     |
| `count()`        | 返回某列的行数           |
| `max()`、`min()` | 返回某列的最大值、最小值 |

```sql
select avg(sal + ifnull(comm, 0)) as avg
from imooc_demo.t_emp;

select sum(sal)
from imooc_demo.t_emp
where deptno in (10, 20);

select ename
from t_emp
where char_length(ename) = (select max(char_length(ename))
                            from imooc_demo.t_emp);

select max(hiredate)
from imooc_demo.t_emp;

select count(*), count(comm)
from imooc_demo.t_emp;

```

### 分组

```sql
select deptno, round(avg(sal))
from imooc_demo.t_emp
group by deptno;

select deptno, avg(sal), sum(sal), max(sal), min(sal)
from imooc_demo.t_emp
group by deptno
with rollup;

# group_concat
select deptno, group_concat(ename), count(*)
from imooc_demo.t_emp
where sal >= 2000
group by deptno;
```

### 表连接查询

* 表连接分为两种：内连接和外连接

* 内连接用于查询多张关系表满足连接条件的记录

* 内连接的多种语法形式

  ```sql
  
  select ename
  from imooc_demo.t_emp
  where deptno = (select deptno from imooc_demo.t_emp where ename = 'SCOTT');
  
  select e2.ename
  from imooc_demo.t_emp e1
           join imooc_demo.t_emp e2 on e1.deptno = e2.deptno
  where e1.ename = 'SCOTT'
    and e2.ename != e1.ename;
  ```

* 外连接的各种语法形式

  ```sql
  # 查询每名员工的编号、姓名、部门、月薪、工资等级、工龄、上司编号、上司姓名、上司部门
  select e1.empno,
         e1.ename,
         d1.dname,
         (e1.sal + ifnull(e1.comm, 0))             as sal,
         s.grade,
         floor(datediff(now(), e1.hiredate) / 365) as gl,
         e2.empno                                     ssbh,
         e2.ename                                     ssxm,
         d2.dname                                     ssbm
  from t_emp e1
           join t_emp e2 on e1.mgr = e2.empno
           left join t_dept d1 on e1.deptno = d1.deptno
           left join t_salgrade s on (e1.sal + ifnull(e1.comm, 0)) between s.losal and s.hisal
           left join t_dept d2 on e2.deptno = d2.deptno;
  
  select e1.empno,
         e1.ename,
         (select d1.dname from t_dept d1 where d1.deptno = e1.deptno)                                       dname,
         (e1.sal + ifnull(e1.comm, 0))             as                                                       sal,
         (select s.grade from t_salgrade s where (e1.sal + ifnull(e1.comm, 0)) between s.losal and s.hisal) grade,
         floor(datediff(now(), e1.hiredate) / 365) as                                                       gl,
         e2.empno                                                                                           ssbh,
         e2.ename                                                                                           ssxm,
         (select d2.dname from t_dept d2 where d2.deptno = e2.deptno)                                       ssbm
  from t_emp e1
           left join t_emp e2 on e1.mgr = e2.empno;
  ```

### 子查询

* 子查询是一种查询中嵌套查询的语句
* WHERE 子查询
* FROM 子查询，只会执行一次
* SELECT 子查询每输出一条记录都要执行一次，查询效率很低
* 单行子查询
* 多行子查询，只能出现在 WHERE 和 FROM 子查询中

## MySQL对数据的基本操作

### 数据添加 `INSERT`

```sql
insert into tableName(col1,clo2,...) values(value1,value2,...),(value1,value2,...);
```

```sql
insert into t_dept(deptno, dname, loc)
values (60, '技术部', '北京'),
       (70, '保安部', '北京');
      
insert into t_emp
(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES (8001, '李娜', 'SALESMAN', 8000, '1988-12-20', 200, null, (select deptno from t_dept where dname = '技术部'));
```

MySQL 的 INSERT 语句方言

```sql
insert [into] tableName set col1 = value1,col2 = value2;

insert into t_dept
set deptno = 8000,
    dname  = '开发部',
    loc    = '上海';
```

IGNORE 关键字会让 INSERT 只插入数据库不存在的记录

```sql
insert ignore into t_dept (deptno, dname, loc)
values (40, '部门名称', '部门地址');
```

### 数据修改

```sql
update [ignore] tableName
set col1 = value1, col2 = value2,...
[where condition]
[order by ...]
[limit ...];
```

* UPDATE 语句的表连接

* 表链接的 UPDATE 语句可以修改多张表的记录

  ```sql
  update table1 [left | right]join table2 on condition set col1 = value1, col2 = value2 ...;
  update table1, table2 set col1 = value1, col2 = value2 where condition;
  ```

  ```sql
  # 把 ALLEN 调往 REASERCH 部门，职务调整为ANALYST
  update t_emp e join t_dept d
  set e.deptno = d.deptno,
      e.job    = 'ANALYST',
      d.loc    = '北京'
  where e.ename = 'ALLEN'
    AND d.dname = 'RESEARCH';
    
    
  # 把底薪低于公司平均底薪的员工，底薪增加150 元
  update t_emp join (select avg(sal) as avg
                     from t_emp) as s
      on sal < s.avg
  set sal = sal + 150;
  
  update t_emp e left join t_dept d on e.deptno = d.deptno
  set e.deptno = 20
  where e.deptno is null
     or (d.dname = 'SALES' and e.sal < 2000);
  ```


### 数据删除

```sql
DELETE [IGNORE] FROM tableName
[where condition]
[order by ...]
[limit ...];
```

```sql
delete
from t_emp
where deptno = 20
order by sal + ifnull(comm, 0) desc
limit 1;
```

* DELETE 语句可以引入表连接

  ```sql
  DELETE table1 ,... FROM table1 JOIN table2 ON codition
  [WHERE ...]
  [ORDER BY ...]
  [LIMIT ...];
  ```

  ```sql
  # 删除 SALES 部门下所有员工和 SALES 部门记录
  DELETE e,d
  FROM t_emp e
           join t_dept d on e.deptno = d.deptno
  where d.dname = 'SALES';
  
  # 删除每个部门中底薪低于部门平均底薪的员工
  delete e
  from t_emp e
           join
       (select deptno, avg(sal) as avg
        from t_emp
        group by deptno) t on e.sal < t.avg;
        
  # 删除员工 KING 和直接下属的记录
  delete e
  from t_emp e
           join (select empno from t_emp where ename = 'KING') t
                on e.mgr = t.empno
                    or e.empno = t.empno; 
  ```

* DELETE 语句既可以是内连接也可以是外连接

* 快速删除数据表中的全量记录

  `TRUNCATE TABLE tableName`

  ## MySQL 函数

### 数字函数

![image-20200906131818682](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225854.png)

![image-20200906132344618](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225855.png)

```sql
# 数字函数
select abs(-100);
select round(4.6288, 2); # 4.63
select floor(9.9); # 9
select ceil(3.2); # 4
select power(2, 3); # 8
select log(7, 3);
select ln(10);
select sqrt(9);
select pi();
```

### 日期函数

* 获取当前日期时间：`now()`、`curDate()`、`curTime()`

* 日期格式化函数：`date_format()`

  ![image-20200906133207708](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/133208.png)

* 日期偏移计算：`date_add(date,interval 偏移量 时间单位)`

  ```sql
  select ADDDATE(now(), 1);
  select date_add(now(), interval 1 day);
  ```

### 字符函数

| 函数                                              | 说明                   |
| ------------------------------------------------- | ---------------------- |
| `left()`                                          | 返回串左边的字符       |
| `right()`                                         | 返回串右边的字符       |
| `ltrim()`                                         | 去掉串左边的空格       |
| `rtrim()`                                         | 去掉串右边的空格       |
| `trim()`                                          | 去掉串左、右两边的字符 |
| `upper()`                                         | 将串转为大写           |
| `lower()`                                         | 将串转为小写           |
| `length()`、`char_length()`、`character_length()` | 返回串的长度           |
| `locate()`                                        | 找出一个串的子串       |
| `substr()`、`substring()`                         | 返回字串的字符         |
| `concat()`                                        | 链接字符串             |
| `replace()`                                       | 替换字符               |
| `insert()`                                        | 插入、替换字符串       |

```sql
select left('left', 0);
select right('right', 4);

select ltrim(' 123 ');
select rtrim(' 123 ');
select trim(' 1 2 3 ');

select upper('Abc');
select lower('Abc');

select length('123'); -- 3
select length('abc'); -- 3
select length('中文 '); -- 7
select length('中A'); -- 4
select char_length('123 '); -- 4
select char_length('abc'); -- 3
select char_length('中文'); -- 2
select char_length('中A'); -- 2
select character_length('123'); -- 3
select character_length('abc'); -- 3
select character_length('中文'); -- 2
select character_length('中A'); -- 2

select locate('ab1c', '中3abcde');

select substr('123', -3);
select substring('123', -1);
select substring_index('www.ee.com', '.', -1);

select insert('你好', 1, 1, '先生');
select replace('先生你好', '先生', '女士');
```

### 条件函数

* `IF()`

  ```sql
  select if('a' > 'b', 'a', 'b');
  select IF(day(now()) = 1, '第一天', '不是第一天')
  ```

* `IFNULL()`

  ```sql
  select ifnull(null, 'a');
  ```

* `CASE()`

  ```sql
  CASE
  	WHEN condition1 THEN value1
  	WHEN condition2 THEN value2
  	ELSE value
  END
  
  select case day(now()) when 1 then '第一天' else '不是第一天' end ;
  ```

## MySQL 的事务机制

### MySQL 事务

```sql
START TRANSACTION;
comment;
[ROLLBACK]|[COMMIT];

start transaction;
delete
from t_emp;
delete
from t_dept;
select *
from t_dept;

commit;
```

### 事务的 ACID

* 原子性：一个事务中的所有操作要么全部成功，要么全部失败。事务执行后，不允许存在中间状态；
* 一致性：不管在任何给定时间，并发事务有多少，事务必须保证运行结果的一致性；
* 隔离性：事务不受其它并发事务的影响；
* 持久性：事务一旦提交，结果便是永久性的。即便发生宕机，依然可以依靠事务日志完成数据的持久化。

### 事务的四个隔离级别

| 隔离级别         | 描述                                                       |
| ---------------- | ---------------------------------------------------------- |
| READ UNCOMMITTED | 可以读取其它事务未提交的数据                               |
| READ COMMITTED   | 读取其它事务提交的数据                                     |
| repeatable read  | 重复读，就是在开始读取数据（事务开启）时，不再允许修改操作 |
| Serializable     | 事务串行化顺序执行，可以避免脏读、不可重复读与幻读         |

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED; 
```



### MySQL 数据导入导出

* 数据导出，导出的是业务数据

* 数据备份，备份数据文件、日志文件、索引文件等等

* `mysqldump` 用来把业务数据导出成 SQL 文件

  ```sql
  mysqldump -uroot -p [no-data] database > path
  
  mysqldump -uxianglin -p mybatis > mybatis.sql
  ```

* `source` 命令用于导入 SQL 文件，包括创建数据表、写入记录等。



## SQL 练习

表结构如下：

```sql
CREATE TABLE `dept`  (
  `deptno` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `dname` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `loc` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`deptno`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 71 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

CREATE TABLE `emp`  (
  `empno` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `ename` varchar(15) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `job` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `mgr` int(10) UNSIGNED NULL DEFAULT NULL,
  `hiredate` date NULL DEFAULT NULL,
  `sal` decimal(10, 2) NULL DEFAULT NULL,
  `comm` decimal(7, 2) NULL DEFAULT NULL,
  `deptno` int(10) UNSIGNED NULL DEFAULT NULL,
  PRIMARY KEY (`empno`) USING BTREE,
  INDEX `deptno`(`deptno`) USING BTREE,
  CONSTRAINT `emp_ibfk_1` FOREIGN KEY (`deptno`) REFERENCES `dept` (`deptno`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE = InnoDB AUTO_INCREMENT = 8019 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
```

* 按部门编号升序、工资倒序排列员工信息

  ```sql
  select * from emp order by deptno asc, sal desc;
  ```

* 列出 deptno=30的部门名称及员工

  ```sql
  select e.*,d.dname from emp e, dept d where e.deptno = d.deptno and d.deptno = 30;
  ```

* 列出每个部门最高、最低、平均工资

  ```sql
  select deptno, max(sal) max, min(sal) min, avg(sal) avg from emp group by deptno; 
  ```

* 列出市场部 SALES 及研发部RESEARCH的员工

  ```sql
  select e.*,d.dname from emp e, dept d where e.deptno = d.deptno and d.dname in ('SALES','RESEARCH') ORDER BY d.deptno;
  ```

* 列出人数超过三人的部门

  ```sql
  select * from dept d where d.deptno in (select e.deptno from emp e group by e.deptno having count(*) > 3);
  
  select d.dname, count(*) from dept d , emp e where d.deptno = e.deptno group by d.dname having count(*) > 3;
  ```

* 计算 MILLER 年薪比 SMITH 高多少

  ```sql
  select s.salary - m.salary from 
  (select e.sal * 12 + ifnull(e.comm,0) salary, e.ename  from emp e where e.ename = 'SMITH') m,
  (select e.sal * 12 + ifnull(e.comm,0) salary, e.ename  from emp e where e.ename = 'MILLER') s
  ```

* 列出直接向KING 汇报的员工

  ```sql
  select * from emp e where e.mgr = (select empno from emp where ename = 'KING');
  
  select e.* from emp e,emp e1 where e.mgr = e1.empno and e1.ename = 'KING';
  
  select e.* from emp e1 left join emp e on e1.empno = e.mgr where e1.ename = 'KING'; 
  ```

* 列出公司所有员工的工龄，并按倒序排列

  ```sql
  select e.ename ,ceil(datediff(now(),hiredate)/365) age from emp e order by age
  ```

* 计算管理者与基层员工平均薪资的差距

  ```sql
  select a.avg - b.avg from 
  (select avg(sal) avg from emp where job = 'PRESIDENT' or job = 'MANAGER') a,
  (select avg(sal) avg from emp where job not IN ('PRESIDENT' ,'MANAGER')) b
  ```

  
