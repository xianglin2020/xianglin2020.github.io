---
title: java数组
date: 2020-09-06 22:49:52
categories: learn
tags: java
---

# Java 数组

* 数组是相同类型的数据按顺序组成的一种引用数据类型

## 一维数组

### 数组声明

* 语法格式：`数据类型[] 数组名;`

### 数组创建

* 先声明后创建

  `数据类型[] 数组名;`

  `数组名 = new 数据类型[]{}`

* 声明时创建数组

  `数据类型[] 数组名 = new 数据类型[数组大小];`

### 数组在内存中的存储

* 数组会被分配连续的内存空间

### 数组的初始化

* 声明数组的同事给数组赋值，叫做数组的初始化

  `int[] arr = {1,2,3};`

### 数组元素的引用

* `数组名[下表]`
* 数组下表从 0 开始

### 数组长度

* 属性`length`表示数组的长度

## 二维数组