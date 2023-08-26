---
title: java 设计模式
date: 2020-09-06 22:50:27
tags: [java, 设计模式]
categories: [learn]
---

# 设计模式

* 设计模式是前人总结的设计经验
* 设计模式的目标是代码更容易理解和维护
* 设计模式可以让代码更可靠

## 设计模式分类

### 创建型模式



### 结构型模式

### 行为型模式



## 单例模式

## 工厂模式

* 工厂模式用于隐藏创建对象的细节
* 工厂模式的核心：工厂类（factory）
* 工厂模式可细分为：简单工厂、工厂方法、抽象工厂

### 简单工厂

![image-20200913121241287](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/121241.png)

* 编写一个自动按区域获取标题语言的工厂方法

  1. 抽象出一个接口，用于获取标题描述

     ```java
     public interface I18N {
         public String getTitle();
     }
     ```

  2. 按不同区域实现各自的方法，返回标题描述

     ```java
     public class Chinese implements I18N {
         @Override
         public String getTitle() {
             return "图书";
         }
     }
     
     public class English implements I18N{
         @Override
         public String getTitle() {
             return "book";
         }
     }
     ```

  3. 使用静态方法，按照传入的区域自动创建返回标题的对象

     ```java
     public class I18NFactory {
         public static I18N getI18N(String country) {
             return switch (country) {
                 case "China" -> new Chinese();
                 case "English" -> new English();
                 default -> null;
             };
         }
     }
     ```

  4. 使用工厂方法获取`I18N`对象

     ```java
     public class SoftWare {
         public static void main(String[] args) {
             I18N i18N = I18NFactory.getI18N("China");
             System.out.println(i18N.getTitle());
         }
     }
     ```

     
