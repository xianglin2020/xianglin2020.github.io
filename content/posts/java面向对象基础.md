---
title: java 面向对象基础
date: 2020-09-06 22:50:16
categories: [learn]
tags: [java]
---

# Java 面向对象

## 对象

在 Java 中，万物皆对象，只要是现实存在的，都可以是对象。

## 面向对象

## 类

* 类是模型，确定对象将会有的属性（特征）和方法（行为）

* 对象是类的实例体现。

* 定义一个类`Cat`描述猫

  ```java
  public class Cat {
      // 属性：昵称、年龄、体重、品种
      /**
       * String 类型的默认值是 null
       * 昵称
       */
      String name;
      /**
       * int 类型的默认值是 0
       * 年龄
       */
      int month;
      /**
       * double 类型的默认值是 0.0
       * 体重
       */
      double weight;
      /**
       * 品种
       */
      String species;
  
      // 方法：跑动、吃东西
      /**
       * 跑动
       */
      public void run() {
          System.out.println("小猫快跑！");
      }
  
      public void run(String name) {
          System.out.println(name + "快跑！");
      }
  
      /**
       * 吃东西
       */
      public void eat() {
          System.out.println("小猫吃鱼！");
      }
  }
  ```


## 对象实例化

* 测试`Cat`对象的方法和属性

  ```java
  public class CatTest {
      public static void main(String[] args) {
          // 对象实例化
          Cat one = new Cat();
          // 测试方法
          one.eat();
          one.run();
  
          // 为属性赋值
          one.name = "花花";
          one.month = 2;
          one.weight = 1000;
          one.species = "英国短毛猫";
  
          // 输出属性
          System.out.println("年龄：" + one.month);
          System.out.println("昵称：" + one.name);
          System.out.println("体重：" + one.weight);
          System.out.println("品种：" + one.species);
  
          one.run(one.name);
      }
  }
  ```

  ### 对象实例化步骤

  * 声明对象 `Cat cat`
  * 实例化对象 `new Cat()`

  ## 构造方法
  
* 构造方法也称为构造器
  
  * 构造方法与类名相同且没有返回值
  
* 构造方法只能在对象实例化的时候调用
  * 当没有指定构造方法时，编译器会自动添加无参的构造方法。

  ### This 关键字

  `this` 是 Java中常用的关键字，代表当前对象本身，可以理解为：指向当前对象的一个引用。
  
  `this` 在Java中可用于调用成员属性、成员方法、构造方法，也可以当作参数进行方法传参以及方法返回值。
  
  * 可以在构造方法中，通过`this`调用本类中的另一个构造方法，但是，调用动作必须置于方法中的第一行。
  * 可以通过`this`调用本类中带参或者无参构造方法，调用带参构造方法时，需要按顺序传入设定的参数。
  
  
  ## 面向对象设计原则
  
  ### 单一职责原则
  
  单一职责原则（SRP：Single responsibility principle）又称单一功能原则。它规定一个类应该只有一个发生变化的原因。所谓职责是指类变化的原因，如果一个类有多余一个的动机被改变，name 这个类就具有多余一个职责。
  
  单一职责原则表明，一个类承担的职责越多，它被复用的可能性及越小，而且一个类承担的职责过多，就相当于将这些职责耦合在一起，当其中一个职责变化时，可能会影响到其它职责的运作，因此要将这些职责进行分离，将不同职责封装在不同的类中，即将不同的变化原因封装在不同的类中，如果多个职责总是同时发生改变则可将它们封装在同一类中。

# Java 封装

* 将类的某些信息在类内部，不允许外部程序访问
* 通过该类提供的方法，访问隐藏的属性
* 封装类（对象）的属性，隐藏实现的细节，方便修改和维护
* 提供改变的接口

## Java 封装的实现

1. 修改属性的可见性，设为`private`
2. 创建`getter`、`setter`方法，设为`public`用于属性的读写
3. 在方法方法中对属性进行操作

## 使用包进行类管理

* `package`语句必须放在 Java 源文件中的第一行，一个 Java 源文件只能有一条`package`语句。
* `import`语句只能加载包路径下直接访问的类

* 使用`package`定义包

  ```java
  package person.xianglin.pkg;
  ```

* 使用`import`引入类

  1. 使用`import`引入包下所有类

  	```java
  	import person.xianglin.pkg;
  	```

  2. 使用`import`引入包下特定类

     ```java
     import person.xianglin.pkg.Cat;
     ```


## `Static`关键字

