---
title: Lambda、Stream与函数式编程
date: 2020-09-12 18:24:41
categories: [learn]
tags: 
  - lambda
  - stream
---

# Lambda

* Java8 开始支持Lambda 表达式，让程序编写更简洁、优雅
* 利用 Lambda 表达式可以更简洁的实现匿名内部类和函数声明与调用
* Lambda 使得 Stream 流式编程为操作集合提供便利

## Lambda 表达式语法结构

* `(参数列表) -> 实现语句`

```java
public static void main(String[] args) {
        // 标准使用方式
        MathOperation operation = (double a, double b) -> {
            System.out.println("加法运算");
            return a + b;
        };
        System.out.println(operation.opt(1, 3));

        // Lambda 允许忽略参数类型
        MathOperation operation1 = (a, b) -> {
            return a - b;
        };
        System.out.println(operation1.opt(4, 1));

        // 单行代码可以省略方法体和 return 语句
        MathOperation operation2 = (a, b) -> a * b;
        System.out.println(operation2.opt(2, 5));
    }
```

# 函数式编程

* 基于函数式接口并使用 Lambda 表达式的编程方式
* 函数式编程的理念是将代码作为可重用数据代入到程序运行中

### 函数式接口

* 有且仅有一个抽象方法的接口
* Java8 提供了一系列新的函数式接口，位于`java.util.function`包中
* `Predicate`用于判断传入的数据是否满足要求，使用`test()`方法进行逻辑判断

1. `java.util.function.Predicate` :用于条件判断，返回 Boolean 值
2. `java.util.function.Consumer`：对应只有一个输入参数，而不需要返回的情况
3. `java.util.function.Function`：对应一个输入参数，且返回数据的情况

# Stream

* `Stream`流式处理是建立在 Lambda 表达式基础上的多数据处理技术
* Stream 对集合数据处理高度抽象，极大简化代码量
* Stream 可以对集合数据进行迭代、去重、筛选、排序、聚合等操作
* 位于`java.util.stream.Stream`

## 创建流的五种方式

* 基于数组创建

  ```java
  String[] arr = {"lili", "mark", "jackson"};
  Stream<String> stringStream = Stream.of(arr);
  stringStream.forEach(System.out::print);
  ```

* 基于集合创建

  ```java
  List<String> list = new ArrayList<>();
  list.add("123");
  list.add("345");
  Stream<String> stream = list.stream();
  stream.forEach(System.out::println);
  ```

* 使用 `generate` 方法创建

  ```java
  Stream<Integer> generate = Stream.generate(() -> new Random().nextInt(100000)).limit(100);
  generate.forEach(System.out::print);
  System.out.println();
  ```

* 使用迭代器创建

  ```java
  Stream<Integer> limit = Stream.iterate(1, n -> n + 1).limit(100);
  limit.forEach(System.out::print);
  ```

* 使用字符序列创建

  ```java
  String string = "abcdefg";
  IntStream chars = string.chars();
  chars.forEach(System.out::println);
  ```

