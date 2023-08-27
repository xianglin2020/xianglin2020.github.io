---
title: MongoDB 基础知识
date: 2022-05-21 15:32:19
tags: [MongoDB]
categories: [learn]
description: MongoDB 入门知识，包括 MongoDB 介绍、安装、基本操作、复制集等内容。
summary: MongoDB 入门知识，包括 MongoDB 介绍、安装、基本操作、复制集等内容。
---

# MongoDB 基础知识

为「MongoDB高手课 - 极客时间视频课程」第一部分「MongoDB 再入门」的学习记录，相关内容地址如下：

视频教程：[MongoDB教程全集](https://www.bilibili.com/video/BV1rL4y1u74D?spm_id_from=333.337.search-card.all.click)

课件资料：[geektime-mongodb-course](https://github.com/geektime-geekbang/geektime-mongodb-course)

## MongoDB 介绍

### OLTP 和 OLAP

[OLTP与OLAP的关系](https://www.zhihu.com/question/24110442/answer/851671343)：`OLTP`（`On-Line Transaction Processing`）翻译为联机事务处理， `OLAP`（`On-Line Analytical Processing`）翻译为联机分析处理，从字面上来看 `OLTP` 是做事务处理，`OLAP` 是做分析处理。从对数据库操作来看，`OLTP` 主要是对数据的增删改，`OLAP` 是对数据的查询。

### 关于 MongoDB

MongoDB 是一个以 `JSON` 为数据模型的文档型数据库（`JSON Document`），属于 `OLTP` 数据库。

主要用途：

* 应用数据库，类似于 MySQL、Oracle
* 海量数据处理
* 数据平台

主要特点：

* 建模过程为可选步骤
* `JSON` 数据模型比较适合开发者
* 横向扩展能力可以支持很大数据量和并发

### MongoDB 和关系型数据库对比

|                | MongoDB                                          | RDBMS                        |
| -------------- | ------------------------------------------------ | ---------------------------- |
| 数据模型       | 文档模型                                         | 关系模型                     |
| 数据库类型     | OLTP                                             | OLTP                         |
| DML            | MQL/SQL                                          | SQL                          |
| 高可用实现方式 | 复制集                                           | 集群模式                     |
| 横向扩展能力   | 通过原生分片完善支持                             | 数据分区或者应用程序分区处理 |
| 索引支持       | B 树、全文索引、地理位置索引、多键索引、TTL 索引 | B+树                         |
| 数据容量       | 没有理论上线                                     | 千万、亿级                   |

## MongoDB 特色及优势

MongoDB 的优势：

* `JSON` 结构和对象模型接近，开发代码量低；
* `JSON` 的动态模型意味着更容易响应新的业务需求；
* 复制集提供 99.999% 的高可用能力；
* 分片架构支持海量数据和无缝扩容。

### 面向开发者的易用且高效数据库

简单直观：以自然的方式建模，以直观的方式来与数据库交互；

结构灵活：弹性模式从容响应需求的频繁变化；

MongoDB 的灵活体现在三个方面：

* 多型性：同一个集合中可以包含不同字段（类型）的文档对象；

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/161042.png" alt="image-20220521161042381" style="zoom:67%;" />

* 动态性：线上修改数据模式，修改时应用和数据库均无须下线；

* 数据治理：支持使用 `JSON Schema` 来规范数据模式。在保证数据模式灵活动态的前提下，提供数据治理能力。

快速开发：以更少的代码完成功能，专注于业务逻辑。

#### 原生的高可用和横向扩展能力

MongoDB 通过复制集提供了高可用能力。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/161313.png" alt="image-20220521161313465" style="zoom:50%;" />

MongoDB 通过分片提供横向扩展能力。特点有：对应用全透明、提供多种数据分布策略、轻松支持 TB ～ PB 的数量级。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/161656.png" alt="image-20220521161656509" style="zoom:50%;" />

## 安装 MongoDB

### 下载 MongoDB

社区版下载地址：https://www.mongodb.com/try/download/community

官方安装教程：https://www.mongodb.com/docs/manual/administration/install-community/

以在 CentOS7 虚拟机上安装使用 MongoDB 社区版二进制包为例：

![image-20220521164016935](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/164016.png)

#### 获取可执行文件地址

![image-20220521163714914](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/163715.png)

在 MongoDB 下载界面中选择「On-permises」—「MongoDB Community Server」，选择合适的版本及平台后，点击「Copy Link」复制下载连接：

`https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-5.0.8.tgz`

#### 下载 MongoDB 可执行文件

创建 MongoDB 下载目录和数据文件存放目录：

```bash
mkdir -p /data/db
```

下载并解压 MongoDB 可执行文件：

```bash
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-5.0.8.tgz
tar -xf mongodb-linux-x86_64-rhel70-5.0.8.tgz
```

将 MongoDB 可执行文件加入 `PATH` 中：

```bash
export PATH=$PATH:/data/mongodb-linux-x86_64-rhel70-5.0.8/bin
```

![image-20220521165234017](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/165234.png)

#### 启动 MongoDB

启动 MongoDB ：

```bash
mongod --dbpath /data/db --port 27017 --logpath /data/db/mongod.log --fork -bind_ip 0.0.0.0
```

连接到 MongoDB：

```bash
mongo localhost:27017
```

开放 27017 端口，供外部访问：

```bash
firewall-cmd --add-port=27017/tcp --permanent
firewall-cmd --reload
```

![image-20220521165300181](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/165300.png)

### 使用 Atlas 免费帐号

[Atlas](https://www.mongodb.com/atlas) 是 MongoDB 提供的数据托管平台，并且给开发者提供了一个免费测试帐号。

#### 注册帐号

在 `https://www.mongodb.com/cloud/atlas/register ` 中输入基本信息，然后验证邮箱即可：

![image-20220521165831287](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/165831.png)

#### 创建 Atlas

![image-20220528091807871](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202206/150813.png)

![image-20220528092727567](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202206/150948.png)

![image-20220528092925782](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202206/151039.png)

![image-20220528093224034](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202206/151104.png)

添加允许连接的 IP 列表，可以使用 `0.0.0.0/0` 表示允许任意 IP 连接。

![image-20220528093246682](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202206/151131.png)

![image-20220528093335144](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202206/151156.png)

#### 连接到 Cluster

在 Altas 控制台中点击「Connect」按钮，在「Setup connection security」中选择通过哪种方式连接。这里选择使用 Java 客户端连接，故选择「Connect your application」。

![image-20220528094354092](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202206/151219.png)

选择对应的 Java 驱动版本，复制生成的连接字符串到应用程序中。

![image-20220528093435177](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202206/151246.png)

连接到 Cluster 和 连接本地 MongoDB了一样，示例代理如下：

```java
package store.xianglin.mongo;

import com.mongodb.client.MongoClients;
import org.bson.Document;

/**
 * 连接 Mongo Atlas
 *
 * @author xianglin
 */
public class QuickStartClusterExample {
    public static void main(String[] args) {
        final var connectionString = "mongodb+srv://xianglin:<password>@cluster0.boqb4.mongodb.net/?retryWrites=true&w=majority";

        try (var mongoClient = MongoClients.create(connectionString)) {
            final var database = mongoClient.getDatabase("mock");
            final var collection = database.getCollection("order");
            collection.insertOne(new Document("key1", "value1").append("key2", "value2"));

            var iter = collection.find();
            iter.forEach(System.out::println);
        }
    }
}
```

随后到 Atlas 控制台中查看刚刚插入的记录：

![image-20220528094151776](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202206/151313.png)

### 导入样本数据

导入课程提供的样本数据：

`mongorestore` 命令在 [Database Tools](https://www.mongodb.com/try/download/database-tools) 中，需要单独安装，或者按照官方教程安装。

```bash
curl -O -k https://github.com/geektime-geekbang/geektime-mongodb-course/blob/master/aggregation/dump.tar.gz

tar -xf dump.tar.gz

mongorestore -h localhost:27017
```

![image-20220521175822744](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/175822.png)

### 下载和使用 MongoDB Compass

下载地址：https://www.mongodb.com/try/download/compass

我这里使用 Manjaro 的包管理工具安装：

```bash
yay mongodb-compass
```

然后连接到刚刚启动的 MongoDB：

![image-20220521174230040](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/174230.png)

![image-20220521175532012](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/175532.png)

## MongoDB 基本操作

MongoDB 基本操作细节请参考官方文档：[MongoDB CRUD Operations](https://www.mongodb.com/docs/manual/crud/)。

CRUD 的基本命令如下：

Create：

```shell
db.collection.insertOne()
db.collection.insertMany()
```

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/181725.png" alt="image-20220521181725818" style="zoom:50%;" />

Read：

```shell
db.collection.find()
```

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/181801.png" alt="image-20220521181801072" style="zoom:50%;" />

Update：

```shell
db.collection.updateOne()
db.collection.updateMany()
db.collection.replaceOne()
```

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/181827.png" alt="image-20220521181827147" style="zoom:50%;" />

Delete：

```shell
db.collection.deleteOne()
db.collection.deleteMany()
```

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/181854.png" alt="image-20220521181854587" style="zoom:50%;" />

## 使用 MongoDB 开发程序

Java API 的使用细节请参考官方教程：[MongoDB Java Driver](https://www.mongodb.com/docs/drivers/java/sync/current/)。

### 创建简单应用

编写一个简单的 Java 应用查询一条样本数据，数据如下：

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/211322.png" alt="image-20220521211322686" style="zoom:50%;" />

#### 创建 Maven 项目

在 IDEA 中使用 Maven 创建一个简单 Java 应用：

![image-20220521205556188](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/205556.png)

引入 MongoDB Java 驱动的依赖：

```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>4.6.0</version>
</dependency>
```

#### 查询导入的样本数据

```java
package store.xianglin.mongo;

import com.mongodb.client.MongoClients;
import com.mongodb.client.model.Filters;

import java.util.Objects;

/**
 * connect mongo use Java
 *
 * @author xianglin
 */
public class QuickStart {
    public static void main(String[] args) {
        final var connectionString = "mongodb://192.168.56.101:27017/mock";
        try (final var mongoClient = MongoClients.create(connectionString)) {
            final var dataBaseMock = mongoClient.getDatabase("mock");
            final var collectionOrders = dataBaseMock.getCollection("orders");
            var documentOrder = collectionOrders.find(Filters.eq("street", "493 Hilll Curve")).first();
            System.out.println(Objects.requireNonNull(documentOrder).toJson());
        }
    }
}
```

#### 使用 `POJOs` 查询

`Document` 的实现是一个 `LinkedHashMap`，实际上使用时，更倾向于使用 POJO。先创建一个 `Order` 的实体类（省略 `getter` 和 `setter` 方法）：

```java
public class Order {
    private String street;
    private String city;
    private String state;
    private String country;
    private String zip;
    private String phone;
    private String name;
    private int userId;
    private LocalDateTime orderDate;
    private String status;
    private BigDecimal shippingFee;
    private List<OrderLine> orderLines;
    private BigDecimal total;

    public static class OrderLine {
        private String product;
        private String sku;
        private int qty;
        private BigDecimal price;
        private BigDecimal cost;
    }
}
```

需要注意的有两点：

* MongoDB 中 `Date` 类型的 `orderDate` 在 Java 中可以使用 `java.timeLocalDateTime` 也可以使用 `java.util.Date`；
* MongoDB 中的 `Decimal128` 对应 Java 中的 `java.math.BigDecimal`。

使用 `Order` 查询的方法如下：

```java
package store.xianglin.mongo;

import com.mongodb.MongoClientSettings;
import com.mongodb.client.MongoClients;
import com.mongodb.client.model.Filters;
import org.bson.codecs.configuration.CodecRegistries;
import org.bson.codecs.pojo.PojoCodecProvider;

/**
 * connect mongo use java with pojos
 *
 * @author xianglin
 */
public class QuickStartPojoExample {
    public static void main(String[] args) {
        final var pojoCodecProvider = PojoCodecProvider.builder().automatic(true).build();
        final var codecRegistries = CodecRegistries.fromRegistries(MongoClientSettings.getDefaultCodecRegistry(), CodecRegistries.fromProviders(pojoCodecProvider));

        final var connectionString = "mongodb://192.168.56.101:27017/mock";
        try (final var mongoClient = MongoClients.create(connectionString)) {
            final var dataBaseMock = mongoClient.getDatabase("mock").withCodecRegistry(codecRegistries);
            final var collectionOrders = dataBaseMock.getCollection("orders", Order.class);
            var order = collectionOrders.find(Filters.eq("street", "493 Hilll Curve")).first();
            System.out.println(order);
        }
    }
}
```

