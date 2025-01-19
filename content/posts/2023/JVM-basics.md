---
title: "JVM基础"
date: 2023-07-31T19:44:41+08:00
description: JVM基础知识体系学习。
summary: JVM基础知识体系学习。
tags: [jvm]
categories: [learn]
---

## JVM规范

### JVM概述

* JVM：Java Virtual Machine

* 程序虚拟机：被设计用来在平台无关的环境中执行计算机程序。

* JVM是通过软件来模拟Java字节码的指令集，是Java程序的运行环境。

* 参见：[虚拟机](https://zh.wikipedia.org/wiki/%E8%99%9B%E6%93%AC%E6%A9%9F%E5%99%A8)、[Java虚拟机](https://zh.wikipedia.org/wiki/Java%E8%99%9A%E6%8B%9F%E6%9C%BA)

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308051133414.png" alt="image-20230805113351220" style="zoom:50%;" />

#### JVM的主要功能

* 通过ClassLoader寻找和装载class文件
* 解释字节码成为指令并执行，提供class文件的运行环境
* 进行运行期间的内存分配和垃圾回收
* 提供与硬件交互的平台

### JVM规范

[Java SE Specifications](https://docs.oracle.com/javase/specs/index.html)

#### JVM规范的作用

* Java的虚拟机规范为不同的硬件平台提供了一种编译Java技术代码的规范
* 该规范使Java软件独立于平台，因为编译是针对Java虚拟机的，即保障了Java的平台无关性
<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308051156971.png" alt="image-20230805115637840" style="zoom:50%;" />

#### JVM规范定义的主要内容

* 字节码指令集（相当于中央处理器CPU）
* `Class`文件格式
* 数据类型和值（范围、实现要求）
* 运行时数据区
* 栈帧
* 特殊方法
  * `<init>`：实例初始化方法，通过JVM的`invokespecial`指令来调用；
  * `<clinit>`：类或接口的初始化方法，不包含参数，返回`void`。
* 虚拟机提供基础类库（应用程序自身无法实现）
  * 反射
  * 加载和创建类或接口，如`ClassLoader`
  * 连接和初始化类和接口的类
  * 安全，如`security`
  * 多线程
  * 弱引用
* 异常处理
* 虚拟机的启动、加载、链接和初始化

### Class字节码解析

* Class文件是JVM的输入，Java虚拟机规范中定义了Class文件的结构，Class文件是JVM实现平台无关、技术无关的基础
  * Class文件是一组以8字节为单位的字节流，各个数据项目按顺序紧凑排列
  * 对于占用空间大于8字节的数据项，按照高位在前的方式分割成多个8字节进行存储
  * Class文件格式里面只有两种类型：无符号数、表
    * 无符号数：基本数据类型，以u1、u2、u4、u8来表示几个字节的无符号数
    * 表：由多个无符号数和其它表构成的复合数据类型，通常以“_info”结尾

* javap工具生成的非正式的“虚拟机汇编语言”。

* ClassFile结构

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308051537350.png" alt="image-20230805153708281" style="zoom:50%;" />

### ASM开发入门

#### ASM概述

* ASM是一个Java字节码操纵框架，能用来动态生成类或者增强既有类的功能
* ASM可以直接产生二进制class文件，也可以在类被加载入虚拟机之前动态改变类行为，ASM从类文件中读入信息后，能够改变类行为，分析类信息，甚至能根据要求生成新类
* 目前许多框架如cglib、Hibernate、Spring都直接或间接地使用ASM操作字节码

#### ASM编程模型

* Core API：提供了基于事件形式的编程模型。该模型不需要一次性将整个类的结构读取到内存中，需要内存较少，但是这种编程方式难度较大。
* Tree API：提供了基于树形的编程模型。该模型需要一次性将一个类的完整结构全部读取到内存中，需要更多的内存，但是这种编程方式较简单。
* ASM Core API 中操纵字节码的功能基于ClassVisitor接口。这个接口中每个方法对应了class文件中的每一项。
* ASM提供了三个基于ClassVisitor接口的类来实现class文件的生成和转换
  * ClassReader：解析一个类的class字节码
  * ClassAdapter：是ClassVisitor的实现类，实现要变化的功能
  * ClassWriter：是ClassVisitor的实现类，用来输出变化后的字节码

## 类加载、连接和初始化

### 类的生命周期

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308061043584.png" alt="image-20230806104325447" style="zoom:50%;" />

* 加载：查找并加载类文件的二进制数据
* 连接：将已经读入内存的类的二进制数据合并到JVM运行时环境中
  * 验证：确保被加载类的正确性
  * 准备：为类的静态变量分配内存并初始化为零值
  * 解析：把常量池中的符号引用转换成直接引用
* 初始化：为类的静态变量赋初始值

### 类加载

* 通过类的全限定名来获取该类的二进制字节流
* 把二进制字节流转化为方法区的运行时数据结构
* 在堆上创建一个`java.lang.Class`对象，用来封装类在方法区内的数据结构，并向外提供了访问方法区内数据结构的接口

#### 加载类的方式

* 从本地文件系统、jar等归档文件中加载
* 动态加载：将Java源文件动态编译成class文件
* 从网络、专有数据库加载

#### 类加载器

* JVM自带的加载器
  * 启动类加载器（BootstrapClassLoader）：用于加载启动基础模块类，比如：`java.base`、`java.management`、`java.xml`等
  * 平台类加载器（PlatformClassLoader）：用于加载一些平台相关的模块，比如：`java.scripting`、`java.compiler*`、`java.corba*`等
  * 应用程序类加载器（AppClassLoader）：用于加载应用级别的模块，比如：`jdk.compiler`、`jdk.jartool`、`jdk.jshell`等，还加载classpath路径中的所有类库
* 用户自定义加载器，是`java.lang.ClassLoader`的子类
* JDK8的类加载器
  * 启动类加载器：负责将<JAVA_HOME>/lib，或者-Xbootclasspath参数指定的路径中的，且是虚拟机识别的类库加载到内存中
  * 扩展类加载器：负责加载<JRE_HOME>/lib/ext或者java.ext.dirs系统变量所指定路径中的所有类库
  * 应用程序加载器：负责加载classpath路径中的所有类库

* JVM规范允许类加载器在预料到某个类将要被使用的时候就预先加载它

#### 类加载器的关系

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308061056543.png" alt="image-20230806105631222" style="zoom: 33%;" />

### 双亲委派模型

* 除了启用类加载器外，其余的类加载器都应该有自己的父级加载器，通过组合形成父子关系，工作过程如下

  * 一个类加载器收到类加载请求后，首先搜索它的内建加载器定义的所有“具名模块”

  * 如果找到了合适的模块定义，就会使用该加载器类加载

  * 如果class没有在这些加载器定义的具名模块中找到，那么将会委托给父级加载器，直到启动类加载器

  * 如果父级加载器反馈它不能完成加载请求，比如在它的搜索路径下找不到这个类，那子加载器才自己加载

  * 在类路径下找到的类将成为这些加载器的无名模块

* 双亲委派模型对于保证Java程序的稳定运行很重要
* 实现双亲委派的代码在`java.lang.ClassLoader`的`loadClass()`方法，如果自定义类加载器，推荐覆盖`findClass()`方法。

### 类连接

#### 类连接主要验证的内容

* 类文件结构检查：按照JVM规范规定的类文件结构进行检查
* 元数据验证：对字节码描述的信息进行语义分析，保证其符合Java语言规范的要求
* 字节码验证：通过对数据流和控制流进行分析，确保程序语义是合法和符合逻辑的，这里主要对方法体进行校验
* 符号引用验证：对类自身以外的信息，也就是常量池中的各种符号引用，进行匹配校验

#### 类连接中的解析

把常量池中的符号引用转换为直接引用的过程，主要针对：类、接口、字段、类方法、接口方法、方法类型、方法句柄、调用点限定符

* 符号引用：以一组无歧义的符号来描述所引用的目标，与虚拟机的实现无关。
* 直接引用：直接指向目标的指针，相对偏移量或是能间接定位到目标的句柄，是和虚拟机实现相关的。

### 类初始化

类的初始化就是为类的静态变量赋初始值，或者说是执行类构造器`<clinit>`方法的过程

* 如果类还没有加载和连接，就先执行加载和连接
* 如果类存在父类，且父类没有初始化，就先初始化父类
* 如果类中存在初始化语句，就依次执行这些初始化语句
* 针对接口：
  * 初始化一个类时，并不会先初始化它实现的接口（如果接口定义了默认方法，则会初始化接口）
  * 初始化一个接口时，并不会初始化它的父接口
  * 只有当程序首次使用接口中的变量或访问接口方法时，才会导致接口初始化
* 调用ClassLoader的loadClass方法，并不会初始化这个类

#### 类初始化时机

JVM必须在每个类或接口“首次主动使用”时才初始化它们。

主动使用类的时机包括：

* 创建类实例
* 访问类或接口的静态变量
* 调用类的静态方法
* 通过反射访问类
* 虚拟机启动时运行的主类

### 类的卸载

* 当代表一个类的Class对象不再被引用，那么Class对象的声明周期就结束了，对应的在方法区中的数据也会被卸载。

* JVM自带类加载器加载的类是不会被卸载的，由用户自定义类加载器加载的类可以被卸载。

## 内存分配

### JVM的简化架构

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308061529136.png" alt="image-20230806152956833" style="zoom:33%;" />

* 程序寄存器（Program Counter Register）
  * 每个线程拥有一个PC寄存器，是线程私有的，记录正在执行的虚拟机字节码指令的地址
  * 在创建线程的时候，创建相应的PC寄存器
  * 执行本地方法时，PC寄存器的值为undefined
  * 是一块较小的内存空间，是唯一一个在JVM规范中没有规定OutofMemory的区域
* 虚拟机栈
  * 栈由一系列帧（Frame）组成，是线程私有的
  * 帧用来保存一个方法的局部变量、操作数栈、常量池指针、动态链接、方法返回值等
  * 每一次方法调用时创建一个帧并压栈，退出方法时修改栈顶指针就可以把栈帧中的内容销毁
  * 局部变量表存放了编译期可知的各种基本数据类型和引用类型，每个slot存放32位的数据，long、double占两个slot
* Java堆
  * 用来存放应用系统创建的对象和数组，所有线程共享Java堆
  * GC主要管理堆空间，对分代GC来说，堆也是分代的
* 方法区
  * 通常用来保存装载的类的结构信息，是线程共享的
  * JVM规范把方法区描述为堆的一个逻辑部分，但它有一个别名Non-heap，应该是为了与Java堆区分开
* 运行时常量池
  * 是Class文件中每个类或接口的常量池表在运行期间的表现形式，通常包括：类的版本、字段、方法、接口等信息
  * 在方法区中分配，通常在加载类和接口到JVM后，就创建相应的运行时常量池

* 本地方法栈是JVM中用来支持native方法执行的栈

### Java堆内存

#### Java堆的结构

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308061554706.png" alt="image-20230806155424412" style="zoom: 25%;" />

* 新生代用来存放新分配的对象，新生代中经过多次垃圾回收没有被回收掉的对象，会被复制到老年代
* 老年代存储一些大对象
* 永久代被元空间取代，元空间直接使用本地内存

#### 对象的内存布局

* 以HotSpot为例：对象头、实例数据和对齐填充
* 对象头
  * Mark Word：存储自身的运行数据，如hashCode、GC年龄、锁状态标志等
  * 类型指针：对象指向它的类元数据的指针
* 实例数据，真正存放对象实例数据的地方
* 对齐填充，仅仅是占位符，因为HotSpot要求对象起始地址都是8字节的整数倍，如果不是就需要对齐

#### 对象的访问定位

JVM规范规定reference类型是一个指向对象的引用，对象的访问方式取决于JVM实现，目前主流的有使用句柄或使用指针两种方式。

* 使用句柄：Java堆中会划分出一块内存来作为句柄池，reference中存储句柄的地址，句柄中存储对象实例数据和类元数据的地址

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308061607760.png" alt="image-20230806160737554" style="zoom: 25%;" />

* 使用指针：reference直接存放对象实例数据的地址，实例数据中保存类元数据的地址

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308061608550.png" alt="image-20230806160808158" style="zoom: 25%;" />

### Java内存分配的参数

[Xlog Configuration](https://docs.oracle.com/en/java/javase/19/docs/specs/man/java.html#convert-gc-logging-flags-to-xlog)

#### Trace跟踪参数

* 打印GC的简要信息：`-Xlog:gc`
* 打印GC的详细信息：`-Xlog:gc*`
* 指定GC log的位置，以文件输出：`-Xlog:gc:filename`
* 每一次GC后，都打印堆信息：`-Xlog:gc+heap=debug`

#### G1日志格式

```bash
[0.090s][info][gc,start    ] GC(0) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.096s][info][gc,heap,exit]  garbage-first heap   total 10240K, used 9315K [0x00000000ff600000, 0x0000000100000000)
```

#### Java堆参数

* `Xms`：初始堆大小，默认物理内存的1/64

  > Sets the minimum and the initial size (in bytes) of the heap. This value must be a multiple of 1024 and greater than 1 MB. 

* `Xmx`：最大堆大小，默认物理内存的1/4

* `Xmn`：新生代大小，默认整个堆的3/8

* `-XX:+HeapDumpOnOutOfMemoryError`：OOM时导出堆到文件

* `-XX:+HeapDumpPath`：HeapDump的存放位置

* `-XX:NewRatio`：老年代与新生代的比值

  > Sets the ratio between young and old generation sizes. By default, this option is set to 2. 

* `-XX:SurvivorRatio`：Eden区与Survivor区的比值

  > Sets the ratio between eden space size and survivor space size. By default, this option is set to 8. 

#### Java栈参数

* `-Xss`：通常只有几百K，决定函数调用的深度

## 字节码执行引擎

### 字节码执行引擎概述

* JVM的字节码执行引擎，功能是输入字节码文件，然后对字节码进行解析并处理，最后输出执行的结果
* 实现方式有通过解析器直接解释执行字节码，通过及时编译器产生本地代码编译执行，也可能两者皆有

### 栈帧

* 栈帧是用于支持JVM进行方法调用和方法执行的数据结构
* 栈帧随着方法调用而创建，随着方法结束而销毁
* 栈帧里存储了方法的局部变量、操作数栈、动态链接、方法返回地址等信息

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308092155972.png" alt="image-20230809215549711" style="zoom:25%;" />

#### 局部变量表

用来存放方法参数和方法内部定义的局部变量的存储空间

* 以变量槽slot为单位，目前一个slot存放32位以内的数据类型
* 对于实例方法，第0位slot存放的是`this`，然后从1到n依次分配给参数列表
* 根据方法体内部定义的变量顺序和作用域来分配slot
* slot是复用的，以节省栈帧空间

#### 操作数栈

用来存放方法运行期间，各个指令操作的数据

* 操作数栈中元素的数据类型必须和字节码指令的顺序严格匹配
* 虚拟机在实现栈帧的时候可能会做一些优化，让两个栈帧出现部分重叠区域，以存放公用的数据

#### 动态链接

每个栈帧持有一个指向运行时常量池中该栈帧所属方法的引用，以支持方法调用过程的动态链接

* 静态解析：类加载时符号引用转化为直接引用
* 动态链接：运行期间转换为直接引用

#### 方法返回地址

方法执行后返回的地址

### 方法调用

确定具体调用哪一个方法，并不涉及方法内部的执行过程

* 部分方法直接在类加载的解析阶段，就确定了直接引用关系
* 对于实例方法（虚方法），因为重载和多态需要在运行期动态指派

#### 分派

* 静态分派：所有依赖静态类型来定位方法执行版本的分配方式，比如重载方法
* 动态分配：根据运行期的实际类型来定位方法执行版本的分配方式，比如覆盖方法

## 垃圾回收

### 垃圾回收基础

内存中已经不再被使用到的内存空间就是垃圾。

引用计数法：给对象添加一个引用计数器，有引用时计数器加一，引用失效时计数器减一。优点是实现简单、效率高，缺点是不能解决对象之间的循环引用。

根搜索算法：从根节点向下搜索对象节点，搜索走过的路径称为引用链，如一个对象与根之间没有连通，则该对象不可用。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308102106113.png" alt="image-20230810210616860" style="zoom:25%;" />

能作为GC Roots的对象包括：虚拟机栈中引用的对象、方法区中类静态属性引用的对象、方法区中常量引用的对象、本地方法栈中JNI引用的对象。

#### 判断垃圾的步骤

1. 根搜索算法判定对象不可用
2. 判断是否有必要执行对象的`finalize()`方法
3. 上述两步后仍未存在引用，就判定对象为垃圾

#### 判断类无用

* JVM中该类的所有实例都已经被回收

* 加载该类的ClassLoader已经被回收

* 没有任何地方引用该类的Class对象

* 无法在任何地方通过反射访问这个类

#### 引用分类

* 强引用：类似`A a = new A()`，不会被回收

* 软引用：有用但不必须的对象，是垃圾回收第二梯队的对象，在系统将要发生内存溢出异常前，会将这部分对象进行回收，用SoftReference来实现
* 弱引用：非必须对象，比软引用还弱，垃圾回收时会回收掉，用WeakReference来实现
* 虚引用：垃圾回收时会回收掉，无法通过虚引用来获取一个对象实例，用PhantomReference来实现

#### GC类型

* `MinorGC/YoungGC`：发生在新生代的收集动作
* `MajorGC/OldGC`：发生在老年代的GC，目前只有CMS收集器有单独收集老年代的行为
* `MixedGC`：收集整个新生代以及部分老年代，目前只有G1收集器会有这种行为
* `FullGC`：收集整个Java堆和方法区的GC

#### StopTheWorld

STW是Java中一种全局暂停的现象，多半是由于GC引起。所有Java代码停止运行，native代码可以执行，但不能和JVM交互。其危害是长时间服务停止，没有响应。对于HA系统，可能引起主备切换，严重危害生产环境。

#### 垃圾收集类型

* 串行收集：GC单线程内存回收，会暂停所有的用户线程，如：Serial

* 并行收集：多个GC线程并发工作，此时用户线程是暂停的，如：Parallel

* 并发收集：用户线程和GC线程同时执行（不一定是并行，可能是交替执行），不需要停顿用户线程，如：CMS

#### 跨代引用

跨代引用：一个代中对象引用另一个代中的对象。

跨代引用假说：跨代引用相对于同代引用来说只是极少数，存在相互引用关系的两个对象，应该是同时生存或同时消亡的。

#### 记忆集和写屏障

记忆集（Remembered Set）：一种用于记录从非收集区域指向收集区域的指针集合的抽象数据结构。

卡表（Card Table）：是记忆集的一种具体实现，定义了记忆集的记录精度和与堆内存的映射关系，卡表的每个元素都对应着其标识的内存区域中一块特定大小的内存块，这个内存块称为卡页（Card Page）。

写屏障可以看成是JVM对“引用类型字段赋值”这个动作的AOP，通过写屏障来实现当对象状态改变后，维护卡表状态。

### 垃圾收集算法

#### 标记清除法

标记清除算法（Mark-Sweep）分为标记和清除两个阶段，先标记出要回收的对象，然后统一回收这些对象。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308121120166.png" alt="image-20230812112027923" style="zoom:25%;" />

优点是简单。缺点是效率不高，标记清除后会产生大量不连续的内存碎片，从而导致在分配大对象时触发GC。

#### 复制算法

复制算法（Coping）：把内存分为两块完全相同的区域，每次使用其中一块。当一块分配完后，就把还存活的对象拷贝到另一块内存区域，然后把这块内存清除掉。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308121126543.png" alt="image-20230812112605293" style="zoom:25%;" />

优点是实现简单，运行高效，不用考虑内存碎片问题。缺点是只能使用一半内存。

#### 标记整理法

标记整理算法（Mark-Compact）：标记过程与标记清除算法一样，整理过程是让所有存活对象都向一端移动，然后直接清除边界以外的内存。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308121137371.png" alt="image-20230812113754140" style="zoom:25%;" />

### 垃圾收集器

#### 串行收集器

Serial收集器/Serial Old收集器，是一个单线程的收集器，在垃圾收集时会Stop-the-World。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308121153222.png" alt="image-20230812115317017" style="zoom:25%;" />

优点是简单，对于单CPU，由于没有多线程的交互开销，可能更高效，是Client模式下默认的新生代收集器。

使用`-XX:+UseSerialGC`来开启，使用Serial+Serial Old的收集器组合。

#### 并行收集器

ParNew收集器，使用多线程进行垃圾回收，在垃圾收集时会Stop-the-World。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308121201744.png" alt="image-20230812120155521" style="zoom:25%;" />

在并发能力好的CPU环境里，它停顿的时间要比串行收集器短。

是Server模式下首选的新生代收集器，且能和CMS收集器配合使用。

`-XX:ParallelGCThreads`：指定GC线程数

> Sets the number of the stop-the-world (STW) worker threads. The default value depends on the number of CPUs available to the JVM and the garbage collector selected.

新生代Parallel Scavenge收集器/Parallel Old收集器，是一款应用于新生代使用复制算法的并行收集器。但更关注吞吐量，能最高效的利用CPU，适合运行后台应用。使用`-XX:+UseParallelGC`来开启。

`-XX:MaxGCPauseMillis`：设置GC的最大停顿时间

> Sets a target for the maximum GC pause time (in milliseconds). 

#### CMS收集器

CMS（Concurrent Mark and Sweep）并发标记清除收集器分为：

* 初始标记：指标及GC Roots能直接关联到的对象；
* 并发标记：进行GC Roots Tracing的过程；
* 重新标记：修正并发标记期间，因程序运行导致标记发生变化的那一部分；
* 并发清除：并发回收垃圾对象。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308121509549.png" alt="image-20230812150919312" style="zoom:25%;" />

优点是低停顿，并发执行。缺点是并发执行对CPU资源压力大，无法处理在垃圾回收过程中产生的垃圾，采用的标记清除算法会导致大量碎片。

#### G1收集器

G1（Garbage First）收集器是一款面向服务端应用的收集器，具有如下特点：

* G1把内存划分为多个独立的区域（Region）；
* G1仍然采用分代思想，保留了新生代和老年代。但它们不再是物理隔离的，而是一部分Region的集合，且不需要Region是连续的；
* G1能充分利用多CPU、多核环境硬件优势，尽量缩短STW；
* G1整体上采用标记整理算法，局部是通过复制算法，不会产生内存碎片；
* G1的停顿是可预测，能明确指定在一个时间段内消耗在垃圾收集的时间比例；
* G1跟踪各个Region里面垃圾堆的价值大小，在后台维护一个优先列表，每次根据允许的时间来回收价值最大的区域，从而保证在有限时间内的高效收集。

G1运行分为四个阶段：

* 初始标记：只标记GC Roots能直接关联到的对象
* 并发标记：经行GC Roots Tracing的过程
* 最终标记：修正并发标记期间，因程序运行导致标记发生变化的那一部分对象
* 筛选回收：根据停顿时间来进行价值最大化的回收

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308121527538.png" alt="image-20230812152711321" style="zoom:25%;" />

#### ZGC收集器

JDK11加入的具有实验性质的低延迟收集器。

ZGC的设计目标是支持TB级别的内存容量，暂停时间低（<10ms），对整个程序吞吐量的影响小于15%。

ZGC里面的新技术：着色指针和读屏障。

### GC性能指标

吞吐量：应用代码执行时间/运行总时间；

GC负荷：GC时间/运行总时间；

暂停时间：发生STW的总时间；

GC频率：GC在一个时间段发生的次数；

反应速度：从对象成为垃圾到被回收的时间。

## 高效并发

### Java内存模型

JCP定义了一种Java内存模型，以前在JVM规范中，后来独立出来成为JSP-133（Java内存模型和线程规范修订）

内存模型：在特定的操作协议下，对特定的内存或高速缓存进行读写访问过程的抽象。Java内存模型主要关注JVM中把变量值存储到内存和从内存中取出变量值这样的底层细节。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308121623844.png" alt="image-20230812162335681" style="zoom:25%;" />

所有变量（共享的）都存储在主内存中，每个线程都有自己的工作内存。工作内存中保存该线程使用到的变量的副本。

线程对变量的所有操作（读写）都应该在工作内存中完成。

不同线程不能相互访问工作内存，交互数据要通过主内存完成。

#### 内存间交互操作

Java内存模型规定了一些操作来实现内存间交互，JVM会保证其原子性：

* lock：锁定，把变量标识为线程独占，作用于主内存变量；
* unlock：解锁，把锁定的变量释放，别的线程才能使用，作用于主内存变量；
* read：读取，把变量值从主内存读取到工作内存中；
* load：载入，把read读取到的值放入工作内存的变量副本中；
* use：使用，把工作内存中一个变量的值传递给执行引擎；
* assign：赋值，把从执行引擎接收到的值赋给工作内存里面的变量；
* store：存储，把工作内存中一个变量的值传递到主内存中；
* write：写入，把store的数据存放到主内存的变量中。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master//202308121632744.png" alt="image-20230812163241539" style="zoom:25%;" />

### 线程可见性

可见性是指当一个线程修改了变量，其它线程可以获取被修改后的值。保证可见性的方法：`volatile`、`synchronized`、`final`。

`volatile`是JVM提供的最轻量级的同步机制，用`volatile`修饰的变量，对所有的线程可见，即对`volatile`变量所做的写操作能立即反映到其它线程中。

用`volatile`修饰的变量，在多线程环境下仍然不是安全的。

`volatile`修饰的变量禁止指令重排优化。

#### 指令重排

指JVM为了优化，在条件允许的情况下，对指令进行一定的重新排列。直接运行当前能够立即执行的后续指令，避开获取下一条指令所需数据造成的等待。

指令重排规则：

* 程序顺序原则：一个线程内要保证语义的串行性；

* `volatile`规则：`volatile`变量的写，先发生于读；

* 锁规则：解锁必然发生在随后的加锁前；

* 传递性：A先于B，B先于C，那么A先于C执行；
* 线程的`start()`方法优先于它的每一个动作；
* 线程的所有操作优先于线程的终结；
* 线程的中断（`interrupt()`）先于被中断线程的代码；
* 对象的构造函数执行结束先于`finalize()`方法。

#### 线程安全的处理方法

* 不可变对象是线程安全的；
* 互斥同步：synchronized、ReentrantLock；
* 非阻塞同步：基于冲突检查的乐观锁策略。

### 锁优化

自旋：如果线程可以很快获得锁，那么可以不在OS层挂起线程，而是让线程做几个忙循环。

自适应自旋：自旋的时间不再固定，而是由前一次在同一个锁上的自旋时间和锁的拥有者状态来决定。

如果锁被占用时间很短，线程能自旋成功，那么能节省线程挂起和线程切换时间，从而提升系统性能。

锁消除：在编译代码的时候，检测到根本不存在共享数据竞争的情况自然无需同步加锁。

逃逸分析，使用`-XX:+DoEscapeAnalysis`开启逃逸分析。

* 如果一个方法中定义的一个对象，可能被外部方法引用，称为方法逃逸。
* 如果对象可能被其它外部线程访问，称为线程逃逸。

锁粗化：通常要求同步块要小，但一系列连续操作对同一个对象反复的加锁和解锁，会导致不必要的性能损耗。这种情况建议把锁同步的范围扩大到整个操作序列。

轻量级锁：没有多线程竞争的情况下，使用类似乐观锁的方式减少使用OS实现互斥所产生的性能损耗。如果轻量级锁失败，表示存在竞争则升级为重量级锁。

偏向锁：在无竞争情况下，锁偏向于当前已经占有锁的线程。获得偏向锁的线程，在将来进入同步块也不需要做同步。

## 性能监控和故障处理工具

### 命令行工具

* jps（JVM Process Status Tool）：主要用来输出JVM中运行的进程状态信息；

* jinfo：打印给定进程或核心文件或远程调试服务器的配置信息；

* jstack：主要用来查看某个Java进程内的线程堆栈信息；

* jmap：用来查看堆内存使用情况；

* jstat：JVM统计监测工具，用于查看JVM各个区内存和GC情况；

* jstatd：虚拟机的jstat守护进程，主要用于监控JVM的创建与终止，并提供一个接口，以允许远程监视工具附加到本地系统上的JVM；

* jcmd：JVM诊断命令工具，将诊断命令请求发送到正在运行的Java虚拟机。比如可以用来导出堆，查看Java进程，导出线程信息，执行GC等。

### 图像化工具

* jconsole：一个用于监视Java虚拟机的符合JMX的图像工具。它可以监视本地和远程JVM，还可以监视和管理应用程序。

* jmc（JDK Mission Control）：Java任务控制客户端包括用于监视和管理Java应用程序的工具，优点是不会引入与这些类型工具相关联的性能开销。下载地址：https://www.oracle.com/java/technologies/jdk-mission-control.html。

  使用命令行工具记录JFR：`jcmd 15592 JFR.start delay=10s duration=1m filename="C:\Users\xiang\Desktop\log.jfr"`

* VisualVM：提供有关在Java虚拟机中运行的基于Java技术的应用程序详细信息：内存和CPU分析、堆转存分析、内存泄漏、访问MBean和垃圾收集。下载地址：https://visualvm.github.io/。

### 两种远程连接方式

* JMX连接可以查看：系统信息、CPU使用情况，线程情况、手动执行GC等比较偏于系统级层面的信息；

  JMC 远程连接参数示例：

  ```bash
  # 有多个网卡时，通过 java.rmi.server.hostname 指定，不然连不上
  jdk-21.0.5+11/bin/java  -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.port=7091 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=192.168.56.103  -jar jvm-goods-0.0.1-SNAPSHOT.jar
  ```

  > https://docs.oracle.com/html/E28023_02/mc_comm.htm#A1005288
  >
  > https://docs.oracle.com/en/java/java-components/jdk-mission-control/9/user-guide/faq.html#GUID-9B310BA1-A7FB-45AB-879D-F762E1AB34EF
* jstatd连接可以提供：JVM内存分布详细信息、垃圾回收分布图、线程详细信息，以及某个对象使用内存的大小。

 # JVM调优基础

### JVM调优涉及的内容

内存方面

* JVM需要的内存总大小；
* 各块内存分配的大小（新生代、存活区、老年代等）；
* 选择合适的垃圾回收算法（垃圾收集器），控制GC停顿的次数和时间；
* 通过JVM监控信息查看热点代码，辅助分析解决内存泄漏问题（优化代码）。

线程方面

* 通过检查线程死锁情况，辅助优化代码；
* Dump线程详细信息，查看线程内部运行情况，查找竞争线程，辅助代码优化；
* 检查系统哪些方法占用了大量CPU时间，辅助代码优化。

### JVM调优步骤

1. 监控JVM的状态，主要是内存、线程、代码和I/O部分；
2. 分析监控信息，判断是否需要优化；
3. 调整内容：垃圾收集器和内存分配、修改优化代码；
4. 不断的重复“监控—分析—调整”三个步骤，直至达到调优的平衡点。

### JVM调优目标

* GC的时间足够小（单次GC时间和总时间）；
* GC的次数足够少；
* 将转移到老年代的对象数量降低到最小；
* 减少Full GC的执行时间；
* 发生Full GC的间隔足够长。

### 常见调优策略

* 减少创建对象的数量；
* 减少使用全局变量和大对象；
* 调整新生代、老年代大小到最合适；
* 选择合适的垃圾收集器，并设置合理的参数。
