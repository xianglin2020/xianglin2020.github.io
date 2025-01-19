---
title: "Kotlin基础"
date: 2023-08-27T10:44:46+08:00
categories:
  - learn
tags:
  - kotlin
summary: "Kotlin的基础知识学习和与Java的异同点比较。"
description: "Kotlin的基础知识学习和与Java的异同点比较。"
author: ["XiangLin"]
cover:
  image: "https://kotlinlang.org/assets/images/open-graph/general.png"
  caption: ""
  alt: ""
  relative: false
---

# Kotlin基础

Kotlin官网：[https://kotlinlang.org/](https://kotlinlang.org/)

Kotlin官方文档中文版：[https://book.kotlincn.net/](https://book.kotlincn.net/)

## 内置类型

### 变量类型

| 类型   | Kotlin             | Java                                  |
| ------ | ------------------ | ------------------------------------- |
| 字节   | Byte               | byte/Byte                             |
| 整型   | Short & Int & Long | short/Short & int/Integre & long/Long |
| 浮点型 | Float & Double     | float/Float & double/Double           |
| 字符   | Char               | char/Character                        |
| 字符串 | String             | String                                |

### 声明变量

```kotlin
val b:String = "Hello Kotlin"

// 自动类型推导
var a = 2

val l = 123L
```

* val：只读变量
* var：可读写变量

### 数值类型转换

```java
int i = 10;
long l = i;
```

```kotlin
val i:Int = 10
val l:Long = i.toLong()
```

Kotlin 中的数字没有隐式拓宽转换，如需将数值转换为不同的类型，请使用显式转换。所有数字类型都支持转换为其他类型：

- `toByte(): Byte`
- `toShort(): Short`
- `toInt(): Int`
- `toLong(): Long`
- `toFloat(): Float`
- `toDouble(): Double`

### 无符号类型

除了整数类型，对于无符号整数，Kotlin 还提供了以下类型：

- `UByte`: 无符号 8 比特整数，范围是 0 到 255
- `UShort`: 无符号 16 比特整数，范围是 0 到 65535
- `UInt`: 无符号 32 比特整数，范围是 0 到 2^32 - 1
- `ULong`: 无符号 64 比特整数，范围是 0 到 2^64 - 1

### 字符串

字符串比较

* `a == b`：比较内容，等价于Java的`equals()`
* `a === b`：比较对象是否是同一个对象

字符串模板

```kotlin
val j = "I❤️China"
println("Value of String 'j' is: $j") // no need brackets
println("Length of String 'j' is: ${j.length}") // need brackets
```

### 数组

|            | Kotlin         | Java        |
| ---------- | -------------- | ----------- |
| 整型       | IntArray       | int[]       |
| 包装类整型 | Array\<Int>    | Integer[]   |
| 字符       | CharArray      | char[]      |
| 包装类字符 | Array\<Char>   | Character[] |
| 字符串     | Array\<String> | String[]    |

### 区间

```kotlin
val intRange = 1..10 // [1, 10]
val charRange = 'a'..'z'
val longRange = 1L..100L

val intRangeExclusive = 1 until 10 step 2 // [1, 10)
val intRangeReverse = 10 downTo 1 // [10, 9, ... , 1]
```

## 集合框架

* 增加了“不可变”集合框架的接口
* 通过类型别名实现复用JavaAPI实现集合：`public actual typealias ArrayList<E> = java.util.ArrayList<E>`
* 提供了丰富易用的方法，例如：`forEach/map/flatMap`
* 提供了运算符支持，简化集合框架的访问

|      | Kotlin                     | Java     |
| ---- | -------------------------- | -------- |
| 列表 | List\<T> / MutableList\<T> | List\<T> |
| 映射 | Map<K,V> / MutableMap<K,V> | Map<K,V> |
| 集合 | Set\<T> / MutableSet\<T>   | Set\<T>  |

```java
List<Integer> intList = new ArrayList<>(Arrays.asList(1, 2, 3, 4));

HashMap<String, Integer> hashMap = new HashMap<>();
hashMap.put("Hello", 10);
```

```kotlin
val intList: List<Int> = listOf(1, 2, 3, 4)
val intList2: MutableList<Int> = mutableListOf(1, 2, 3, 4)
val map: Map<String, Any> = mapOf("name" to "benny", "age" to 20)
val map2: Map<String, Any> = mutableMapOf("name" to "benny", "age" to 20)
```

## 函数

* 函数有自己的类型，是Kotlin中的”一等公民“
* 可以赋值、传递，并在合适的条件下调用
* 方法可以认为是函数的一种特殊类型，从形式上看，有receiver的函数即为方法。

### 基本概念

函数的定义

```kotlin
fun main(args: Array<String>):Unit{
    
}
```

函数的类型

```kotlin
fun foo(){}				() -> Unit
fun foo(p0: Int): String {}		(Int) -> String
class Foo{
    fun bar(p0: String, p1: Long):Any{}	Foo.(String, Long) -> Any
}
```

函数的引用

```kotlin
fun foo(){}			::foo		val f: () -> Unit = ::foo
fun foo(p0: Int): String{} 	::foo		val g:(Int) -> String = ::foo
class Foo{
    fun bar(p0: String, p1: Long):Any{}	Foo::bar	val h:(Foo, String, Long) -> Any = Foo:bar
}
```

变长参数

```kotlin
fun multiParameters(vararg ints: Int) {
    println(ints.contentToString())
}
```

多返回值

```kotlin
fun multiReturnValues(): Triple<Int, Long, Double> {
    return Triple(1, 3L, 4.0)
}

val (a, b, c) = multiReturnValues()
```

默认参数和具名参数

```kotlin
fun defaultParameter(x: Int = 0, y: String, z: Long = 1L){
    
}

defaultParameter(y = "Hello")
```

### 匿名函数

```kotlin
// 匿名函数
fun() {
    
}

// 匿名函数的传递
val func = fun() {
    
}

// 匿名函数的类型
val func: () -> Unit = fun() {
    
}
```

### Lambda表达式

```kotlin
val lambda = {
    
}

val lambda: () -> Unit = {
    
}
```

### 高阶函数

参数类型包含函数类型或返回值类型为函数类型的函数称为高阶函数。

```kotlin
// 参数类型包含函数类型
fun cost(block: ()->Unit){
    val start = System.currentTimeMillis()
    block()
    println("${System.currentTimeMillis() - start}ms")
}

// 返回值类型为函数类型
fun fibonacci(): () -> Long {
    var first = 0L
    var second = 1L
    return {
        val next = first + second
        val current = first
        first = second
        second = next
        current
    }
}
```

### 内联函数

```kotlin
inline fun cost(block: ()->Unit){
    val start = System.currentTimeMillis()
    block()
    println("${System.currentTimeMillis() - start}ms")
}

cost {
    println("inline")
}

// 内联后等价于：
val start = System.currentTimeMillis()
println("inline")
println("${System.currentTimeMillis() - start}ms")
```

高阶函数的内联：

* 函数本身被内联到调用处
* 内联函数的函数参数被内联到调用处

## 类和接口

 类的定义

```kotlin
class SimpleClass{
    
}

// 没有内容可以省略 {}
class SimpleClass

class SimpleClass{
    // 成员必须初始化
    var x: Int = 0
    fun y(){
        
    }
}

class SimpleClass{
    var x: Int
    // 类的构造方法（辅助构造器）
    constructor(x: Int){
        this.x = x
    }
}

class SimpleClass
	// 主构造器
	constructor(x: Int){
    var x: Int = x
}
// ==>
class SimpleClass(x: Int){
    var x: Int = x
}
// ==>
class SimpleClass(var x: Int){
    
}
```

类的实例化

```kotlin
val simpleClass = SimpleClass(10)
println(simpleClass.x)
```

接口的定义

```kotlin
interface SimpleInf {
    fun method()
}
```

接口实现

```kotlin
class SimpleClass(var x: Int) : SimpleInf {
    override fun method() {

    }
}
```

抽象类的定义

```kotlin
abstract class AbsClass{
    abstract fun absMethod()
    // 默认不可覆写，使用 open 关键字声明方法可以被子类覆写
    open fun overridable(){}
    fun nonOverridable(){}
}
```

类的继承

```kotlin
class SimpleClass(var x: Int):AbsClass(), SimpleInf{}
```

属性的定义

```kotlin
class Person(age: Int, name: String) {
    // property
    var age: Int = age
        get() {
            return field
        }
        set(value) {
            field = value
        }
}

val ageRef = Person::age
val person = Person(18, "Name")
val nameRef = person::name
ageRef.set(person, 20)
nameRef.set("NewName")

```

### 内部类

```kotlin
class Outer {
    inner class Inner
    class StaticInner
}

val inner = Outer().Inner()
val statucInner = Outer.StaticInner()
```

### 匿名内部类

```kotlin
// 可以继承父类或实现多个接口
object: Runnable, Cloneable {
    override fun run(){
        
    }
}
```

### 数据类

```kotlin
data class Book(val id: Long,
               val name: String,
               val auther: Person)

val (id, name, auther) = book
```

### 枚举类

```kotlin
enum class State(val id: Int) {
    Idle, Busy
}

State.Idle.name
State.Idle.ordinal
```

### 密封类

密封类是一种特殊的抽象类，密封类的子类定义在与自身相同的文件中。

```kotlin
sealed class PlayerState
object Idle: PlayerState()
class Playing(val song: Song): PlayerState() {}
```

### 内联类

内联类是对某一个类型的包装。

```kotlin
// 包装 Int 必须是 val
inline class BoxInt(val value: Int)
```

## 扩展

Kotlin 能够对一个类或接口扩展新功能而无需继承该类或者使用像*装饰者*这样的设计模式。 这通过叫做*扩展*的特殊声明完成。

### 扩展方法

扩展方法的类型

```kotlin
fun String.times(count: Int): String{}
String::times   (String, Int) -> String
"*"::times 		(Int) -> String
```

### 扩展属性

由于扩展没有实际的将成员插入类中，因此对扩展属性来说backing field是无效的。这就是为什么*扩展属性不能有初始化器*。他们的行为只能由显式提供的 getter/setter 定义。

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

## 空类型安全

```kotlin
var nonNull: String = "Hello"
val len = nonNull.length


var nullable: String? = "World"
val len1 = nullable?.length?: 0 // elvis 运算符
val len2 = nullable!!.length 
```

空类型的继承关系

```kotlin
var x: String = "Hello"
var y: String? = "World"

y = x
```

## 类型转换

```kotlin
// 继承关系的转换
interface Kotliner
class Person(val name: String, val age: Int) : Kotliner

val kotliner: Kotliner = Person("benny", 20)
if (kotliner is Person) {
    println(kotliner.name)
}

var value: String? = null
value = ""
if(value != null) {
    println(value.length)
} 
```

## 表达式

### 运算符重载

https://book.kotlincn.net/text/operator-overloading.html

```kotlin
class MathComplex(val r: Double, val v: Double) {
    override fun toString(): String {
        return "$r + ${v}i"
    }

    operator fun plus(c: MathComplex): MathComplex {
        return MathComplex(this.r + c.r, this.v + c.v)
    }

    operator fun minus(c: MathComplex): MathComplex {
        return MathComplex(this.r - c.r, this.v - c.v)
    }

    operator fun times(c: MathComplex): MathComplex {
        return MathComplex(this.r * c.r - this.v * c.v, this.r * c.v + this.v * c.r)
    }

    operator fun div(c: MathComplex): MathComplex {
        // 未实现
        return MathComplex(0.0, 0.0)
    }
}

fun main(){
    val c1 = MathComplex(1.0,1.0)
    val c2 = MathComplex(2.0,2.0)

    println(c1 + c2)
}
```

### 中缀表达式

```kotlin
infix fun String.rotate(count: Int): String {
    val index = count % length
    return this.substring(index) + this.substring(0, index)
}

fun main(){
	println("HelloWorld" rotate 5)
}
```

## 委托

### 接口委托

对象X代替当前类A实现接口B的方法

```kotlin
// 定义接口
interface IApi {
    fun a()
    fun b()
}

// 接口的默认实现类
class ApiImpl1 : IApi {
    override fun a() {
        println("ApiImpl1.a()")
    }
    override fun b() {
        println("ApiImpl1.b()")
    }
}

// 使用静态代理，增强 a() 
class WrapperApi(private val api: IApi) : IApi {
    override fun a() {
        // enhance
        println("enhance")
        api.a()
    }
    override fun b() {
        api.b()
    }
}

// 使用委托
class DelegateApi(private val api: IApi) : IApi by api {
    override fun a() {
        println("enhance")
        api.a()
    }
}
```

### 属性委托

对象X代替属性a实现`getter/setter`方法

```kotlin

class Person1(val name: String) {
    // lazy 
    val firstName: String by lazy { name.split(" ")[0] }

    // observable
    var other: String by Delegates.observable("") { property, oldValue, newValue ->
        println("$oldValue -> $newValue")
    }
}
```

## 泛型

```kotlin
fun <T> maxOf(t1: T, t2: T): T {}


sealed class List<T>
```

### 泛型约束

```kotlin
fun <T : Comparable<T>> maxOf(t1: T, t2: T): T {
    return if (t1 > t2) t1 else t2
}

fun <T> callMax(a: T, b: T) where T : Comparable<T>, T : () -> Unit {
    if (a > b) a() else b()
}

fun <T, R> callMax(a: T, b: T): R where T : Comparable<T>, T : () -> R, R : Number {
    return if (a > b) a() else b()
}
```

### 泛型型变

泛型实参的继承关系对泛型类型的继承关系的影响：

* 协变：继承关系一致
* 逆变：继承关系相反
* 不变：没有继承关系

```kotlin
sealed class List<T> {
    object Nil: List<Nothing>()
}

// 协变
sealed class List<out T> {
    object Nil: List<Nothing>()
}

// 逆变
interface Comparable<in T>{}
```

## 反射

反射是可以在运行时自省程序结构的一组语言和库功能。

```kotlin
val c = MyClass::class
```

## 注解

### 注解概念

注解是将元数据附加到代码的方法。注解的附加属性可以通过用元注解标注注解类来指定：

- `@Target`指定可以用该注解标注的元素的可能的类型（类、函数、属性与表达式）；
- `@Retention` 指定该注解是否存储在编译后的 class 文件中，以及它在运行时能否通过反射可见 （默认都是 true）；
- `@Repeatable`允许在单个元素上多次使用相同的该注解；
- `@MustBeDocumented`指定该注解是公有 API 的一部分，并且应该包含在生成的 API 文档中显示的类或方法的签名中。

```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION,
        AnnotationTarget.TYPE_PARAMETER, AnnotationTarget.VALUE_PARAMETER, 
        AnnotationTarget.EXPRESSION)
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
annotation class Fancy
```

注解参数支持：基本类型、KClass、枚举、其它注解，以及其数组。

### 内置注解

* `kotlin.annotation.*`用于标注注解的注解
* `kotlin.*`标准库的一些通用用途的注解
* `kotlin.jvm.*`用于与Java虚拟机交互的注解

## 协程

### 协程的概念

协程是可以由程序自行控制挂起、恢复的程序。

协程可以用来实现多任务的协作执行。

协程可以用来解决异步任务控制流的灵活转移。

协程可以让异步代码同步化、降低程序的设计复杂度。

![image-20230912093622330](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/202309120936481.png)

### 协程的分类

按调用栈分类

* 有栈协程：每个协程会分配单独的调用栈，类似线程的调用栈
* 无栈协程：不会分配单独的调用栈，挂起点状态通过闭包或对象保存

按调用关系分类

* 对称协程：调度权可以转移给任意协程，协程之间是对等关系
* 非对称协程：调度权只能转移给调用自己的协程，协程存在父子关系

![image-20230912100618447](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/202309171421327.png)

### 协程的常见实现

Python Generator：无栈的非对称协程。

```python
def numbers():
    i = 0
    while True:
        yield i
        i += 1
        time.sleep(1)
        
        
gen = numbers()

print(f"[0]{next(gen)}")
print(f"[1]{next(gen)}")
```

Lua Coroutine：有栈的非对称协程。

* 创建协程：coroutine.create(\<function>)
* 查询状态：coroutine.status(\<Coroutine-Object>)
* 挂起协程：coroutine.yield(\<Values-to-Yield>)
* 恢复协程：coroutine.resume(\<Coroutine-Object>)

```lua
function producer()
    for i = 0,3 do
        print("send"..i)
        coroutine.yield(i)
    end
    print("End Producer")
end

function consumer(value)
    repeat
        print("receive"..value)
        value = coroutine.yield()
    until(not value)
    print("End Consumer")
end

producerCoroutine = coroutine.create(producer)
consumerCoroutine = coroutine.create(consumer)
repeat
    status, product = coroutine.resume(producerCoroutine)
    coroutine.resume(consumerCoroutine, product)
until(not status)
print("End Main")
```

Go rotine

```go
channel := make(chan int)
var readChannel <-chan int = channel
var writeChannel chan<- int = channel

go func(){
    for i :=0; i<3; i++{
        fmt.Println("wirte",i)
        wirteChannel <- i
    }
    close(writeChannel)
}()

go func(){
    fmt.Println("wait for read")
    for i:=range readChannel{
        fmt.Println("read",i)
    }
    fmt.Println("read end")
}()
```

async / await：无栈非对称的协程实现。

```javascript
// Promise<AxiosResponse<T>>
function getUser(name){
    return axios.get(`https://api.github.com/users/${name}`);
}

async function main(){
    const user = await getUser("JetBrains");
    console.log(user);
}
```

### 协程的基本要素

挂起函数：以suspend修饰的函数，挂起函数的调用处称为挂起点。

挂起函数只能在其它挂起函数或协程中调用。

挂起函数调用时包含了协程“挂起”的语义，挂起函数返回时则包含了协程“恢复”的语义。

```kotlin
public interface Continuation<in T> {
    public val context: CoroutineContext
    public fun resumeWith(result: Result<T>)
}

public inline fun <T> Continuation<T>.resume(value: T): Unit =
    resumeWith(Result.success(value))

public inline fun <T> Continuation<T>.resumeWithException(exception: Throwable): Unit =
    resumeWith(Result.failure(exception))
```

挂起函数的类型

```kotlin
suspend fun foo(){}			suspend () -> Unit
fun foo(continuation: Continuation<Unit>): Any{}

suspend fun bar(a: Int):String{} suspend (Int) -> String
fun bar(a:Int continuation: Continuation<String>):Any {}
```

在没有真正挂起时，Any承载返回值，真正挂起时，Any承载挂起标志`COROUTINE_SUSPENDED`。

使用suspendCoroutine获取到挂起函数的Continuation。

```kotlin
public suspend inline fun <T> suspendCoroutine(crossinline block: (Continuation<T>) -> Unit): T {}
```

协程执行过程中需要携带数据，使用协程上下文CoroutineContext表示。

拦截器ContinuationInterceptor是一类协程上下文元素，可以对协程上下文所在的Continuation进行拦截。

```kotlin
public interface ContinuationInterceptor : CoroutineContext.Element {
	public fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T>
}
```

### 协程的创建

```kotlin
public fun <T> (suspend () -> T).createCoroutine(
    completion: Continuation<T>
): Continuation<Unit> 
public fun <R, T> (suspend R.() -> T).createCoroutine(
    receiver: R,
    completion: Continuation<T>
): Continuation<Unit>

public fun <T> (suspend () -> T).startCoroutine(
    completion: Continuation<T>
) 
public fun <R, T> (suspend R.() -> T).startCoroutine(
    receiver: R,
    completion: Continuation<T>
)
```

* suspend函数本身执行需要一个Continuation实例在恢复时调用，即参数：completion。
* 返回值Continuation\<Unit>则是创建出来的协程的载体，receiver、suspend函数会被传递给该实例作为协程的实际执行体。

### 协程的启动模式

* DEAFULT：立即开始调度协程体，调度前若取消则直接取消

  ![image-20230916194348937](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/202309161944632.png)

* ATOMIC：立即开始调度，且在第一个挂起点前不能被取消

  ![image-20230916194418982](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/202309161944052.png)

* LAZY：只有在需要（start/join/await）时开始调度

  ![image-20230916194606873](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/202309161946944.png)

* UNDISPATCHED：立即在当前线程执行协程体，指导遇到第一个挂起点，后面取决于调度器

  ![image-20230916194654818](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/202309161946887.png)

### 协程的调度器

| 调度器     | JVM      | JavaScript | Native     |
| ---------- | -------- | ---------- | ---------- |
| Default    | 线程池   | 主线程循环 | 主线程循环 |
| Main       | UI线程   | 主线程循环 | 主线程循环 |
| Unconfined | 直接执行 | 直接执行   | 直接执行   |
| IO         | 线程池   | --         | --         |

 ### 协程框架

#### Channel的概念

* 非阻塞的通信基础设施，类似于BlockingQueue+挂起函数。

| 分类       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| RENDEZVOUS | send调用后挂起直到receive到达                                |
| UNLIMITED  | 无限容量，send调用后直接返回                                 |
| CONFLATED  | 保留最新，receive只能获得最近一次send的值                    |
| BUFFERED   | 默认容量，可通过`kotlinx.coroutines.channels.defaultBuffer`参数设置大小，默认为64 |

```kotlin
suspend fun main() {
    val channel = Channel<Int>()
    val producer = GlobalScope.launch {
        for (i in 0..5) {
            log("send $i")
            channel.send(i)
            log("sent $i")
        }
        channel.close()
    }
    val consumer = GlobalScope.launch {
        while (!channel.isClosedForReceive) {
            log("receiving")
            val i = channel.receiveCatching().getOrNull()
            log("received $i")
        }
    }
    producer.join()
    consumer.join()


    val receiver = GlobalScope.produce(capacity = Channel.UNLIMITED) {
        for (i in 0..5) {
            send(i)
        }
    }
    for (i in receiver) {
        println(i)
    }

    
    val sender = GlobalScope.actor<Int>(capacity = Channel.BUFFERED){
        println(receive())
    }
    for (i in 0..4) {
        sender.send(i)
    }
}
```

#### Select的概念

* Select是一个IO多路复用的概念，协程的Select用于挂起函数的多路复用。
* select总是按顺序检查事件，selectUnbiased会随机检查事件，对事件处理更公平。

#### Flow的概念

Flow是一个按顺序产生值的异步数据流。

```kotlin
val intFlow = flow {
    emit(1)
    delay(100)
    emit(2)
    emit(3)
}
intFlow.collect(::println)

listOf(1,2,3).asFlow()
flowOf(1,2,3)
val intChannel = Channel<Int>()
intChannel.consumeAsFlow()
```

使用调度器

```kotlin
intFlow
    .flowOn(Dispatchers.IO)
    .collect(::println)
```

异常处理

```kotlin
flow {
    emit(1)
    throw ArithmeticException("Div 0")
}.catch { it: Throwable ->
// 捕获之前的异常（不包括取消异常）
}.onCompletion { it: Throwable? ->
// 前序flow完成调用时，无异常时t为null
}
```

Flow的取消

* Flow的运行依赖于协程，Flow的取消取决于collect所在协程的取消。
* collect作为挂起函数可以响应所在协程的取消状态。

Back Pressure

```kotlin
intFlow.buffer(capacity = 10).collect {}
intFlow.conflate().collect {}
intFlow.collectLatest { }
```
