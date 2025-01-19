---
title: Play2 in Java
date: 2022-04-18 21:18:35
tags: [play2]
categories: [learn]
summary: Play is a high-productivity Java and Scala web application framework that integrates the components and APIs you need for modern web application development.
description: Play is a high-productivity Java and Scala web application framework that integrates the components and APIs you need for modern web application development.
---

# Getting started

*已经成功将项目从 Play 改为 SpringBoot 不再更新*

官方网站：

[play](https://www.playframework.com/)

[sbt](https://www.scala-sbt.org/)

## 创建 Play2 项目

*你可能需要一个梯子🪜*

### 使用 IDEA 创建 Play2 项目

参见：[Getting started with Play 2.x](https://www.jetbrains.com/help/idea/getting-started-with-play-2-x.html)。

启动 Scala 插件：

![image-20220420212825206](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/202825.png)

通过 IDEA 的模板创建项目：

![image-20220418212421717](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/182421.png)

创建完成后 Play2 项目结构如下：

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/182609.png" alt="image-20220418212609419" style="zoom:50%;" />

### 使用 sbt 从命令行创建

参见：[Creating a New Application](https://www.playframework.com/documentation/2.8.x/NewApplication)。

使用如下命令创建 Java Play 项目：

```shell
sbt new playframework/play-java-seed.g8
```

![image-20220420213629111](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/203629.png)

创建完成后的结构如下：

![image-20220420213903281](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/203903.png)

从 IDEA 中启动项目或者在控制台中输入`sbt run` 启动项目后访问 `http:localhost:9000`：

![image-20220418213906167](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/183906.png)

## 项目结构介绍

参见：[Anatomy of a Play application](https://www.playframework.com/documentation/2.8.x/Anatomy)。

```
app                      → Application sources
 └ assets                → Compiled asset sources
    └ stylesheets        → Typically LESS CSS sources
    └ javascripts        → Typically CoffeeScript sources
 └ controllers           → Application controllers
 └ models                → Application business layer
 └ views                 → Templates
build.sbt                → Application build script
conf                     → Configurations files and other non-compiled resources (on classpath)
 └ application.conf      → Main configuration file
 └ routes                → Routes definition
 └ logback.xml           → logback definition
dist                     → Arbitrary files to be included in your projects distribution
public                   → Public assets
 └ stylesheets           → CSS files
 └ javascripts           → Javascript files
 └ images                → Image files
project                  → sbt configuration files
 └ build.properties      → Marker for sbt project
 └ plugins.sbt           → sbt plugins including the declaration for Play itself
lib                      → Unmanaged libraries dependencies
logs                     → Logs folder
 └ application.log       → Default log file
target                   → Generated stuff
 └ resolution-cache      → Info about dependencies
 └ scala-2.13
    └ api                → Generated API docs
    └ classes            → Compiled class files
    └ routes             → Sources generated from routes
    └ twirl              → Sources generated from templates
 └ universal             → Application packaging
 └ web                   → Compiled web assets
test                     → source folder for unit or functional tests
```

### `app/`

`app` 目录包含所有可执行文件：Java 和 Scala 的源代码，模板文件和 `assets`；

默认创建三个包对应 MVC 的三层架构：

`app/controllers`

`app/models`

`app/views`

`app` 下可以随意创建包，包路径也是随意的，比如：`app/store/xianglin/play2/controllers` 或者 `app/store/xiangln/play2/services`。

### `public/`

`public` 存储资源文件，三个子目录分别用于存储 images、CSS 和 JavaScript。

### `conf/`

`conf` 目录存放 Play2 的配置文件，主要有三个配置文件：

`application.conf`：保存应用大部分的配置项，参见：[Configuration file syntax and features](https://www.playframework.com/documentation/2.8.x/ConfigFile)；

`routes`：保存 Play2 的路由规则，即如何将 `uri` 和 `Action` 对应；

`logback.xml`：保存 Logback 日志框架的配置。

### `lib/`

`lib` 是可选的，用于存放需要手动管理的依赖，只需要添加需要的 `JAR` 文件，Play2 会自动把它加入到应用的 `classpath`，参见：[Unmanaged dependencies](https://www.scala-sbt.org/1.x/docs/Library-Dependencies.html)。

### `build.sbt`

类似于 Maven 项目中的 `pom.xml` ，定义项目构建的细节，如：项目名称、版本、依赖等。

### `project/`

保存如下两个文件，用于定义 sbt 的构建规则：

`plugins.sbt`：定义项目使用的 sbt 插件；

`build.properties`：一般用于定义项目使用的 sbt 版本。

### `target/`

保存项目构建产生的文件。

## 开发 `Hello World` 实例程序

参见：[ Implementing Hello World](https://www.playframework.com/documentation/2.8.x/ImplementingHelloWorld)。

使用 Play 创建 `Hello World` 请求很简单，主要有如下几步：

1. 创建 `Hello World` 页面；
2. 创建一个 `Action` 方法；
3. 定义请求映射规则；
4. 测试请求。

### 创建 `Hello World` 页面

在 `app/views` 目录下创建 `hello.scala.html` ，添加如下内容：

```html
@(name: String)
@main("Hello") {
    <section id="top">
        <div class="wrapper">
            <h1>Hello @name</h1>
        </div>
    </section>
}
```

### 创建 `Action` 方法

在 `app/controllers/HomeController` 中添加方法处理请求：

```java
public Result hello(String name) {
    return ok(views.html.hello.render(name));
}
```

### 定义映射规则

在 `conf/routes` 中添加请求 `/hello` 和 `Action` 的映射规则：

```scala
GET        /hello               controllers.HomeController.hello(name:String)
```

### 访问请求

在浏览器输入 `http://localhost:9000/hello?name=MyName` 访问 Hello 页面，如图：

![image-20220421213501584](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/213501.png)

## Play 应用的处理流程

参见：[Play Application Overview](https://www.playframework.com/documentation/2.8.x/PlayApplicationOverview)。

Play 处理 `http://localhost:9000/` 请求的主要步骤如下：

1. 浏览器使用 `GET` 请求访问根路径 `/` ；
2. Play 内部的 HTTP Server 收到请求；
3. Play 使用 `routes` 文件解析请求，将请求映射到对应的 `Action` 方法；
4. `Action` 方法渲染 `index` 页面；
5. HTTP Server 返回 HTML 格式的响应内容。

如下图：

![img](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202204/214438.png)

## Play 示例项目

[Play Tutorials](https://www.playframework.com/documentation/2.8.x/Tutorials) 有很多 Play 的实例项目，供学习使用。

#  Main concepts for Java

参见：[Main concepts for Java](https://www.playframework.com/documentation/2.8.x/JavaHome)。

## 使用 Play `Configuration`

使用依赖注入的方式在项目中使用 Play 的配置对象 `Config`：

```java
package controllers;

import com.typesafe.config.Config;
import play.mvc.Controller;

import javax.inject.Inject;

public class ConfigController extends Controller {
    private final Config config;

    @Inject
    public ConfigController(Config config) {
        this.config = config;
    }
    
    public Result config() {
        return ok(Json.toJson(config));
    }
}
```

## 处理 HTTP 请求

### Play 基础概念：`Action`、`Controller` 和 `Result`

#### `Action`

Play 接收的大部分方法都交由 `Action` 处理，`Action` 用于处理请求并返回结果，如下所示：

```java
public play.mvc.Result index(play.mvc.Http.Request request) {
    return play.mvc.Results.ok("Got request " + request + "!");
}
```

#### `Controller`

Play 中使用的 `Controller` 继承自 `play.mvc.Controllr` ，用于定义一组 `Action` ，如下所示：

```java
package controllers;

import play.*;
import play.mvc.*;

public class Application extends Controller {

  public Result index() {
    return ok("It works!");
  }
  
  public Result hello() {
    return ok("Hello world!");
  }
}
```

#### `Result`

HTTP Response 包括：响应行、响应头和返回数据，Play 中 `play.mvc.Result` 定义了 HTTP 响应结构，`play.mvc.Results` 定义了许多静态方法，方便返回各种 `Result`，如下所示：

```java
// 200 OK
Result ok = ok("Hello world!");
// 404 Not Found
Result notFound = notFound();
Result pageNotFound = notFound("<h1>Page not found</h1>").as("text/html");
// 400 Bad Request
Result badRequest = badRequest(views.html.form.render(formWithErrors));
// 500 Internal Server Error
Result oops = internalServerError("Oops");
// play.mvc.Http.Status
Result anyStatus = status(488, "Strange response type");
// 3XX Redirection
Result redirect = redirect("/user/home");
Result temporaryRedirect =  temporaryRedirect("/user/home");
```

### HTTP 映射

`router` 的作用是将每个请求映射到一个 `Action` 方法的调用，HTTP 请求包括两个部分：

* 请求路径（例如：`/clients/1234`、`/photos/list`），包括查询字符串；
* HTTP 请求方法（`GET`、`POST` 等）。

Play2 的映射定义在 `conf/routes` 文件中，Play2 会编译它，因此可以直接在浏览器中看到错误。映射定义为如下格式：

```properties
Protocol    URLPATH  ControllerMapping
```

最简单的映射如下：

```properties
GET        /                    controllers.HomeController.index()
GET        /hello               controllers.HomeController.hello(name:String)
```

存在动态参数的映射如下：

```properties
# match /bookshop/book/123/
GET /bookshop/book/:id/         controllers.Application.getBook(id:String)

# match 
# /bookshop/book/images/book1.jpg
# /bookshop/book/images/books/book2.jpg
# /bookshop/book/images/books/thumbnails/small/image1.jpg
GET  /bookshop/book/images/*  controllers.Application.fetchImage(name:String)

# $variablename<regular expression>
# match /bookshop/book/10/page/12/
# not match /bookshop/book/10/page/a/
GET  /bookshop/book/:id/page/$page<[0-9]+>/
controllers.Application.fetchBookpage(bookid:String, page:Integer)
```

存在固定参数的映射如下：

```properties
GET /bookshop//authors/    controllers.Application.authors(limit: Integer = 10)
```

存在可选参数的映射如下：

```properties
# variable? = default-value
GET   /bookshop//showcomment/    controllers.Application.showComment(userid ?= null)
```

参见官方文档中有更详细的介绍。

### 处理响应

#### 更改默认的 `Content-Type`

Play2 中 HTTP 响应的 `ContentType` 会根据 Body 中内容的 Java 类型自动推导，例如：

```java
// text/plain
Result textResult = ok("Hello World!");

// application/json
Result jsonResult = ok(Json.toJson(object));
```

可以通过如下方法修改 `Content-Type` ：

```java
public play.mvc.Result as(String contentType) {
}
```

例如：

```java
Result htmlResult = ok("<h1>Hello World!</h1>").as("text/html");
Result htmlResult = ok("<h1>Hello World!</h1>").as(play.mvc.Http.MimeTypes.HTML);
```

#### 更改 HTTP 响应头

通过如下方法修改响应头：

```java
public play.mvc.Result withHeader(String name, String value) {
}
// The headers are processed in pairs, so nameValues(0) is the first header's name, and nameValues(1) is the first header's value, nameValues(2) is second header's name, and so on.
public play.mvc.Result withHeaders(String... nameValues) {
}
```

例如：

```java
public Result index() {
  return ok("<h1>Hello World!</h1>")
      .as(MimeTypes.HTML)
      .withHeader(CACHE_CONTROL, "max-age=3600")
      .withHeader(ETAG, "some-etag-calculated-value");
}
```

#### 使用 `Cookies`

通过如下方法设置 `Cookies` 或删除 `Cookies` ：

```java
// set cookies
public play.mvc.Result withCookies(play.mvc.Http.Cookie... newCookies) {
}
// discarding cookies
public play.mvc.Result discardingCookie(String name) {
}
```

例如：

```java
// add a Cookie
public Result index() {
  return ok("<h1>Hello World!</h1>")
      .as(MimeTypes.HTML)
      .withCookies(
          Cookie.builder("theme", "blue")
              .withMaxAge(Duration.ofSeconds(3600))
              .withPath("/some/path")
              .withDomain(".example.com")
              .withSecure(false)
              .withHttpOnly(true)
              .withSameSite(Cookie.SameSite.STRICT)
              .build());
}

// discard a Cookie
public Result index() {
  return ok("<h1>Hello World!</h1>").as(MimeTypes.HTML).discardingCookie("theme");
}
```

### Session And Flash

保存在 Session 中的数据在整个会话中有效；保存在 Flash 中的数据仅会在下次请求中生效。Play 中 Session 和 Flash 都是通过 Cookies 实现的，所以有几条注意事项：

* 数据的大小不能超过 4K；
* 只能存储字符串内容；
* Cookies 中的内容在浏览器中可见，可能会导致敏感信息泄漏。

#### Session 配置

`play.http.session.cookieName`：Cookie 名称，默认为：`PLAY_SESSION`；

`play.http.session.maxAge`：Session 的超时时间，默认没有超时时间，仅在浏览器关闭时无效。

#### 操作 Session

使用如下方式设置、读取和删除 Session：

```java
public class SessionAndFlashController extends Controller {
    public Result addSession(Http.Request request) {
        // 添加 Session
        return redirect("/session/read").addingToSession(request, "connected", "user@gmail.com");
    }

    public Result readSession(Http.Request request) {
        // 读取 Session
        return request
                .session()
                .get("connected")
                .map(user -> ok("Hello " + user))
                .orElseGet(() -> unauthorized("Oops, you are not connected"));
    }

    public Result clearSession() {
        // 清除 Session
        return redirect("/session/read").withNewSession();
    }
}
```

#### 操作 Flash

Flash 类似 Session，但仅在下一次请求有效。使用如下方式设置、读取 Flash：

```java
public Result addFlash() {
    return redirect("/flash/read").flashing("success", "The item has been created");
}

public Result readFlash(Http.Request request) {
    return ok(request.flash().get("success").orElse("Welcome!"));
}
```

### 请求体解析器

#### 什么是请求体解析器

HTTP 请求的格式是：请求行、请求头和请求体。通常来说请求头很小，能够安全的缓存在内存中，Play 使用 `play.mvc.Http.RequestHeader` 表示请求头。请求体可能很大，因此不能直接缓存在内存中，一般称之为流。事实上很多请求体比较小而且能映射为某种模型缓存在内存中，Play 使用 `play.mvc.BodyParser` 来完成请求体从流到模型的映射。Play 是一个异步框架，因此不能使用传统的 `java.io.InputStream` 来读取请求体，Play 使用 [Akka Stream](https://doc.akka.io/docs/akka/2.6/stream/index.html?language=java&_ga=2.144468661.1268954494.1652018498-216591448.1652018498) 完成这一操作。

#### 使用自带的请求体解析器

没有明确指定时 Play 根据 `Content-Type` 来选择合适的请求体解析器，如下面表格所示：

| Content-Type                                         | Type                                                         | Method                  |
| ---------------------------------------------------- | ------------------------------------------------------------ | ----------------------- |
| `text/plain`                                         | `String`                                                     | `asText()`              |
| `application/json`                                   | `com.fasterxml.jackson.databind.JsonNode`                    | `asJson()`              |
| `application/xml`、`text/xml`、`application/XXX+xml` | `org.w3c.Document`                                           | `asXml()`               |
| `application/x-www-form-urlencoded`                  | `Map<String, String[]>`                                      | `asFormUrlEncoded()`    |
| `multipart/form-data`                                | `play.mvc.Http.MultipartFormData`、`play.mvc.Http.MultipartFormData.FilePart` | `asMultipartFormData()` |
| other                                                | `play.mvc.Http.RawBuffer`                                    | `asRaw()`               |

使用方法如下：

```java
public class BodyParserController extends Controller {
    public Result json(Http.Request request) {
        JsonNode jsonNode = request.body().asJson();
        return ok("Got name: " + jsonNode.get("name").asText());
    }
}
```

#### 明确指定请求体解析器

使用 `@play.mvc.BodyParser.Of` 注解明确指定当前请求的请求体解析器，Play 自带的解析器作为 `play.mvc.BodyParser` 的内部类提供，如下所示：

```java
@play.mvc.BodyParser.Of(BodyParser.Json.class)
public Result json(Http.Request request) {
    JsonNode jsonNode = request.body().asJson();
    return ok("Got name: " + jsonNode.get("name").asText());
}
```

#### 请求体长度限制

`play.http.parser.maxMemoryBuffer`：内存缓存限制，默认100KB；

`play.http.parser.maxDiskBuffer`：磁盘缓存限制，默认10MB。

### 组合 Action

使用组合 Action 需要两步：

1、实现一个 Action 用于完成通用功能，比如打印日志等：

```java
public class VerboseAction extends Action.Simple {
    private final Logger logger = LoggerFactory.getLogger(VerboseAction.class);

    @Override
    public CompletionStage<Result> call(Http.Request req) {
        logger.info("Calling action for {}", req);
        return delegate.call(req);
    }
}
```

2、在需要打印日志的 Action 方法上增加 `@With()` 注解：

```java
public class VerboseController extends Controller {
    @With(VerboseAction.class)
    public Result verboseIndex() {
        return ok("It works!");
    }
}
```

测试 `verboseIndex` 请求，控制台中打印出对应的日志：

```bash
2022-05-15 15:58:55 INFO  controllers.actions.VerboseAction  Calling action for GET /composition/index
```

 `@With()` 注解能应用在 `Controller` 中，此时其中所有的 Action 都会生效。

#### 自定义注解

上面 `@With(VerboseAction.class)` 也可以用一个自定义的注解实现，比如 `@VerboseAnnotation` ，这个注解自身需要添加 `@With` 注解，定义自定义注解需要三步：

1、定义一个注解，表示需要的功能：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@With(VerboseAnnotationAction.class)
public @interface VerboseAnnotation {
    boolean value() default true;
}
```

2、在 Action 中实现通用功能，比如打印日志：

`VerboseAnnotationAction` 通过 `configuration` 获取 `@VerboseAnnotation` 中定义的值：

```bash
public class VerboseAnnotationAction extends Action<VerboseAnnotation> {
    private final static Logger logger = LoggerFactory.getLogger(VerboseAction.class);

    @Override
    public CompletionStage<Result> call(Http.Request req) {
        if (configuration.value()) {
            logger.info("Calling action for {}", req);
        }
        return delegate.call(req);
    }
}
```

3、在 Action 方法上增加自定义的注解：

```java
@VerboseAnnotation
public Result verboseHome() {
    return ok("It works too!");
}
```

事实上 `@play.mvc.Security.Authenticated` 和 `@play.cache.Cached` 也是通过此方式实现的。

#### 打印组合 Action 的作用顺序

在 `logback.xml` 配置文件中添加如下配置，就能在日志中看到完整的 Action 组合链：

```xml
<logger name="play.mvc.Action" level="DEBUG" />
```

```bash
2022-05-15 16:29:45 DEBUG play.mvc.Action  ### Start of action order
2022-05-15 16:29:45 DEBUG play.mvc.Action  1. controllers.actions.VerboseAnnotationAction defined on public play.mvc.Result controllers.VerboseController.verboseHome()
2022-05-15 16:29:45 DEBUG play.mvc.Action  2. play.http.DefaultActionCreator$1()
2022-05-15 16:29:45 DEBUG play.mvc.Action  3. play.core.j.JavaAction$$anon$1()
2022-05-15 16:29:45 DEBUG play.mvc.Action  ### End of action order
```

### 拦截 HTTP 请求

Play 提供了两种方式拦截 Action 的调用，第一种是 `ActionCreator` ，通过 `createAction` 方法返回一个 Action，这个 Action 会作为 Action 组合链

中第一个 Action 或者最后一个 Action，取决于配置：`play.http.actionComposition.executeActionCreatorActionFirst`。

第二种方式是实现自定义的 `HttpRequestHandler`。

#### Action creators

默认需要在根包下定义一个名为 `ActionCreator` 且实现 `play.http.ActionCreator` 的类，如下所示：

```java
public class ActionCreator implements play.http.ActionCreator {
  @Override
  public Action createAction(Http.Request request, Method actionMethod) {
    return new Action.Simple() {
      @Override
      public CompletionStage<Result> call(Http.Request req) {
        return delegate.call(req);
      }
    };
  }
}
```

或者使用如下配置：

```properties
play.http.actionCreator = "com.example.MyActionCreator"
```

## 异步 HTTP 编程

### 处理异步结果

#### 创建和使用 `CompletionStage<Result>`

Java8 提供了 `CompletionStage` 表示 `promise`，`CompletionStage<Result>` 最终会得到一个 `Result`。通过 `CompletableFuture.supplyAsync()` 方法可以得到一个 `CompletionStage`：

```java
// creates new task
CompletionStage<Integer> promiseOfInt = CompletableFuture.supplyAsync(() -> intensiveComputation());
```

`supplyAsync` 会创建一个新的任务提交到 `fork/join` 框架中执行。

#### 使用 `HttpExecutionContext`

在 Action 中使用 Java `CompletionStage` 时，必须显式提供 HTTP 执行上下文作为执行期，以确保类加载器保持在作用域内，代码如下所示：

```java
public class AsyncController extends Controller {
    private final HttpExecutionContext httpExecutionContext;

    @Inject
    public AsyncController(HttpExecutionContext httpExecutionContext) {
        this.httpExecutionContext = httpExecutionContext;
    }

    public CompletionStage<Result> index() {
        CompletionStage<Integer> promiseOfInt = CompletableFuture.supplyAsync(this::intensiveComputation, httpExecutionContext.current());
        return promiseOfInt.thenApply(integer -> ok(String.valueOf(integer)));
    }

    private Integer intensiveComputation() {
        int value = Integer.MIN_VALUE;
        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            value++;
        }
        return value;
    }
}

```

* [ ] [以确保类加载器保持在作用域内](https://www.playframework.com/documentation/2.8.x/ThreadPools#Class%20loaders)。

#### 使用 `CustomExecutionContext` and `HttpExecution`

使用 `CompletionStage` 或者 `HttpExecutionContext` 时仍然使用的是 Play 默认的执行上下文，如果需要执行一些阻塞 API 比如 JDBC，最好的方式是实现自己的 `CustomExecutionContext`，将阻塞任务从 Play 的线程池中移出去，通过继承 `play.libs.concurrent.CustomExecutionContext` 实现自定义的 `ExecutionContex`：

```java
public class MyCustomExecutionContext extends CustomExecutionContext {
    @Inject
    public MyCustomExecutionContext(ActorSystem actorSystem) {
        super(actorSystem, "blocking-io-dispatcher");
    }
}
```

`blocking-io-dispatcher` 在 `application.conf` 中定义：

```properties
blocking-io-dispatcher {
  type = Dispatcher
  executor = "thread-pool-executor"
  thread-pool-executor {
    fixed-pool-size = 32
  }
  throughput = 1
}
```

使用方法如下：

```java
public class AsyncWithCustomExecutionContentController extends Controller {
    private final MyCustomExecutionContext myCustomExecutionContext;

    @Inject
    public AsyncWithCustomExecutionContentController(MyCustomExecutionContext myCustomExecutionContext) {
        this.myCustomExecutionContext = myCustomExecutionContext;
    }

    public CompletionStage<Result> index() {
        var executor = HttpExecution.fromThread(myCustomExecutionContext);
        return CompletableFuture.supplyAsync(() -> ok("It's work!"), executor);
    }
}

```

使用 `HttpExecution.fromThread` 方法从 `MyCustomExecutionContext` 中获取已存在的线程池。

#### 处理超时

请求经常需要限定等待时间，避免出错时浏览器无限等待。通过 `play.api.libs.concurrent.Futures.timeout` 包装 `CompletionStage` 实现：

```java
public class TimeoutController extends Controller {
    private final Futures futures;
    private final MyCustomExecutionContext myCustomExecutionContext;

    @Inject
    public TimeoutController(Futures futures, MyCustomExecutionContext myCustomExecutionContext) {
        this.futures = futures;
        this.myCustomExecutionContext = myCustomExecutionContext;
    }

    public CompletionStage<Result> index() {
        return futures.timeout(delayedResult(), Duration.ofSeconds(1));
    }
    
    public CompletionStage<Result> delayedResult() {
        ExecutionContextExecutor executor = HttpExecution.fromThread(myCustomExecutionContext);
        long start = System.currentTimeMillis();
        return futures.delayed(() -> CompletableFuture.supplyAsync(() -> {
            long end = System.currentTimeMillis();
            long seconds = end - start;
            return "rendered after " + seconds + " seconds";
        }, executor), Duration.of(3, SECONDS)).thenApply(Results::ok);
    }
}
```

需要注意的是，超时 `timeout` 和取消 `cancel` 不同，超时后 `delayedResult` 仍会执行，尽管结果不会被返回。

### 流式 HTTP 响应

#### `Content-Length` 响应头

HTTP1.1 支持长连接，一个 TCP 连接可以服务多次 HTTP 请求，此时服务端必须使用 `Content-Length` 响应头。默认情况下，使用 Play 不需要明确指定 HTTP 响应头，因为返回的内容是明确的，Play 可以自行计算并添加对应的响应头。HTTP 响应由响应头和响应体组成，使用 `play.http.HttpEntity` 表示响应体，Play 中一个 HTTP 响应的表示如下：

```java
public Result httpResponse() {
    return new Result(
        new ResponseHeader(Http.Status.OK, Collections.emptyMap()),
        new HttpEntity.Strict(ByteString.fromString("It's work!"), Optional.of(Http.MimeTypes.TEXT))
    );
}
```

#### 发送大量数据

使用 Play 返回一个文件给客户端，构建一个 `Source[ByteString, _]` 作为响应内容，然后使用 `play.http.HttpEntity` 返回，如下所示：

```java
public Result index() {
    var path = Paths.get("/home/xianglin/Downloads/20220504100611.json");
    var source = FileIO.fromPath(path);
    var contentLength = Optional.<Long>empty();

    return new Result(
        new ResponseHeader(Http.Status.OK, Collections.emptyMap()),
        new HttpEntity.Streamed(source, contentLength, Optional.of(Http.MimeTypes.TEXT))
    );
}
```

上面的响应没有明确指定 `contentLength`，Play 就会将文件加载到内存然后计算长度。对于大文件，需要明确指定 `contentLength` 避免这种情况发送，如下：

```java
public Result index() throws IOException {
    var path = Paths.get("/home/xianglin/Downloads/20220504100611.json");
    var source = FileIO.fromPath(path);

    // compute the response size
    var contentLength = Optional.of(Files.size(path));

    return new Result(
        new ResponseHeader(Http.Status.OK, Collections.emptyMap()),
        new HttpEntity.Streamed(source, contentLength, Optional.of(Http.MimeTypes.TEXT))
    );
}
```

#### 处理文件

Play 可以使用 `play.mvc.Results#ok(java.io.File)` 方法直接返回文件，此方法会添加 `Content-Type` 和 `Content-Disposition` 响应头。默认情况下是 `Content-Disposition: inline; filename=`，如果不想浏览器下载文件而是直接打开文件，可以使用 `play.mvc.Results#ok(java.io.File, boolean)`：

```java
public Result file() {
    return ok(new File("/home/xianglin/Downloads/20220504100611.json"), Optional.of("示例文件.json")).as(Http.MimeTypes.BINARY);
}

public Result fileToDisplay() {
    return ok(new File("/home/xianglin/Downloads/20220504100611.json"), false);
}
```

#### 分片响应

分块传输编码允许 HTTP 服务器发送给客户端的数据分成多个部分。Play 中可以使用 `play.mvc.Results#ok(java.io.InputStream)`  和 `ok().chunked()` 方法返回分块数据：

```java
public Result fileStream() throws FileNotFoundException {
    return ok(new FileInputStream("/home/xianglin/Downloads/20220504100611.json"));
}

public Result trunked() {
    Source<ByteString, ?> trunked = Source.<ByteString>actorRef(256, OverflowStrategy.dropNew())
        .mapMaterializedValue(
        sourceActor -> {
            sourceActor.tell(ByteString.fromString(""), null);
            sourceActor.tell(ByteString.fromString(""), null);
            sourceActor.tell(ByteString.fromString(""), null);
            sourceActor.tell(ByteString.fromString(""), null);
            return NotUsed.getInstance();
        }
    );
    return ok().chunked(trunked);
}
```

## Play modules

### Play with Mongo

[PlayMorphia](https://github.com/morellik/play-morphia) 是 [Morphia](https://github.com/MorphiaOrg/morphia) 针对 Play 的拓展，方便使用 Morphia 进行 Play MongoDB 开发。

#### 安装方法

添加 Morphia 的依赖：

```scala
libraryDependencies ++= Seq(
    guice,
    "dev.morphia.morphia" % "core" % "1.6.1"
)
```

从 PlayMorphia 项目中下载 [play_morphia_jar](https://github.com/morellik/play-morphia/tree/master/out/artifacts/play_morphia_jar) 放在项目的 `lib` 目录下；

在 `conf/application.conf` 中配置 Mongo 的连接信息，最终效果如下图所示（如果使用 IDEA,可以右键点击 `lib` 选择“添加为库”）：

![image-20220515214011364](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/214011.png)

#### 使用方法

简单按照 `controllers`、`models` 和 `repositories` 的结构创建三个包，以操作 `User` 为例。

`User` Model：

```java
package models;

import dev.morphia.annotations.Entity;
import dev.morphia.annotations.Id;
import org.bson.types.ObjectId;

/**
 * @author xianglin
 */
@Entity("DB.users")
public class User {
    @Id
    private ObjectId id;
    private String firstname;
    private String lastname;
    private String email;

    // getter and setter
}
```

`UserRepository`：

```java
package repositories;

import dev.morphia.Key;
import it.unifi.cerm.playmorphia.PlayMorphia;
import models.User;
import org.bson.types.ObjectId;

import javax.inject.Inject;

/**
 * @author xianglin
 */
public class UserRepository {
    private final PlayMorphia playMorphia;

    @Inject
    public UserRepository(PlayMorphia playMorphia) {
        this.playMorphia = playMorphia;
    }

    public User findById(String id) {
        return playMorphia
                .datastore()
                .createQuery(User.class)
                .field("id")
                .equal(new ObjectId(id))
                .first();
    }

    public Key<User> save(User user) {
        return playMorphia
                .datastore()
                .save(user);
    }
}
```

`MongoController`：

```java
package controllers;

import models.User;
import play.libs.Json;
import play.mvc.Controller;
import play.mvc.Result;
import repositories.UserRepository;

import javax.inject.Inject;

/**
 * 测试 MongoDB 操作
 *
 * @author xianglin
 */
public class MongoController extends Controller {
    private final UserRepository userRepository;

    @Inject
    public MongoController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public Result getUser(String id) {
        return ok(Json.toJson(userRepository.findById(id)));
    }

    public Result saveUser() {
        User user = new User();
        user.setFirstname("Xiang");
        user.setLastname("Lin");
        user.setEmail("mail@mail.com");
        return ok(Json.toJson(userRepository.save(user)));
    }
}

```

`routes`：

```properties
GET         /mongo/user/get            controllers.MongoController.getUser(id: String)
POST        /mongo/user/save           controllers.MongoController.saveUser()
```

最后使用 postman 测试接口，结果如下：

![image-20220515215246529](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/215246.png)

![image-20220515215331674](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/215331.png)

#### 使用新版 Morphia

Morphia 2.0 以上版本需要 Java11，已经更高版本的 mongo-drive，请使用 [play-morphia](https://github.com/xianglin2020/play-morphia) 中修改的 [play-morphia.jar](https://github.com/xianglin2020/play-morphia/blob/master/out/artifacts/play_morphia_jar/play-morphia.jar)。