* `static`是 Java 中常用的关键字，代表“全局”或者“静态”的意思。通常用于修饰成员变量和方法，页可以用于静态代码块。
* 静态成员是属于整个类的，由类进行维护，仅在类初次加载时会被初始化，在类销毁时回收。
* 通过该类实例化的所有对象都共享类中静态资源，任一对象中信息的修改都将影响所有对象。

# Java 继承

面向对象的编程思想来源为生活

## 继承

* 描述一种类与类之间的关系
* 使用已存在的类定义作为基础建立新类
* 新类的定义可以增加新的数据或新的功能，也可以使用父类的功能，但不能选择性地继承父类
* 满足`A is a B`的关系就可以形成继承关系
* Java 使用`extends`实现继承关系
* 子类可以获取父类非私有的成员

## 方法重写

* 在子类中定义，方法名、参数列表与父类中方法一致
* 方法重载：
  1. 在同一个类中
  2. 方法名相同，参数列表不同（参数顺序、个数、类型）
  3. 方法返回值、访问修饰符任意
* 方法重写：
  1. 在有继承关系的子类中
  2. 方法名相同，参数列表相同（参数顺序、个数、类型）
  3. 与方法的参数名无关
  4. 方法返回值小于等于父类方法返回值
  5. 方法的访问修饰符大于等于父类的访问修饰符

## 访问修饰符

* 共有的：`public`，允许在任意位置访问
* 私有的：`private，只能在本类中访问`
* 保护的：`protected`，能在当前类、相同包中的类及子类中访问
* 默认：允许在相同包中被访问

| 访问修饰符  | 本类 | 同包 | 子类 | 其他 |
| :---------: | :--: | :--: | :--: | :--: |
|  `public`   |  √   |  √   |  √   |  √   |
| `protected` |  √   |  √   |  √   |      |
|  `default`  |  √   |  √   |      |      |
|  `private`  |  √   |      |      |      |

## 继承的初始化顺序

```java
public class BaseClass {
    static {
        System.out.println("父类的静态代码块");
    }

    {
        System.out.println("父类的构造代码块");
    }

    public BaseClass() {
        System.out.println("父类的构造方法");
    }
}

public class SubClass extends BaseClass {
    static {
        System.out.println("子类的静态代码块");
    }

    {
        System.out.println("子类的构造代码块");
    }

    public SubClass() {
        System.out.println("子类的构造方法");
    }
}

```

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/212507.png" alt="image-20200602213033364" style="zoom: 67%;" />

## `final`关键字

* 修饰类：表示类不能被继承
* 修饰方法：表示方法不能被重写
* 修饰变量：对于基本数据类表示变量的值不能改变，对于引用数据类型表示引用指向的地址不能改变
  * 修饰局部变量：局部变量在使用前赋值即可
  * 修饰成员变量：定义时初始化、构造块初始化、构造方法里初始化
  * 修饰静态变量：定义时初始化，静态代码块里初始化

## 注解简介

* JDK1.5引入注解
* 用来对元素进行说明、注释

# Java 多态

## 多态实现形式

* 编译时多态：方法重载
* 运行时多态：方法重写

## 实现多态的条件

* 满足继承关系
* 父类引用指向子类应用

## 转型

* 向上转型：隐式转型、自动转型，父类引用指向子类对象
* 向下转型：强制转型：子类引用指向父类对象

## 抽象类

* 为子类提供一个公共的引用类型
* 封装子类中的重复内容
* 父类需要限定子类应该包含的方法，但无法准确知道方法的实现。
* 抽象方法必须包含在抽象类中，抽象类中可以没有抽象方法
* 抽象方法使用`abstract`修饰，不允许有方法体

# 接口 `interface`

* Java 只支持单继承
* 使用`interface`定义接口
* 使用`implements`实现接口
* 接口的访问修饰符：`public`或者 default
* 接口方法的修饰语默认是：`public abstract`
* 接口中可以定义常量，常量的修饰语是：`public static final`
* JDK1.8 后接口方法可以用`default`修饰，提供默认实现；可以创建静态方法
* 当类实现接口时，需要实现接口所有方法，或者将类修饰为抽象类

# 内部类

*  Java 中可以将一个类定义在另一个类的内部或方法的内部，这样的类称为内部类；包含内部类的类称为外部类。
* 内部类能将信息封装在类的内部

## 成员内部类

* 内部类中最常见的就是成员内部类，也称为普通内部类

* 获取内部类

  * 方式一：`new outerClass().new innerClass();`
  * 方式二：使用外部类对象获取内部类：`outer.new innerClass()`
  * 方式三：获取内部类对象实例

  ```java
  public InnerClass getInner() {
      return new InnerClass();
   }
  ```

