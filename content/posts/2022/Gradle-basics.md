---
title: Gradle基础
date: 2022-11-06 22:18:16
tags: 
  - gradle
  - groovy
description: gradle 的基础用法，以及 groovy 与 Java 语法的区别。
summary: gradle 的基础用法，以及 groovy 与 Java 语法的区别。
categories: [learn]
---

# Gradle 基础

官网：[Gradle](https://gradle.org/)

## 安装

Gradle 需要 JDK8 及以上版本。

从 [releases](https://gradle.org/releases/) 页面下载 Gradle 二进制包，解压后添加环境变量即可。使用 `gradle -v` 验证安装。

![08_1667879688](https://fastly.jsdelivr.net/gh/xianglin2020/gallery@master/202211/202211092254536.png)

## 创建 Java 应用

### 使用 `init` 任务创建项目

创建空文件夹 `gradleJava` 保存项目。

在 `gradleJava` 文件夹中执行 `gradle init` 命令，根据提示完成创建。

![08_1667880245](https://fastly.jsdelivr.net/gh/xianglin2020/gallery@master/202211/202211092254312.png)

由 Gradle 创建的 Java 应用结构如下：

![08_1667880412](https://fastly.jsdelivr.net/gh/xianglin2020/gallery@master/202211/202211092254752.png)

`gradle`：gradle wrapper 目录；

`gradlew`、`gradlew.bat`：gradle wrapper 启动脚本；

`settings.gradle`：配置文件，定义了应用名称和子项目；

`app/build.gradle`：app 项目的构建脚本；

`app/src/main(test)/java`：Java 源（测试）代码路径。

### 项目文件介绍

`settings.gradle` 内如如下：

```groovy
rootProject.name = 'gradleJava'
include('app')
```

* `rootProject.name`：指定应用名称。
* `include()`：指定该应用包含的子项目。

子项目 `app` 的配置文件是：`app/build.gradle`，内容如下：

```groovy
plugins {
    id 'application'
}
repositories {
    // 使用 Maven 中央仓库
    mavenCentral()
}
dependencies {
    // 指定依赖
    testImplementation 'org.junit.jupiter:junit-jupiter:5.8.2'
    implementation 'com.google.guava:guava:31.0.1-jre'
}
application {
    mainClass = 'store.xianglin.gradle.App'
}
tasks.named('test') {
    useJUnitPlatform()
}
```

# Groovy 与 Java 比较

## 默认导入

groovy 默认导入如下包和类：

* `java.io.*`
* `java.lang.*`
* `java.net.*`
* `java.util.*`
* `java.math.BigInteger`
* `java.math.BigDecimal`
* `groovy.lang.*`
* `groovy.util.*`

## 运行时参数类型匹配

在 groovy 中，重载方法根据运行时参数的实际类型选择，Java 中重载方式基于编译时静态声明的参数类型选择。
比如如下重载方法在 groovy 和 Java 中结果不一致：

```groovy
int method(String arg) {
    1
}

int method(Object arg) {
    2
}

Object o = "object"
// result: 1
int result = method(o)
```

```java
public class Main {
    int method(String arg) {
        return 1;
    }

    int method(Object arg) {
        return 2;
    }

    public static void main(String[] args) {
        Object o = "object";
        // result: 2
        int result = new java.MainJava().method(o);
    }
}
```

## 创建数组

Java 中初始化数组有如下两种方式：

```java
class Array {
    // 简短语法
    int[] shortArr = {1, 2, 3};
    // 完整语法
    int[] longArr = new int[]{1, 2, 3};
}
```

Groovy 中 `{...}` 代码块称为闭包，所以不能使用 Java 的简短语法创建 Groovy 数组。Groovy 中初始化数组有以下两种方式：

```groovy
int[] shortArr = [1, 2, 3]
int[] longArr = new int[]{1, 2, 3}
```

## 包可见性

Java 中省略修饰符时为 `default` 包可见。Groovy 中属性省略修饰符时，代表一个私有字段，并自动关联 setter 和 getter 方法。
使用 `@PackageScope` 声明一个包可见性的字段。

```groovy
// package g
class Scope {
    String name
    @PackageScope
    int age
}

//package g.s
import g.Scope

new Scope().getName()
```

## ARM 代码块

Java 中 Automatic Resource Management (`try-with-resources`) 如下：

```java
public class Main {
    public static void main(String[] args) {
        try (var reader = Files.newBufferedReader(Paths.get("file"), StandardCharsets.UTF_8)) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

Groovy 通过各种使用闭包参数的方法支持 ARM，如下：

```groovy
new File("file").eachLine { print it }
new File("file").withReader('UTF-8') { reader ->
    {
        reader.eachLine { print it }
    }
}
```

## Lambda 表达式和方法引用操作符

Java8 和 Groovy 都支持 Lambda 和 method reference operator `::` 如下：

```java
public class Main {
    public static void main(String[] args) {
        Runnable run = () -> System.out.println("lambda");
        List.of(1, 2, 3).forEach(System.out::println);
    }
}
```

```groovy
Runnable runnable = () -> println "lambda"
List.of(1, 2, 3).forEach(System.out::println)
List.of(1, 2, 3).forEach({ print it })
```

## GStrings

Groovy 支持单引号字符串和双引号字符串，双引号字符串的类型时 `GString`，如下：

```groovy
String s = "double"
String s1 = 'singly-quoted string'
def s2 = "${s}-quoted string"
// class java.lang.String
println s1.getClass()
// class org.codehaus.groovy.runtime.GStringImpl
println s2.getClass()
```

## String 和 Character 字面量

单引号字面量用 `String` 表示，双引号字面根据是否有插值 `${}` 使用 `String` 或 `GString` 表示。
单个字符仅仅在使用 `char` 声明时才表示字符，如下：

```groovy
def s = 's'
// class java.lang.String
println s.getClass()
char c = 'c'
// class java.lang.Character
println c.getClass()
```

Groovy 中强制类型转换如下：

```groovy
assert ((char) 'c').class == Character
assert ('c' as char).class == Character
assert 'cx'.asType(Character) == 'c'
```

## `==` 行为

Java 中使用 `==` 比较基本类型和对象引用是否相等。
Groovy 中使用 `==` 比较非基本类型时，如果对象实现了 `Comparable` 接口则使用 `a.compareTo(b) == 0`，否则使用 `a.equals(b)`。
比较对象引用是否相等使用 `is` 方法或者运算符 `===`，如下：

```groovy
def s1 = "123"
def s2 = "12" + "3"
// true
println s1 == s2
// false
println s1 === s2
println s1.is(s2)
```

## 基本类型和包装类型

Java 在处理方法入参和返回值时会自动装箱和拆箱，如下：

```java
public class Main {
    public static void main(String[] args) {
        var i1 = 1;
        var i2 = 2;
        var result = add(i1, i2);
    }

    static int add(Integer i1, Integer i2) {
        return i1 + i2;
    }
}
```

在 Groovy 中，基本类型可以直接使用其包装类型的方法，例如 `true.toString()`，如下方法在 groovy 和 Java 中结果不一致：

```groovy
int i = 1
m(i)
// java 中类型提升会优先于装箱操作
void m(long l) {
    println "in m(long)"
}

// groovy 所有基本类型都是使用对应的包装类
void m(Integer i) {
    println "in m(Integer)"
}
```

## 额外的关键字

Groovy 有如下额外的关键字：`as`、`def`、`in`、`trait` 和 `it`（仅在闭包中）。
Groovy 同样支持局部变量类型推断 `var`。
