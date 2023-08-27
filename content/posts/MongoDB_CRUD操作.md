---
title: MongoDB-CRUD操作
date: 2022-06-26 09:21:48
tags: [MongoDB]
categories: [learn]
summary: MongoDB CRUD 基础操作。
description: MongoDB CRUD 基础操作。
---

# MongoDB 基本操作

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

## 插入文档

使用 `db.collection.insertOne()` 插入单条记录到集合中，如果插入的集合没有指定 `_id` 字段，MongoDB 会自动生成一个 `ObjectId`，如果集合不存在，那么插入时会创建对应的集合。

```shell
db.inventory.insertOne( { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } })
```

![image-20220521221500429](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/221500.png)

插入成功后会返回记录对应的 `_id`。

插入多条记录和插入单条记录一样，只需要使用 `db.collection.insertMany()`：

```shell
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
```

![image-20220521221939960](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/221940.png)

### 插入行为

如果集合不存在，插入操作会新建集合；

集合中的每条文档记录需要一个唯一的 `_id` 字段作为主键，如果插入的文档没有提供 `_id` 字段，MongoDB 会自动生成一个 `ObjectId` 类型的 `_id` 作为主键；

MongoDB 中的所有写操作在单个文档级别上都是原子的。

## 查询文档

查询选择器：[query-selectors](https://www.mongodb.com/docs/manual/reference/operator/query/#query-selectors)。

### 基本查询操作

使用如下测试数据：

```shell
db.inventory.insertMany([
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);
```

使用 `db.collection.find({})` 查询文档，查询所有记录：

```shell
db.inventory.find( {} )
```

指定相等条件查询：`status` 等于 `D`

```shell
db.inventory.find( { status: "D" } )
```

指定 `AND` 条件查询：`status` 等于 `A` 且 `qty` 小于 30

```shell
db.inventory.find({status: "A", qty:{$lt: 30}})
```

指定 `OR` 查询条件：`status` 等于 `A` 或 `qty` 小于 30

```shell
db.inventory.find({$or: [{status: "A"}, {qty: {$lt: 30}}]})
```

同时指定 `AND` 和 `OR` 查询条件：`status` 等于 `A` 并且 `qty` 小于 30 或者 `item` 以 `p` 开头：

```shell
db.inventory.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )
```

### 查询嵌套文档

使用如下测试数据：

```shell
db.inventory.insertMany( [
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);
```

 匹配嵌入文档（要求文档内的键值及顺序完全匹配）：`size` 等于 `{ h: 14, w: 21, uom: "cm" }`

```shell
db.inventory.find({size:{ h: 14, w: 21, uom: "cm" }})
```

查询嵌套字段：查询 `size` 中 `uom` 值为 `in` 的文档

```shell
db.inventory.find({'size.uom':'in'})
```

### 查询简单元素 Array

使用如下测试数据：

```shell
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
   { item: "notebook", qty: 50, tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
   { item: "paper", qty: 100, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
   { item: "planner", qty: 75, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
   { item: "postcard", qty: 45, tags: ["blue"], dim_cm: [ 10, 15.25 ] }
]);
```

匹配数组（要求数组值及顺序完全匹配）：`tags` 等于 `["blank", "red"]`

```shell
db.inventory.find({tags:['blank','red']})
```

匹配数组中部分元素（不要求值及顺序完全匹配）：`tags` 包含 `blank` 和 `red`

```shell
db.inventory.find({tags:{$all:['blank','red']}})
```

匹配数组中某个元素：`tags` 包含 `red`

```shell
db.inventory.find({tags:'red'})
```

`dim_cm` 中有大于 25 的元素：

```shell
db.inventory.find({dim_cm:{$gt:25}})
```

`dim_cm` 中包含大于 15 或小于 20 的元素：

```shell
db.inventory.find({dim_cm:{$gt:15,$lt:20}})
```

`dim_cm` 中包含大于 15 且小于 20 的元素：

```shell
db.inventory.find({dim_cm:{$elemMatch:{$gt:15,$lt:20}}})
```

按数组索引位置查询元素：`dim_cm` 中第二个元素大于 20

```shell
db.inventory.find({'dim_cm.1':{$gt:20}})
```

按数组长度查询数组：`tags` 数组有三个元素

```shell
db.inventory.find({tags:{$size:3}})
```

### 查询嵌套文档数组

使用如下测试数据：

```shell
db.inventory.insertMany( [
   { item: "journal", instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] },
   { item: "notebook", instock: [ { warehouse: "C", qty: 5 } ] },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 15 } ] },
   { item: "planner", instock: [ { warehouse: "A", qty: 40 }, { warehouse: "B", qty: 5 } ] },
   { item: "postcard", instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);
```

以文档形式匹配数组中某个元素：

```shell
db.inventory.find({instock:{warehouse:'A',qty:5}})
```

在文档数组中某个字段上指定查询条件：

```shell
db.inventory.find({'instock.qty':{$lt:20}})
```

在数组中指定索引的文档某个字段上指定查询条件：

```shell
db.inventory.find({'instock.0.qty':{$lte:20}})
```

### 查询 `null` 或不存在字段

使用如下测试数据：

```shell
db.inventory.insertMany([
   { _id: 1, item: null },
   { _id: 2 }
])
```

查询字段存在且等于 `null` 或者字段不存在：

```shell
db.inventory.find({item:null})
```

类型检查 [bson-types](https://www.mongodb.com/docs/manual/reference/bson-types/)，查询查询字段存在且等于 `null` ：

```shell
db.inventory.find({item:{$type:10}})
```

存在检查：

```shell
db.inventory.find( { item : { $exists: false } } )
```

## 更新文档

使用如下测试数据：

```shell
db.inventory.insertMany( [
   { item: "canvas", qty: 100, size: { h: 28, w: 35.5, uom: "cm" }, status: "A" },
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "mat", qty: 85, size: { h: 27.9, w: 35.5, uom: "cm" }, status: "A" },
   { item: "mousepad", qty: 25, size: { h: 19, w: 22.85, uom: "cm" }, status: "P" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" },
   { item: "sketchbook", qty: 80, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "sketch pad", qty: 95, size: { h: 22.85, w: 30.5, uom: "cm" }, status: "A" }
] );
```

[Update Operators](https://www.mongodb.com/docs/manual/reference/operator/update/)
