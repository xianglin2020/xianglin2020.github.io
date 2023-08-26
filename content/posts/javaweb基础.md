---
title: Java Web 基础
date: 2020-09-06 22:54:32
categories: [learn]
tags: [java, web]
---

# XML入门

## XML的作用

* XML（EXtensible Makup Language）是可扩展标记语言
* 编写XML就是编写标签，XML没有预定义标签
* XML重在保存和传输数据
* 良好的可读性

### XML用途

1. 用于Java程序的配置文件（`web.xml`）
2. 用于保存、传输程序产生的数据（EJB调用、WebService的soap协议）

## XML的语法规则

1. XML第一行必须是XML声明；XML声明说明XML文档的基本信息，包括版本号与字符集。

   `<?xml version="1.0" encoding="UTF-8"?>`

2. XML只有一个根节点

### XML书写规则

* 合法的标签名

* 适当的注释与缩进可以让XML文档易于阅读理解

* 合理使用属性

* 处理特殊字符：

  1. 实体引用：

     | 实体引用 | 对应符号 | 说明   |
     | -------- | -------- | ------ |
     | `&lt;`   | `<`      | 小于   |
     | `&gt;`   | `>`      | 大于   |
     | `&amp;`  | `&`      | 和号   |
     | `&apos;` | `''`     | 单引号 |
     | `&quot;` | `""`     | 双引号 |

  2. 使用CDATA标签：CDATA指的是不应该由XML解析器进行解析的文本数据

     `<![CDATA[1+1<3]]>`

* 有序的子元素

## Java解析XML

* DOM文档对象模型：定义了访问和操作XML文档的标准方法
* Dom4j是一个易用的，开源的库，用于解析XML。
* Dom4j将XML视为一个Document对象
* XML标签被Dom4j定义为Element对象

## XML语义约束

* XML文档结构正确，但可能不是有效的
* XML语义约束的两种方式：DTD、Schema

### DTD

* DTD（Document Type Definition），是一种简单易用的语义约束方式。
* DTD文件的扩展名为`.dtd`
* 在XML中使用`<!DOCTYPE>`标签来引用DTD
* 利用DTD中的`<!ELEMENT>`标签，定义XML中运行出现的节点及数量

### XML Schema

* XML Schema提供了数据类型，格式限定，数据范围等特性
* XML Schema是W3C标准

## XPath路径表达式

* XPath路径表达式是XML文档中查找数据的语言

* 最常用的基本表达式

  | 表达式     | 描述                                                 |
  | ---------- | ---------------------------------------------------- |
  | `nodename` | 选取此节点的所有子节点                               |
  | `/`        | 从根节点选取                                         |
  | `//`       | 从匹配的当前节点选择文档中的节点，而不考虑它们的位置 |
  | `.`        | 选取当前节点                                         |
  | `..`       | 选取当前节点的父节点                                 |
  | `@`        | 选取属性                                             |

* XPath谓语表达式

  ![](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225704.png)

* Jaxen：Jaxen的是一个Java编写的开源的XPath库。

* 适应多种不同的对象模型，包括DOM，XOM，dom4j和JDOM。

# Servlet入门

### J2EE

* J2EE（Java 2 Platform Enterprise Edition）是指Java 2 企业版
* J2EE是一组技术规范与指南，具体实现由软件厂商实现
* 开发Web应用程序是J2EE最核心功能
* J2EE是由13个功能模块组成的
  * Servlet、JSP、JDBC、XML、EJB、RMI、JNDI、JMS、JTA、JavaMail、JAF、CORBA、JTS

### Tomcat

* Tomcat是J2EE Web标准实现

## Servlet 开发技巧

* Servlet（Server Applet）服务器小程序，主要用于生成动态Web内容

* 标准Java WEB工程目录

  ![](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202008/235805.png)

### Servlet开发步骤

1. 创建类，继承自`javax.servlet.http.HttpServlet`
2. 在`service`方法中处理事件
3. 在`web.xml`中配置servlet访问路径和servlet实现类的映射

## Servlet执行原理

### Servlet生命周期

1. 装载--扫描`web.xml`和`@Servlet`注解
2. 执行构造函数创建实例
3. 执行初始化方法`init()`进行初始化
4. 通过`service()`方法提供服务
5. Servlet销毁时执行`destroy()`方法

### 使用注解简化Servlet配置

* 在Servlet3.x之后引用了注解特性
* Servlet核心注解：`@Servlet`

### 启动时加载Servlet

* 在`web.xml`中使用`<load-on-startup>`
* 启动时加载在工作中常用于系统预处理

## Servlet核心对象

# JSP入门

## JSP用途

### Servlet的缺点

* 静态的HTML和和Java代码柔和在一起，难以维护
* Servlet利用`out.pringln()`输出HTML代码，效率低下
* JSP全称是：Java Server Page，本质就是Servlert

## JSP执行原理

### JSP的执行过程

* 浏览器请求
* Tomcat找到对应的JSP（index.jsp），将其转译为Servlet ( index_jsp.java)
* Servlet编译成Java字节码 (index_jsp.class)

## JSP基本语法规则

### JSP代码块

用于在JSP中嵌入Java代码

JSP代码块的语法：`<% code %>`

### JSP声明构造块

JSP声明构造块用于定义声明方法和变量

JSP声明语法块语法：`<%! code %>`

### JSP输出指令

JSP输出指令用于在JSP页面中显示Java代码执行的结果

JSP输出指令语法：`<%= code %>`

### JSP处理指令

JSP处理指令用于提供JSP执行过程中的辅助信息

| 指令           | 描述                           |
| -------------- | ------------------------------ |
| `<%@page%>`    | 定义当前JSP页面的全局设置      |
| `<%@include%>` | 将其他JSP页面与当前JSP页面合并 |
| `<%@taglib%>`  | 引入JSP标签库                  |

### JSP中的注释

1. JSP注释：`<%-- statment --%>`
2. Java注释：`// /* */`
3. HTML注释：`<!-- -->`

## JavaWeb核心特性

### Cookies

### Session

### ServletContext

### Tomcat乱码问题

* Tomcat默认使用ISO-8859-1编码规则
* Tomcat的`server.xml`中配置`URLEncoding="UTF-8"`

## JSP九大内置对象

![image-20200802132136208](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202008/235816.png)

# JSTL和EL表达式

## EL表达式

* Expression Language，用于简化JSP的输出

* EL表达式的基本语法：`${expression}`

* 作用域对象，忽略作用域对象时，EL表达式默认会按作用域范围从小到大依次尝试获取

  ![image-20200802140924186](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202008/235821.png)

* EL表达式输出：`${[scope.]attr[.subAttr]}`

* EL表达式支持运算

## JSTL常用标签

* JSP Standard Tag Library，JSP标准标签库

* JSTL由Sun定义规范，由Apache Tomcat进行实现

* JSTL下载地址：https://tomcat.apache.org/download-taglibs.cgi

* JSTL按功能划分为五类标签库

  ![image-20200802205819340](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202008/235854.png)

* 核心标签库Core是JSTL标签库中最重要的标签库，提供了JSTL的基础功能

* `<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>`：`taglibs-standard-impl-1.2.5.jar!\META-INF\c.tld`

  * `<c:if>`单分支判断
  * `<c:when> <c:choose> <c:otherwise>`多分支判断
  * `<c:forEach>`遍历集合中的每一个对象

* 格式化标签库

* `<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>`

  * `<fmt:formatDate>`
  * `<fmt:formatNumber>`
