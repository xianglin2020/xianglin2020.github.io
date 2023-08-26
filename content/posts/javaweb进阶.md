---
title: Java Web 进阶
date: 2020-09-06 22:54:39
categories: [learn]
tags: [java, web]
---

# JSON入门

## JSON语法规则

* JavaScript Object Notation
* JSON是轻量级的数据交换格式
* 独立于语言、自解释、易于理解书写

## JavaScript与JSON

* `JSON.parse()`将json格式的字符串转为JSON对象
* `JSON.stringify()`将JSON对象转为json字符串

## Java与JSON

* Java 的JSON工具包：FastJson、Jackson、Gson、Json-lib
* fastjson是阿里巴巴的开源JSON解析库，它可以解析JSON格式的字符串，支持将Java Bean序列化为JSON字符串，也可以从JSON字符串反序列化到JavaBean。
* `JSON.toJSONString(Object object)`
* `JSON.parseObject(String text, Class<T> clazz)`
* `JSON.parseArray(String text, Class<T> clazz)`

# jQuery&Ajax

## jQuery使用

* jQuery是一个轻量级的js库，使用简便
* jQuery的核心是选择器，用于获取页面的元素
* jQuery[官网](https://jquery.com/)，采用独立JS文件发布

### jQuery选择器

* jQuery选择器用于选中需要操作的页面元素：`$(expression)`

* jQuery基础选择器

  ![image-20200808105406866](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202008/235955.png)

* jQuery层叠选择器

  ![image-20200808110521981](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225741.png)

* jQuery属性选择器

  ![image-20200808111327519](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225744.png)

* 位置选择器

  ![image-20200808125829925](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/20200808125830.png)

* 表单选择器

  ![image-20200808130313340](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225746.png)

### jQuery操作元素

* 操作元素的属性：`$().attr()`、`$().removeAttr(name)`
* 操作元素的样式：`css()`、`addClass()`、`removeClass()`
* 操作元素的内容：`val()`、`text()`、`html()`

### jQuery处理事件

* `on('eventName',function(event){})`

* jQuery常用事件

  ![image-20200808134153452](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225747.png)

## Ajax原理

* Ajax:Asynchronous JavaScript and XML（异步的JavaScript和XML）

### XMLHttpRequest使用步骤和[属性及方法](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)

  1. 创建 XMLHttpRequest

  2. 发送请求

  3. 处理服务类响应

     ```javascript
      		let xhr;
         // 创建请求
         if (window.XMLHttpRequest) {
           xhr = new XMLHttpRequest();
         } else {
           xhr = new ActiveXObject("Microsoft.XMLHTTP");
         }
         // 发送请求
         xhr.open('GET', 'http://localhost:8080/ajax/ajax', true);
         xhr.send();
         // 处理响应
         xhr.onreadystatechange = function () {
           // 响应已被接收且服务器处理成功
           if (xhr.readyState === 4 && xhr.status === 200) {
             document.getElementById("divContent").innerHTML = xhr.responseText;
           }
         }
     ```

* XMLHttpRequest状态

  ![image-20200808215829083](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225748.png)

### jQuery 对 Ajax 的支持

* Ajax 常用设置项

  ![截屏2020-08-08 22.32.33](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225750.png)

* 使用方法：

  ```javascript
  $.ajax({
    url: '/ajax/news',
    type: 'get',
    dataType: 'json',
    success: function (json) {
      console.log(json);
      $.each(json, (i, news) => {
        console.log(news);
        console.log(i);
        let content = $('#divContent');
        content.html(content.html() + `<h2>${i + 1}.${news.title}</h2><br><p>${news.date}:${news.content}</p>`);
      });
    }
  })
  ```

  

# Java正则表达式

## 正则表达式语法

* 正则表达式是检查、匹配字符串的表达式

* 字符串的查找、替换是正则表达式的主要用途

* 字符范围匹配，正则表达式[测试工具](https://tool.oschina.net/regex)

  ![image-20200809101941816](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202008/101942.png)

* 元字符

  ![image-20200809102743977](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225754.png)

* 多次重复匹配

  ![image-20200809103658986](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225755.png)

* 定位匹配

  ![image-20200809104705932](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225756.png)

* 贪婪模式和非贪婪模式

* 表达式分组

## 使用正则表达式

* Java 使用正则表达式

  ```java
  public class RegexSample {
      // 1. 创建正则表达式
      private static final Pattern PATTERN = Pattern.compile("<li>([\\u4e00-\\u9fa5]{2,8})([a-zA-Z]{1,10})</li>");
  
      public static void main(String[] args) throws IOException {
          // 2. 获取内容
          String source = "/Users/xianglin/Documents/IdeaProjects/JavaWeb/json-ajax-reg-filter-listener/web/sample.html";
          BufferedReader reader = new BufferedReader(new FileReader(source, StandardCharsets.UTF_8));
          StringBuilder content = new StringBuilder();
          String lineText;
          while ((lineText = reader.readLine()) != null) {
              content.append(lineText);
          }
          // 3. 匹配正则表达式
          Matcher matcher = PATTERN.matcher(content);
          // 4. 分析处理结果
          while (matcher.find()) {
              // 匹配完整字符串
              System.out.println(matcher.group(0));
  
              String chineseName = matcher.group(1);
              String englishName = matcher.group(2);
              System.out.println(chineseName + "-" + englishName);
          }
  
          reader.close();
      }
  }
  ```

  

# 过滤器

## 过滤器的作用和使用场景

* 过滤器是 J2EE Servlet 模块下的标准组件
* Filter 的作用是对 URL进行统一的拦截处理
* Filter通常用于应用程序层面的全局处理
* 任何过滤器都要实现接口：`javax.servlet.Filter`
* 在Filter 实现类中的`doFilter`方法编写过滤器的功能代码
* 在 `web.xml `中对过滤器进行配置，说明拦截 URL 的范围
* 在`web.xml`中配置`<init-param>`为 Filter 提供参数

## 过滤器开发和原理

### 过滤器的生命周期

1. 初始化：Filter 创建 `Filter.init()`
2. 提供服务：`Filter.doFilter()`
3. 销毁：`Filter.destroy()`

### 过滤链

* 每个过滤器应该只有单独的职能
* 过滤器的执行顺序是以`<filter-mapping>`配置为准
* 注解形式的过滤链是按照过滤器的类名的字母升序的顺序依次执行的而不是filterName名称
* ![image-20200809134059344](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/225758.png)



# 监听器

## 监听器的作用

* 监听器 Listener 是 J2EE Servlet 中的标准模块
* Listener的作用是对Web 应用对象的行为进行监控
* 三种监听对象
  1. `ServletContext`:`javax.servlet.ServletContextListener`
  2. `HttpSession`:`javax.servlet.http.HttpSessionListener`
  3. `ServletRequest`:`javax.servlet.ServletRequestListener`
* 属性监听接口
  1. `javax.servlet.ServletContextAttributeListener`
  2. `javax.servlet.http.HttpSessionAttributeListener`
  3. `javax.servlet.ServletRequestAttributeListener`

## 监听器的使用

* 实现 XXXListener 接口，不同接口对应不同的监听对象
* 实现监听器中的独有方法
* 在`web.xml`对监听器进行配置



## 重定向与请求转发

* 请求转发是服务器跳转，只会产生一次请求
* 请求转发语句是：`request.getRequestDispatcher().forward()`
* 重定向是浏览器端的跳转，会产生两次请求 
* `Status Code 302` `Location: /sb`
* 重定向语句是：`response.sendRedirect()`



## Session原理



# Freemarker

## 模板引擎的实现过程

* 数据+模板=结果
* 将数据与展示解耦
* JSP、Freemarker

# HTTP

## HTTP 请求结构

* HTTP 请求包含三个结构：请求行、请求头、请求体
