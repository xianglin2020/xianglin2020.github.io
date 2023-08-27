---
title: Java 反射
description: 反射是在运行时动态访问类与对象的技术。
summary: 反射是在运行时动态访问类与对象的技术。
date: 2020-09-12 18:23:05
categories: [learn]
tags: [java]
---

# Java反射

## 反射概念

* 反射是在运行时动态访问类与对象的技术
* 反射是JDK1.2版本后的高级技术，隶属于`java.lang.reflect`包中
* 大多数Java 框架都基于反射实现参数配置，动态注入等特性

## 反射的核心类

* `Class`
* `Constructor`
* `Method`
* `Field`

### `Class`

* `Class`类是JVM 中表示“类和接口”的类
* `Class` 对象具体包含了某个特定类的结构信息
* 通过`Class`对象可以获取对应类的方法和成员信息

```java
public static void main(String[] args) {
  try {
    Class<?> employeeClass = Class.forName("reflect.entity.Employee");
    System.out.println("Employee已被加载！");
    Employee employee = (Employee) employeeClass.newInstance();
    System.out.println(employee);
    // 类不存在时抛出ClassNotFoundException 访问私有成员是抛出IllegalAccessException 类无法实例化时抛出InstantiationException
  } catch (ClassNotFoundException | IllegalAccessException | InstantiationException e) {
    e.printStackTrace();
}
```

`Constructor`

* `Constructor`类是对 Java类中构造方法的抽象
* 包含了类某个具体构造方法的声明
* 调用 Constructor 带参构造方法创建对象

```java
public static void main(String[] args) {
    try {
        Class<?> employeeClass = Class.forName("reflect.entity.Employee");
        Constructor<?> constructor = employeeClass.getConstructor(Integer.class, String.class, Float.class, String.class);
        Employee employee = (Employee) constructor.newInstance(1, "姓名", 300.4F, "部门");
        System.out.println(employee);
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (NoSuchMethodException e) {
        // 找不到特定方法时引发
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        // 无访问权限
        e.printStackTrace();
    } catch (InstantiationException e) {
        // 无法实例化指定的类对象时引发
        e.printStackTrace();
    } catch (InvocationTargetException e) {
        // 当被调用方法内部抛出异常而没有被捕获时
        e.printStackTrace();
    }
}
```

### `Method`

```java
public static void main(String[] args) {
  try {
    Class<?> aClass = Class.forName("reflect.entity.Employee");
    Constructor<?> constructor = aClass.getConstructor(Integer.class, String.class, Float.class, String.class);
    Employee employee = (Employee) constructor.newInstance(1, "姓名", 123F, "部门");
    Method method = aClass.getMethod("updateSalary", Float.class);
    employee = (Employee) method.invoke(employee, 100F);
    System.out.println(employee);
  } catch (ClassNotFoundException | InvocationTargetException | IllegalAccessException | InstantiationException | NoSuchMethodException e) {
    e.printStackTrace();
  }
}
```

### `Field`

* `Field`对应类中的成员变量的声明
* 可以通过`Field`类对成员变量赋值和取值

```java
public static void main(String[] args) {
    Class<Employee> aClass = Employee.class;
    String ename = null;
    try {
        Constructor<Employee> constructor = aClass.getConstructor(Integer.class, String.class, Float.class, String.class);
        Employee employee = constructor.newInstance(1, "姓名", 3000F, "部门");
        Field field = aClass.getField("ename");
        field.set(employee, "李磊");
        ename = (String) field.get(employee);
    } catch (NoSuchMethodException | InvocationTargetException | IllegalAccessException | InstantiationException e) {
        e.printStackTrace();
    } catch (NoSuchFieldException e) {
        // 表示类没有指定名称的字段
        e.printStackTrace();
    }
    System.out.println("ename: " + ename);
}
```