* 内部类可以直接访问外部类的成员属性和方法，如果重名，优先访问内部类属性

* 可以使用 `OutClass.this.field` 访问外部类的信息

* 成员内部类无法定义静态变量、静态方法。外部类访问内部类属性需要通过内部类实例对象访问

## 静态内部类

* 静态内部类创建可以不依赖外部类
* 获取静态内部类实例：`Outer.Inner myHeart = new Outer.Inner();`
* 静态内部类实例化时，不依赖外部类
* 可以通过 `Outer.Inner.method` 访问内部类的静态资源

## 方法内部类

* 定义在方法中的类，也称为局部内部类
* 方法内部类的成员只能在方法内部使用
* 方法前面不能加访问修饰符和 `static`
* 类中不能包含静态成员
* 方法内部类可以定义为抽象类，类中可以定义抽象方法

## 匿名内部类

* 将类的定义与对象实例创建一起完成
* 匿名内部类没有类型名称、实例对象名称
* 编译后的文件命名：外部类$数字.class
* 无法使用访问修饰符
* 无法编写构造方法

# 面试题

## 面向对象的三大特征

## 接口和抽象类

## 静态变量与实例变量

* 使用`static`关键字定义静态变量
* 静态变量属于类，不需要依附于实例
* 静态变量在类加载时初始化与 JVM 的方法区，实例变量在对象创建时初始化与堆内存中

## 代码块的执行顺序

* 静态块优先
* 父类优先
* 构造块由于构造函数

```java
public class OrderDemo {
    public static void main(String[] args) {
        new SubClass();
    }
}

class Parent {

    Parent() {
        System.out.println("父类构造器");
    }

    {
        System.out.println("父类构造块");
    }

    static {
        System.out.println("父类静态块");
    }
}

class SubClass extends Parent {
    SubClass() {
        System.out.println("子类构造器");
    }

    {
        System.out.println("子类构造块");
    }

    static {
        System.out.println("子类静态块2");
    }

    static {
        System.out.println("子类静态块1");
    }
}

/*
父类静态块
子类静态块2
子类静态块1
父类构造块
父类构造器
子类构造块
子类构造器
*/
```

## Java 的异常体系

### Exception 和 Error 的区别

## 字符串与常量池

* 字符串不可变与字符串引用不可变是两个概念
* 可以通过反射改变`value[]`的值，不过没有实际意义
* JDK9 开始，存储字符串的数组由`private final char[] value` 变为`private final byte[] value`

```java
import java.lang.reflect.Field;

public class StringDemo {
    public static void main(String[] args) throws Exception {
        String str = "abcdefg";
        System.out.println(str);
        Field valueField = String.class.getDeclaredField("value");
        valueField.setAccessible(true);
      	// java9 
        byte[] obj = (byte[]) valueField.get(str);
        obj[0] = (byte) (obj[0] - 32);
        System.out.println(str);
    }
}

// abcdefg
// Abcdefg
```

## String、StringBuilder、StringBuffer

* `String` 不可变，使用`final byte[] value`存储字符
* `StringBuilder`、`StringBuffer`继承自`AbstractStringBuilder`，字符串存储于父类的`byte[] value`
* `StringBuffer`主要方法均使用`synchronized`保证线程安全
* `AbstractStringBuilder`并没有实现`equals`方法

## Java IO

### 从装饰者模式理解 Java IO

### 文件拷贝

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;

public class FileCopyDemo {

    public static void main(String[] args) {
        copyFile("/Users/xianglin/Documents/IdeaProjects/share-mybatis-plugin.zip",
                "/Users/xianglin/Documents/IdeaProjects/share-mybatis-plugin");
    }

    public static void copyFile(String srcFilePath, String descDirPath) {
        File srcFile = new File(srcFilePath);
        File descDir = new File(descDirPath);
        if (!srcFile.exists() || !srcFile.isFile()) {
            return;
        }
        if (descDir.isFile()) {
            return;
        }
        if (!descDir.exists()) {
            if (descDir.mkdirs()) {
                return;
            }
        }

        int index = srcFilePath.lastIndexOf("/");
        String fileName = srcFilePath.substring(index);

        try (FileInputStream inputStream = new FileInputStream(srcFile);
                FileOutputStream outputStream = new FileOutputStream(new File(descDir, fileName))) {
            byte[] bytes = new byte[1024];
            int length;
            while ((length = inputStream.read(bytes)) != -1) {
                outputStream.write(bytes, 0, length);
            }
        } catch (Exception exception) {
            exception.printStackTrace();
        }
    }
}
```

