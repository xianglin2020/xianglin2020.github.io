---
title: Maven 基础
date: 2020-09-12 18:22:16
categories: [ learn ]
tags: [ maven ]
cover:
  image: "https://maven.apache.org/images/maven-logo-black-on-white.png"
  caption: ""
  alt: ""
  relative: false
---

# Maven

## Maven 的核心特性

* Maven 是项目管理工具，对软件项目提供构建和依赖管理
* Maven 是 Apache 下的 Java 开源项目
* Maven 为 Java 项目提供了统一的管理方式，已成为业界标准
* 项目设置遵循统一的规则，保证不同开发环境的兼容性
* 强大的依赖管理，项目依赖组件自动下载，自动更新
* [Maven 官网](https://maven.apache.org/)

### Maven 项目的标准结构

* 使用 `mvn`命令创建 Maven 项目

  `mvn -B archetype:generate -DgroupId=person.xianglin -DartifactId=myApp -DarchetypeArtifacttId=maven-archetype-quickstart -DarchetyVersion=1.4`

* 查看 Maven 项目结构

  `tree`

  ```shell
  ➜  ImoocJava tree myApp
  myApp
  ├── pom.xml
  └── src
      ├── main
      │   └── java
      │       └── person
      │           └── xianglin
      │               └── App.java
      └── test
          └── java
              └── person
                  └── xianglin
                      └── AppTest.java
  ```

  ![image-20200912224059582](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/224100.png)

## Maven 的依赖管理

* Maven 使用 Dependency自动下载，管理第三方 Jar 包
* 在`pom.xml`文件中配置项目的依赖
* Maven 自动将依赖从远程仓库下载至本地仓库，并引入项目中

## Maven 的打包方式

## Maven 常用命令

| 命令                     | 用途                |
| ------------------------ | ------------------- |
| `mvn archetype:generate` | 创建 Maven 工程结构 |
| `mvn compile`            | 编译源代码          |
| `mvn test`               | 执行测试用例        |
| `mvn clean`              | 清除产生的项目      |
| `mvn package`            | 项目打包            |
| `mvn install`            | 安装至本地仓库      |

### `archetype:generate`

1. 键入命令`mvn archetype:generate`

   ![image-20200913113443619](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/113444.png)

2. 输入项目信息

   ![image-20200913113629033](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/113629.png)

### `compile`

1. 键入编译命令`mvn compile`

   ![image-20200913113849747](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/113849.png)

2. 查看编译后的结构

   <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/113923.png" alt="image-20200913113923269" style="zoom: 50%;" />

### `test`

1. 运行当前项目中所有的测试用例：`mvn test`

   ![image-20200913114032162](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/114032.png)

### `package`

1. 将当前项目打包

   `mvn package`

   ![image-20200913114330716](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/114330.png)

### `clean`

1. 清除产生的项目

   `mvn clean`

   ![image-20200913114453103](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/114453.png)

### `install`

1. 安装项目至本地仓库

   `mvn install`

   ![image-20200913114609715](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/114609.png)

2. 查看本地仓库文件

   ![image-20200913114715053](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/120345.png)

# Maven 实战

## Maven 安装

### Maven 安装方式

1. 至[官网](https://maven.apache.org/)下载二进制压缩包解压至本地目录即可完成安装

   ```shell
   wget https://downloads.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
   
   tar -xzvf apache-maven-3.6.3-bin.tar.gz
   ```

2. 使用 homebrew 完成安装

   ```shell
   brew install maven
   ```

3. 使用`mvn -v`命令验证安装

   ```shell
   bin ./mvn -v
   Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
   Maven home: /Users/xianglin/Documents/JavaToolPackage/apache-maven-3.6.3
   Java version: 14.0.2, vendor: AdoptOpenJDK, runtime: /Library/Java/JavaVirtualMachines/adoptopenjdk-14.jdk/Contents/Home
   Default locale: zh_CN_#Hans, platform encoding: UTF-8
   OS name: "mac os x", version: "10.15.6", arch: "x86_64", family: "mac"
   ```

   ![image-20200913220832043](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/220832.png)

4. 运行`mvn help:system`

### 安装目录分析

使用`tree`命令查看 maven 的目录结构

```shell
tree -L 1 apache-maven-3.6.3
apache-maven-3.6.3
├── LICENSE
├── NOTICE
├── README.txt
├── bin
├── boot
├── conf
└── lib
```

* `bin`

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/221240.png" alt="image-20200913221240470" style="zoom:50%;" />

  `bin`目录包含了`mvn`命令运行的脚本，在命令行输入任何命令时，都是调用这些脚本。`mvnDebug`比`mvn`多了一条`MAVEN_DEBUG_OPTS`，作用是在运行 Maven 时开启 debug，以便调试Maven 本身。`m2.conf`是`classworlds`的配置文件。

* `boot`

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/222012.png" alt="image-20200913222012160" style="zoom:50%;" />

  该目录只包含一个文件，即`plexus-classworlds-2.6.0.jar`。plexus-classworlds 是一个类加载器框架，Maven 使用该框架加载自己的类库。

* `conf`

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/222245.png" alt="image-20200913222245513" style="zoom:50%;" />

  该目录包含一个非常重要的配置文件`settings.xml`，修改该文件，能在全局范围内定制Maven 的行为。

* `lib`

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202009/222505.png" alt="image-20200913222505219" style="zoom:50%;" />

  该目录包含了所有 Maven 运行时所需要的Java 类库，包含 Maven 依赖的第三方依赖。

## Maven 使用入门

### 编写`pom.xml`文件

Maven 项目的核心是`pom.xml`,`POM(Project Object Model)`项目对象模型定义了项目的基本信息，描述项目如何构建，声明项目依赖，等等。

现为 Hello-World 项目编写最简单的`pom.xml`文件

1. 创建名为`hello-world`的文件夹

   ```shell
   mkdir hello-world
   cd hello-world
   ```

2. 编写`pom.xml`文件

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">  
     <modelVersion>4.0.0</modelVersion>  
     <groupId>person.xianglin</groupId>  
     <artifactId>hello-world</artifactId>  
     <version>1.0-SNAPSHOT</version>  
     <name>Maven Hello World Project</name>
     
     <build>
     	<plugins>
     		<plugin>
     			<groupId>org.apache.maven.plugins</groupId>
     			<artifactId>maven-compiler-plugin</artifactId>
     			<version>3.1</version>
     			<configuration>
     				<source>1.8</source>
     				<target>1.8</target>
     			</configuration>
     		</plugin>
     	</plugins>
     </build>
   </project>
   ```
   

* 第一行指定了 XML 文件的版本和编码方式
* `project`元素是`pom.xml`的根元素
* `modelVersion`指定了当前 POM 模型的版本
* `groupId`、`artifactId`、`version`这三个元素定义一个项目的基本坐标。`groupId`定义项目属于哪个组，这个组一般与项目所在的公司和组织相关；`artifactId`定义当前Maven 项目在组中的唯一 ID；`version`指定当前项目的版本。
* `name`元素声明一个友好的项目名称

3. 编写主代码

   默认情况下，Maven 假定项目主代码位于`src/main/java`路径下，我们遵循 Maven 约定，创建该目录，然后在该目录下创建`person/xianglin/helloworld/HelloWorld.java`文件。

   ```shell
   mkdir -p src/main/java
   cd src/main/java
   mkdir -p person/xianglin/helloworld
   cd person/xianglin/helloworld
   vim HelloWorld.java
   ```

   ```java
   package person.xianglin.helloworld;
   
   public class HelloWorld{
       private String sayHello(){
           return "Hello Maven";
       }
   
       public static void main(String[] args){
           System.out.println(new HelloWorld().sayHello());
       }
   }
   
   ```

4. 使用 Maven 进行编译

   在项目的根路径下运行`mvn clean compile`命令

   ```shell
   mvn clean compile
   
   Building Maven Hello World Project 1.0-SNAPSHOT
   
   [INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ hello-world ---
   [INFO] Deleting /Users/xianglin/Documents/Maven/hello-world/target
   [INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ hello-world ---
   [INFO] skip non existing resourceDirectory /Users/xianglin/Documents/Maven/hello-world/src/main/resources
   
   [INFO]
   [INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ hello-world ---
   [INFO] Changes detected - recompiling the module!
   [WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
   [INFO] Compiling 1 source file to /Users/xianglin/Documents/Maven/hello-world/target/classes
   [INFO] ------------------------------------------------------------------------
   [INFO] BUILD SUCCESS
   [INFO] ------------------------------------------------------------------------
   [INFO] Total time:  1.832 s
   [INFO] Finished at: 2020-09-13T23:01:12+08:00
   [INFO] ------------------------------------------------------------------------
   ```

   Maven 首先执行了`clean:clean`任务，删除target 目录，然后执行`resources:resources`任务，最后执行`compile:compile`任务，将项目代码编译至`target/classes`目录下。

5. 编写测试代码

   Maven 项目默认的测试代码位于`src/test/java`目录下。为项目添加`junit`依赖，并编写测试代码

   ```java
   package person.xianglin.helloworld;
   import static org.junit.Assert.assertEquals;
   import org.junit.Test;
   
   
   public class HelloWorldTest{
   	@Test
   	public void testSayHello(){
   		HelloWorld helloworld = new HelloWorld();
   
   		String result helloworld.sayHello();
   
   		assertEquals("Hello Maven", result);
   	}
   }
   ```

6. 调用 Maven执行测试

   使用`mvn clean test`命令

   ```shell
   Building Maven Hello World Project 1.0-SNAPSHOT
   
   maven-clean-plugin:2.5:clean (default-clean) @ hello-world
   
   maven-resources-plugin:2.6:resources (default-resources) @ hello-world
   
   maven-compiler-plugin:3.1:compile (default-compile) @ hello-world
   
   maven-resources-plugin:2.6:testResources (default-testResources) @ hello-world
   
   maven-compiler-plugin:3.1:testCompile (default-testCompile) @ hello-world
   
   maven-surefire-plugin:2.12.4:test (default-test) @ hello-world
   
   
   -------------------------------------------------------
    T E S T S
   -------------------------------------------------------
   Running person.xianglin.helloworld.HelloWorldTest
   Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.123 sec
   
   Results :
   
   Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
   
   [INFO] ------------------------------------------------------------------------
   [INFO] BUILD SUCCESS
   [INFO] ------------------------------------------------------------------------
   [INFO] Total time:  2.921 s
   [INFO] Finished at: 2020-09-13T23:17:42+08:00
   [INFO] ------------------------------------------------------------------------
   ```

   可见，Maven 先后执行了`clean:clean resources:resources compile:compile resources:testResources compile:testCompile test:test`六个任务。最后显示输出测试报告，显示一共运行了多少、失败、错误、跳过了多少测试用例。

7. 打包和运行

   `pom.xml`中没有指定打包类型时，使用`mvn clean package`默认打包类型为 jar。

   ```shell
   [INFO]
   [INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ hello-world ---
   [INFO] Building jar: /Users/xianglin/Documents/Maven/hello-world/target/hello-world-1.0-SNAPSHOT.jar
   [INFO] ------------------------------------------------------------------------
   [INFO] BUILD SUCCESS
   [INFO] ------------------------------------------------------------------------
   [INFO] Total time:  3.320 s
   [INFO] Finished at: 2020-09-13T23:24:32+08:00
   [INFO] ------------------------------------------------------------------------
   ```

   Maven 会在打包之前执行编译、测试等操作，`jar:jar`任务负责打包，打包的文件按照`artifactId-version.jar`的格式命名。可以使用`finalName`来自定义该文件的命令。

8. 借助`maven-shade-plugin`生成可执行 jar

   在`pom.xml`中配置如下插件

   ```xml
   <plugin>
     <groupId>org.apache.maven.plugins</groupId>
     <artifactId>maven-shade-plugin</artifactId>
     <version>1.4</version>
     <executions>
       <execution>
         <phase>package</phase>
         <goals>
           <goal>shade</goal>
         </goals>
         <configuration>
           <transformers>
             <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
               <mainClass>person.xianglin.helloworld.HelloWorld</mainClass>  								
             </transformer>
           </transformers>
         </configuration>
       </execution>
     </executions>
   </plugin>
   ```

   ```shell
   java -jar hello-world-1.0-SNAPSHOT.jar
   Hello Maven
   ```

 * 使用Archetype生成项目骨架

   ```shell
   mvn archetype:generate
   ```

## 坐标和依赖

### Maven 坐标

Maven 坐标为各种构件引入了秩序，任何一个构件都必须明确定义自己的坐标，而一组 Maven 坐标是通过一些元素定义的，它们是：`groupId`、`artifactId`、`version`、`packaging`、`classifier`。

```xml
<groupId>org.sonatype.nexus</groupId>
<artifactId>nexus-indexer</artifactId>
<version>2.0.0</version>
<packaging>jar</packaging>
```

* `groupId`：定义当前项目的实际项目。由于Maven中模块的概念，一个项目往往会被划分为很多模块。
* `artifactId`：定义实际项目中的一个Maven模块，推荐做法是使用实际项目的名称作为artifactId的前缀。
* `version`：定义Maven项目当前所处的版本。
* `package`：定义Maven项目的打包方式，打包方式会影响到构建的生命周期。当不定义packaging时，Maven会使用默认值`jar`。
* `classifier`：用来帮助定义构建输出的一些辅助构建。

### 依赖的配置

一个依赖声明可以包含如下一些元素

```xml
<dependency>
    <groupId></groupId>
    <artifactId></artifactId>
    <version></version>
    <type></type>
    <scope></scope>
    <optional></optional>
    <exclusions>
      <exclusion>
      </exclusion>
    </exclusions>
</dependency>
```

* `type`：依赖的类型，对应于项目坐标定义中的`packaging`。大部分情况下，该元素不必声明，默认值为`jar`。
* `scope`：依赖的范围。
* `option`：标记依赖是否可选。
* `exclusions`：用来排除传递性依赖。

### 依赖范围

依赖范围就是用来控制依赖与三种 `classpath`（编译 classpath、测试 classpath、运行 classpath）的关系，Maven 中有以下几种依赖范围：

* `compile`：编译依赖范围，默认值。使用此依赖范围的 Maven 依赖，对于编译、测试、运行三种 classpath 都有效。

* `test`：测试依赖范围，只对测试 classpath 有效，在编译主代码和运行项目时将无法使用此类依赖，典型的例子就是 JUnit。

* `provided`：已提供依赖范围。使用此依赖范围的 Maven 依赖，对于编译和测试 classpath 有效，但在运行时无效。典型的例子就是`servlet-api`，编译和测试时需要此依赖，在运行项目时，容器已提供，就不想要重复地引入。

* `runtime`：运行时依赖范围。使用此依赖范围的 Maven 依赖，对于测试和运行 classpath 有效，但在编译主代码时无效。典型的例子是 JDBC 驱动实现，项目主代码编译时只需要 JDK 提供的 JDBC 接口，只有在执行时测试或运行项目时才需要具体的 JDBC 驱动。

* `system`：系统依赖范围。和`provided`依赖范围一致。使用 system 范围的依赖必须通过 systemPath 元素显示地指定依赖文件的路径。systemPath 可以引用环境变量。

  ```xml
  <dependency>
      <groupId>javax.sql</groupId>
      <artifactId>jdbc-stdext</artifactId>
      <version>2.0</version>
      <scope>system</scope>
      <systemPath>${java.home}/lib/rt.jar</systemPath>
  </dependency>
  ```

* `import`：导入依赖范围。该依赖范围不会对三种 classpath 产生实际的影响。

### 传递性依赖

假设 A 依赖于 B，B 依赖于 C，A 对于 B 是第一直接依赖，B 对于 C 是第二直接依赖，A 对于 C 是传递性依赖。第一直接依赖的范围和第二直接依赖的范围决定了传递性依赖的范围。

如表所示，最左边一行表示第一直接依赖，最上面一行表示第二直接依赖，中间单元格表示传递性依赖范围

|            | `compile`  | `test` | `provided` | `runtime`  |
| :--------: | :--------: | :----: | :--------: | :--------: |
| `compile`  | `compile`  |  `--`  |    `--`    | `runtime`  |
|   `test`   |`test`|`--`|`--`|`test`|
| `provided` | `provided` |  `--`  | `provided` | `provided` |
| `runtime`  | `runtime`  |  `--`  |    `--`    | `runtime`  |

### 依赖调解

1. 路径最近者优先

   比如项目A 中有如下依赖：A->B->C->X(1.0)、A->D->X(2.0)，X是 A 的传递性依赖，X(1.0)的路径长度为 3，而 X(2.0)都路径长度为 2，依据路径最近者优先原则，X(2.0)会被项目 A 解析使用。

2. 第一声明者优先

   在依赖路径长度相等的前提下，在 POM 中依赖声明的顺序决定了谁会被解析使用，顺序最靠前的那个依赖优先。

3. 覆盖优先

   在同一 POM 文件中后面声明的依赖会覆盖前面的依赖

### 可选依赖

假设项目 A 依赖于项目B，项目 B依赖于项目 X或 Y，B 对于 X 和 Y 的依赖都是可选依赖，即A->B、B->X(optional)、   B->Y(optional)。由于 X 和 Y是可选依赖，对 A 不会产生任何影响。如图所示

![项目 A](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202010/234706.png)

### 排除依赖

假设项目 A 依赖于项目 B，项目 B依赖于项目 C，但由于某些原因不想引入传递性依赖C，而是自己显示的声明对项目 C 特定版本的依赖。使用`exclusions`元素声明排除依赖，使用 `exclusion` 时只需要`groupId`和`artifictId`。

```xml
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.2.3</version>
  <exclusions>
    <exclusion>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.30</version>
</dependency>
```

