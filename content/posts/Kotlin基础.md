---
title: Kotlin 基础
date: 2023-04-26 20:31:57
tags: [kotlin]
---

# 内置类型

## 基本类型

| 类型   | `Kotlin`         | `Java`                        |
| ------ | ---------------- | ----------------------------- |
| 字节   | `Byte`           | `byte/Byte`                   |
| 整型   | `Int & Long`     | `int/Integer & long/Long`     |
| 浮点型 | `Float & Double` | `float/Float & double/Double` |
| 字符   | `Char`           | `char/Character`              |
| 字符串 | `String`         | `String`                      |

## 声明变量

```kotlin
val b:String = "Hello Kotlin"
var a:Int = 2
```

修饰符：

* `val`：只读变量
* `var`：可读写变量

类型自动推导

```kotlin
var a = 2
var b = "Hello Kotlin"
```

## 数值类型转换

```java
int e = 10;
long f = e;
```

Kotlin

```kotlin
val e:Int = 10
val f:Long = e.toLong()
```

## 无符号类型

|        | 有符号  | 无符号   |
| ------ | ------- | -------- |
| 字节   | `Byte`  | `UByte`  |
| 短整型 | `Short` | `UShort` |
| 整型   | `Int`   | `UInt`   |
| 长整型 | `Long`  | `ULong`  |

## 字符串

字符串比较

```kotlin
val s1 = String("Today".toCharArray())
val s2 = "Today"

println(s1 == s2)
println(s1 === s2)
```

* `a == b`：比较内容
* `a === b`：比较对象引用

字符串模板

```kotlin
"Range of UInt is ${UInt.MIN_VALUE} ~ ${UInt.MAX_VALUE}"
```

## 数组

|          | `Kotlin`        | `Java`        |
| -------- | --------------- | ------------- |
| 整型     | `IntArray`      | `int[]`       |
| 整型装箱 | `Array<Int>`    | `Integer[]`   |
| 字符     | `CharArray`     | `char[]`      |
| 字符装箱 | `Array<Char>`   | `Character[]` |
| 字符串   | `Array<String>` | `String[]`    |

创建数组

```kotlin
// 创建数组
val c0 = intArrayOf(1, 2, 3)
val c1 = IntArray(5) { it + 1 }
// 打印数组
println(c0.contentToString())
// 数组长度
println(c1.size)
// 遍历数组
for (i in c1) {
    print(i)
}
c1.forEach { it -> print(it) }
// indexOf
if (1 in c1) {
    println("Exists")
}
```

## 区间

```kotlin
val intRange = 1..10 step 2
val charRange = 'a'..'z'
val ir = 1 until 10
val id = 10 downTo 1
```

## 集合框架

创建集合

```kotlin
val intList: List<Int> = listOf(1, 2, 3)
val intList2: MutableList<Int> = mutableListOf(1, 2, 3)

val map: Map<String, Any> = mapOf("name" to "benny")
val map2: MutableMap<String, Int> = mutableMapOf("first" to 1, "second" to 2)

val stringList = ArrayList<String>()
```

### 类型别名

```kotlin
public actual typealias ArrayList<E> = java.util.ArrayList<E>
public actual typealias LinkedHashMap<K, V> = java.util.LinkedHashMap<K, V>
public actual typealias HashMap<K, V> = java.util.HashMap<K, V>
public actual typealias LinkedHashSet<E> = java.util.LinkedHashSet<E>
public actual typealias HashSet<E> = java.util.HashSet<E>
```

### `Pair` & `Triple`

```kotlin
var pair = "A" to "B"
pair = Pair("A", "B")
val (x, y) = pair

val triple = Triple("F", "S", "T")
```

## 函数

### 函数定义

```kotlin
fun main(args: Array<String>): Unit {
    println(args.contentToString())
}
```

# 类和接口

## 类和接口定义

定义类

```kotlin
class Simple {
    var x: Int = 0
    fun y() {
        
    }
}

class Simple(var x: Int) {
    fun y() {

    }
}
```

定义接口

```kotlin
interface SimpleInf {
    fun simpleMethod()
}

// 实现接口
class Simple(var x: Int) : SimpleInf {
    fun y() {

    }

    override fun simpleMethod() {
        TODO("Not yet implemented")
    }
}
```

抽象类定义

```kotlin
abstract class AbsClass {
    abstract fun absMethod()

    // 可以被重写
    open fun overridable() {}
    fun noOverridable() {}
}

// 继承类
class Simple(var x: Int) : AbsClass() {

    override fun absMethod() {
        TODO("Not yet implemented")
    }

    override fun overridable() {
        println("overridable")
    }
}
```

## 扩展方法

## 空类型安全

```kotlin
val nonNull: String = "Hello"
// nonNull = null
val length = nonNull.length

var nullable: String? = "Hello"
//  nullable = null
// val len = nullable!!.length
val len = nullable?.length ?: 0
```

# 表达式

## 常量和变量

```kotlin
var a = 1
val b = 3

// 常量
const val constant = ""
```

## 分支表达式

```kotlin
c = if (a == 2) 4 else 5

when (b) {
    0 -> a = 5
    1 -> a = 6
    else -> a = 7
}

a = when {
    b == 0 -> 5
    else -> 7
}

a = try {
    c / b
} catch (e: Exception) {
    e.printStackTrace()
    0
}
```

## 运算符和中缀表达式

