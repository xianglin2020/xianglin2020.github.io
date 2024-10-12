---
title: "Java各版本语言特性"
date: 2024-03-02T10:29:51+08:00
categories:
  - learn
tags:
  - java
summary: "Java8到Java21语言相关的部分特性。"
description: "Java8到Java21语言相关的部分特性。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## Java 8

https://openjdk.org/projects/jdk8/features

### Lambda & Functional Interface

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}

public class LambdaExamples {
    @Test
    public void test(){
        new Thread(() -> {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

内部优化通过`invokedynamic`指令直接调用了匿名函数。

### 方法引用操作符`::`

```java
public class MethodReference {
    @Test
    public void test(){
        var val = List.of(1,2,3,4,5).stream()
                .map(Object::toString)
                .map(Integer::new)
                .reduce((a,b)->a+b);
                
        System.out.println(val);

    }
}
```

### Monad

* Stream：函数式、流计算
* Optional：流计算，减少空值检查
* Try Monad：https://github.com/jasongoodwin/better-java-monads

### 接口的方法(static, default, private)

```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {

   // 每个Stream Monad都有这个方法
   default Stream<T> takeWhile(Predicate<? super T> predicate) {
   ...
       foo(); // 可以调用
   }
   
   // 可以通过Stream.empty()构造空的流
   public static<T> Stream<T> empty() {
    ...     
   }
    
   // 不是源代码中的程序（Java9）
   private foo(){}
    
}
```

### 本地化日期处理升级

提供了Local和Zoned两种处理日期的方式，在新的API中不再允许mutable的操作（非线程安全）；另外对时区采取更好的处理方法：如果用户的系统不考虑国际化，那么就用Local的日期，如果考虑国际化，可以构造带时区的DateTime。

```java
@Test
public void test_date(){
    // 2022年2月22日
    var date = LocalDate.of(2022, 2, 22);

    // 10:55:59
    var time = LocalTime.of(10, 55, 59);

    // 当前时间
    var datetime = LocalDateTime.of(date, time);

    var zoneDT = ZonedDateTime.of(datetime, ZoneId.of("Asia/Shanghai"));
}
```

### 内置了Base64工具

> 对0~255之间的字符，Base64编码是以4个可见字符去描述3个字符。会增加数据的体积，但是因为所有字符都可以读，用Base64编码描述的字符串，在URL、XML中都不会被转义。

`Base64.Decoder`和`Base64.Encoder`类。

## Java 9

https://openjdk.org/projects/jdk9/

### 模块系统

https://openjdk.org/jeps/200

模块系统应用于平台本身和 JDK。其主要目标是使平台的实现更容易扩展到小型设备，提高安全性和可维护性，改善应用程序性能，并为开发人员提供更好的大型编程工具。

### 交互式编程环境(JShell)

![image-20240302152536091](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202403031130611.png)

### 新的HTTP 2.0 Clinet

主要针对旧Client，提供Buidler，方便用户设置header和参数。

另外支持HTTP2.0，HTTP2.0兼容HTTP1.1的能力，主要是从性能角度进行了调优。理解HTTP2.0主要是这几个特性：

- 多个HTTP 请求/返回在一个TCP请求上多路复用（客户端要负责实现多路复用）
- 头部压缩算法（客户端要负责解压）
- Server Push：服务器可能会给客户端额外的文件（浏览器要负责识别，并缓存这些文件，客户端不一定要实现）

```java
HttpClient httpClient = HttpClient.newHttpClient(); 

HttpRequest httpRequest 
    = HttpRequest 
    .newBuilder() 
    .headers("x-key", "123456", "x-name", "something")
    .uri(new URI("https://www.baidu.com")) 
    .GET() 
    .build(); 

HttpResponse<String> response 
    = httpClient.send( 
    httpRequest, 
    HttpResponse.BodyHandler.asString()); 

System.out.println(response.statusCode());
System.out.println(response.body());
```

### 改进了Javadoc

可以生成符合HTML5标准的网页。

```bash
javadoc -d foo -html5 MyHelloWorld.java
```

### 支持multirelease jar包

项目目录结构可以这样安排，不同的版本的程序在不同的目录下。

```bash
|-java
|   └──java8
|       └── hello
|           └── xxx.java
|   └──java9
|       └── hello
|           └── xxx.java
```

然后可以通过`javac` 编译成不同版本的class文件：

```bash
javac --release 9 java8/hello/xxx.java
javac --release 8 java9/hello/xxx.java
```

最后用`jar` 可以产生不同版本的`jar` 包：

```bash
jar -c -f xxx.jar -C java8 . --release 9 -C java9 .
```

这样`xxx.jar` 中会同时有java8和java9的程序。

```bash
java -cp test.jar hello // 不同java版本访问不同的类
```

这个功能的意义是Java升级太快了，多版本同时发布，让程序的提供者可以兼容不同用户的需求。

### 新增集合的工厂方法

```java
var set = Set.of("Apple", "Google");
var list = List.of("...", "abc", "1");
var map = Map.of("String", 5, "Integer", 6); 
```

### Stream API增加了方法

* takeWhile

  字面意思：take while x != 7 is hold ， 中文意思：在x != 7的时候，拿，否则停下来

  ```java
  Stream.of(2,1,3,7,4,6,8,0).takeWhile(x -> x != 7)
     .collect(Collectors.toList());
  // [2,1,3]
  ```

* dropWhile

  字面意思： drop while x!=7 is hold， 中文意思：在x!=7的时候，丢弃，否则停下来

  ```java
  Stream.of(2,1,3,7,4,6,8,0).dropWhile(x -> x!=7)
    .collect(Collectors.toList());
  //[7, 4, 6, 8, 0]
  ```

* iterate

  ```java
   Stream.iterate(
       10,  // 初始值
       x -> x < 100, // 终结断言
       x->x+2 // 递推函数
   ).collect(Collectors.toList());
  
  // [10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 34, 36, 38, 40, 42, 44, 46, 48, 50, 52, 54, 56, 58, 60, 62, 64, 66, 68, 70, 72, 74, 76, 78, 80, 82, 84, 86, 88, 90, 92, 94, 96, 98]
  
  ```

* ofNullable

  ```java
  Stream.ofNullable(null); // Stream.empty()
  ```

### try-with-resources的改进

Java中的一部分资源对象会继承于Closeable接口，这就可以使用`try-with-resources` 能力。例如：

```java
public abstract class InputStream implements Closeable {
    ...
}
```

如果要使用的话：

```java
@Test
public void test() throws FileNotFoundException, IOException {
    var fin = new FileInputStream("somefile");
    try(fin) { // AutoClose
        fin.read();
    }
}
```

## Java 10

https://openjdk.org/projects/jdk/10/

### 局部变量类型推断

```java
var list = new LinkedList<Integer>();
// list is LinkedList<Integer>

var o = Stream.of(1,2,3);
// o is Stream<Integer>
```

### JDK代码仓库整理

所有JDK代码不再使用8个仓库存储：root、corba、hotspot、jaxp、jaxws、jdk、langtools、nashorn。

统一成为1个仓库。主要是解决原子提交的问题：1个功能可能需要更新多个代码仓库。

### 应用程序数据共享（Application Data Sharing)

允许多个JVM实例共享共同用到的类，这些类以共享内存的形式存在。这样对于运行了多个JVM的机器，可以节省内存空间以及类的加载速度。

### 计划移除JNI头生成工具

JNI（Java Native Interface)是Java程序和Native(C, C++)程序沟通的接口。 一个Java类，如果要Native调用，通常是在Android开发当中，需要生成一个C/C++的头文件。之前可以用javah来生成，现在用javah工具生成的时候，会有一行warning。

以后JNI的能力会被Panama项目替代，一个专门为非java语言提供接口的库。

### 增加实验性的Graal编译器

可以配置参数开启：

```bash
-XX：+ UnlockExperimentalVMOptions -XX：+ UseJVMCICompiler
```

Graal是一款同时支持JIT和AOT的编译器。

* JIT(Just in Time)一边编译一边执行，执行完第一次之后，下一次不需要编译。
* AOT(Ahead of Time)，类似C/C++那样，先编译成机器码，再执行。 注意，是机器码，越过了JVM的bytecode。

## Java 11

https://openjdk.org/projects/jdk/11/

### 引入NestedMembers概念

嵌套类和它的父亲作为一组NestedMembers，可以互相访问元数据。

```java
public class Nested {

    class A{
        private String name = "123";
    }

    class B{
        public void bar() throws NoSuchFieldException {
            System.out.println(A.class.getDeclaredField("name"));
        }
    }

    @Test
    public void test() throws NoSuchFieldException {
        var b = new B();
        b.bar();
        // name 

        System.out.println(Arrays.toString(A.class.getNestMembers()));
        // [Nested, A, B]
        
        System.out.println(A.class.getNestHost());
        // Nested
        
        System.out.println(Arrays.toString(B.class.getNestMembers()));
        // [Nested, A, B]
        
        System.out.println(B.class.getNestHost());
        // Nested
    }
}
```

### 增加无操作GC回收器:Epsilon

Epsilon不会进行垃圾回收操作，但是仍然承担着内存分配的工作。

优点：

* 对于开发者明确知道不需要GC的程序有助于减少延迟
* 对于性能测试、压力测试场景，可以忽略GC带来的延迟

### 增加字符串处理函数

```java
" ".isBlank();                // true
" Foo Bar ".strip();          // "Foo Bar"
" Foo Bar ".stripTrailing();  // " Foo Bar"
" Foo Bar ".stripLeading();   // "Foo Bar "
"Java".repeat(3);             // "JavaJavaJava"
"A\nB\nC".lines().count();    // 3
```

## Java 12

### `Shenandoah` 低暂停时间垃圾回收器（实验性）

该算法通过与运行中的 Java 线程同时进行疏散工作来减少 GC 暂停时间。使用 Shenandoah 算法的暂停时间与堆大小无关，这意味着无论堆大小是 200 MB 还是 200 GB，暂停时间都是一致的。

可以配置参数开启：

```bash
-XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC
```

### Switch 表达式（预览）

扩展switch语句，使其可以作为语句也可以作为表达式使用，将简化日常编码，并为在switch中使用[模式匹配](https://openjdk.org/jeps/305)做好准备。

### G1 收集器增强

* 保证G1收集器不超过预定的暂停时间
* 使其在空闲时自动将 Java 堆内存归还给操作系统

## java 13

https://openjdk.org/projects/jdk/13/

### 增强ZGC

增强 ZGC，将未使用的堆内存归还给操作系统。

### 重新实现SocketAPI

用一个更简单、更现代、更易于维护和调试的实现来取代 java.net.Socket 和 java.net.ServerSocket API 所使用的底层实现。

### 文本块（预览）

为 Java 语言添加文本块。文本块是一种多行字符串字面形式，可避免使用大多数转义序列，以可预测的方式自动格式化字符串，并在需要时让开发人员控制格式。

## Java 14

### `instanceof` 的模式匹配（预览）

通过实例操作符的模式匹配增强 Java 编程语言。模式匹配允许更简洁、更安全地表达程序中的常见逻辑，即从对象中有条件地提取组件。

### 优化空指针异常信息

通过精确描述哪个变量为空，提高 JVM 生成的 NullPointerException 的可用性。

以前的`NullPointerException`异常信息在如下场景下很难定位到问题，例如

```java
a.b.c.i = 99;
a[i][j][k] = 99;
a.i = b.j;
x().y().i = 99;

// message
Exception in thread "main" java.lang.NullPointerException
    at Prog.main(Prog.java:5)
```

改进后的异常信息如下

```bash
Exception in thread "main" java.lang.NullPointerException: 
        Cannot assign field "i" because "a" is null
    at Prog.main(Prog.java:5)
```

### Records（预览）

通过 records 增强Java语言，为声明不可变类提供了一种简洁的语法。

### Switch表达式

原来的switch语句

```java
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        System.out.println(6);
        break;
    case TUESDAY:
        System.out.println(7);
        break;
    case THURSDAY:
    case SATURDAY:
        System.out.println(8);
        break;
    case WEDNESDAY:
        System.out.println(9);
        break;
}
```

使用 `case L -> `形式的switch语句

```java
switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> System.out.println(6);
    case TUESDAY                -> System.out.println(7);
    case THURSDAY, SATURDAY     -> System.out.println(8);
    case WEDNESDAY              -> System.out.println(9);
}
```

扩展switch以支持表达式

```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
};
```

如果switch表达式在标签后有语句块，可以通过yield返回一个值

```java
int j = switch (day) {
    case MONDAY  -> 0;
    case TUESDAY -> 1;
    default      -> {
        int k = day.toString().length();
        int result = f(k);
        yield result;
    }
};
```

### 垃圾收集器改动

* 移除CMS收集器
* 将ZGC移植到MacOS和Windows上
* 弃用ParallelScavenge + SerialOld的GC组合

## Java 15

https://openjdk.org/projects/jdk/15/

### 密封类（预览）

封闭的类和接口限制了其他类或接口对它们的扩展或实现。

### 重新实现DatagramsocketAPI

将 java.net.DatagramSocket 和 java.net.MulticastSocket API 的底层实现替换为易于维护和调试的更简单、更现代的实现。

### 弃用和禁用偏向锁

默认情况下禁用偏向锁，并废弃所有相关的命令行选项。原因如下：

* 新应用多使用非同步容器，应用程序不会从偏向锁中获得明显的性能提升。
* 偏向锁在同步子系统中引入了大量复杂的代码，同时也侵入了其他 HotSpot 组件。这种复杂性阻碍了对代码各部分的理解，也妨碍了对同步子系统进行重大设计变更。

### 文本块

使用文本块的示例

```java
String html = """
              <html>
                  <body>
                      <p>Hello, world</p>
                  </body>
              </html>
              """;

String query = """
               SELECT "EMP_ID", "LAST_NAME" FROM "EMPLOYEE_TB"
               WHERE "CITY" = 'INDIANAPOLIS'
               ORDER BY "EMP_ID", "LAST_NAME";
               """;
```

### 垃圾收集器改动

* 可扩展的低延迟垃圾回收器 ZGC 成为生产特性
* 低暂停时间垃圾回收器 Shenandoah 成为生产特性

## Java 16

https://openjdk.org/projects/jdk/16/

### `instanceof` 的模式匹配

原来使用instanceof判断类型

```java
if (obj instanceof String) {
    String s = (String) obj;    // grr...
    ...
}
```

使用instanceof模式匹配后

```java
if (obj instanceof String s) {
    // Let pattern matching do the work!
    ...
}
```

### Records

定义一个表示Point的类

```java
class Point {
    private final int x;
    private final int y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    int x() { return x; }
    int y() { return y; }

    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point other = (Point) o;
        return other.x == x && other.y == y;
    }

    public int hashCode() {
        return Objects.hash(x, y);
    }

    public String toString() {
        return String.format("Point[x=%d, y=%d]", x, y);
    }
}
```

使用Record后

```java
record Point(int x, int y) { }
```

## Java 17

https://openjdk.org/projects/jdk/17/

### switch表达式模式匹配（预览）

### 密封类

通过 sealed 密封类或接口，通过 permits 指定可以扩展此密封类或接口的类。

```java

public abstract sealed class Shape
    permits Circle, Rectangle, Square { ... }


abstract sealed class Root { ... 
    final class A extends Root { ... }
    final class B extends Root { ... }
    final class C extends Root { ... }
}
```

## Java 18

https://openjdk.org/projects/jdk/18/

### 指定 UTF-8 为默认编码

指定 UTF-8 为标准 Java API 的默认字符集。有了这一更改，依赖于默认字符集的 API 将在所有实现、操作系统、本地化和配置中表现一致。

### 简单的Web服务器

提供一个命令行工具，用于启动一个仅提供静态文件的最小网络服务器。

比如我在 Java21 目录下执行

```bash
jwebserver
```

![image-20240302181727987](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202403031130696.png)

### Java API 文档中的代码片段

为 JavaDoc 的 Standard Doclet 引入 `@snippet` 标签，以简化在 API 文档中包含示例源代码的工作。

```java
/**
 * The following code shows how to use {@code Optional.isPresent}:
 * {@snippet :
 * if (v.isPresent()) {
 *     System.out.println("v: " + v.get());
 * }
 * }
 */


// 引入外部片段
/**
 * The following code shows how to use {@code Optional.isPresent}:
 * {@snippet file="ShowOptional.java" region="example"}
 */
```

## Java 19

https://openjdk.org/projects/jdk/19/

### 虚拟线程（预览）

将虚拟线程引入 Java 平台。虚拟线程是轻量级线程，可显著减少编写、维护和观察高吞吐量并发应用程序的工作量。

## java 20

https://openjdk.org/projects/jdk/20/

无正式特性

## Java 21

https://openjdk.org/projects/jdk/21/

### 字符串模板（预览）

字符串模板是对 Java 现有字符串字面量和文本块的补充，它将字面文本与表达式和模板处理器结合起来，从而产生结果。

### 序列容器

新增 `SequencedCollection` 、`SequencedSet` 和 `SequencedMap`，引入 `reversed` 、`getFirst`、`getLast`等方法。

![img](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/2024/202403031130019.png)

### Record Patterns

通过 record patterns 解构 record 的值。

```java
// As of Java 16
record Point(int x, int y) {}

static void printSum(Object obj) {
    if (obj instanceof Point p) {
        int x = p.x();
        int y = p.y();
        System.out.println(x+y);
    }
}

// As of Java 21
static void printSum(Object obj) {
    if (obj instanceof Point(int x, int y)) {
        System.out.println(x+y);
    }
}
```

### switch 的模式匹配

```java
// Prior to Java 21
static String formatter(Object obj) {
    String formatted = "unknown";
    if (obj instanceof Integer i) {
        formatted = String.format("int %d", i);
    } else if (obj instanceof Long l) {
        formatted = String.format("long %d", l);
    } else if (obj instanceof Double d) {
        formatted = String.format("double %f", d);
    } else if (obj instanceof String s) {
        formatted = String.format("String %s", s);
    }
    return formatted;
}

// As of Java 21
static String formatterPatternSwitch(Object obj) {
    return switch (obj) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> obj.toString();
    };
}
```

### 虚拟线程

https://openjdk.org/jeps/444

```java
public static void main(String[] args) throws IOException {
    var s = System.currentTimeMillis();

    try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
        IntStream.range(0, 100_000).forEach(i -> {
            executor.submit(() -> {
                Thread.sleep(Duration.ofSeconds(1));
                return i;
            });
        });
    }

    System.out.printf("time usage in virtual threads: %d ms", (System.currentTimeMillis() - s));
}
```

其它使用虚拟线程的方法

```java
// 创建一个名为duke的虚拟线程，未启动
var duke = Thread.ofVirtual().name("duke").unstarted(() -> {
});
// 创建并启动一个虚拟线程
var thread = Thread.startVirtualThread(() -> {
});
```

