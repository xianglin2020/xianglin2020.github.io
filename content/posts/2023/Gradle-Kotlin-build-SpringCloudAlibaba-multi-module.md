---
title: "Gradle+Kotlin构建SpringCloudAlibaba多模块项目"
date: 2023-10-24T18:47:39+08:00
categories:
  - learn
tags:
  - gradle
  - kotlin
summary: "使用 Gradle Kotlin 构建 SpringCloudAlibaba 多模块项目"
description: ""
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## Gradle 基础

官网：https://gradle.org/

基础教程：https://docs.gradle.org/current/userguide/part1_gradle_init.html

多项目构建：https://docs.gradle.org/current/userguide/intro_multi_project_builds.html

Gradle 插件：https://plugins.gradle.org/

## 脚手架

SCA 脚手架：https://start.aliyun.com/

SpringBoot 脚手架：https://start.spring.io/

## 案例代码

[multi-sca](https://github.com/xianglin2020/multi-sca)

## 创建 SCA 多模块项目

SpringBoot 脚手架中没有 SpringCloudAlibaba 相关依赖，SCA 脚手架中项目构建方式不支持 Gradle-Kotlin，且两个脚手架生成的都是单模块项目，所以需要使用 `gradle init` 命令生成多模块项目，然后通过脚手架查看依赖，手动创建 SCA 项目。

### 生成多模块项目

这里使用 Gradle 8.3 和 Java17。

![image-20231025194131395](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698234091.png)

使用`gradle init`命令，按提示创建项目。

![image-20231025194602991](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698234363.png)

将生成的项目用 IDEA 打开，先不执行导入，依次做如下调整：

![image-20231025195112812](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698234672.png)

调整项目的 SDK，Gradle 使用的 JVM。

![image-20231025200434104](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698235474.png)

![image-20231025200310064](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698235390.png)

![image-20231025200322358](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698235402.png)

![image-20231025200504188](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698235504.png)

删除 `settings.gradle.kts` 中的 `plugins` 元素。

修改 `gradle/wrapper/gradle-wrapper.properties` 中的 `distributionUrl` 属性，将其指向本地已下载好的 Gradle 文件，可以不用重复下载，节约时间。

```properties
# distributionUrl=https\://services.gradle.org/distributions/gradle-8.3-bin.zip
distributionUrl=file:///Users/xianglin/Downloads/gradle-8.3-all.zip
```

修改 `buildSrc/build.gradle.kts`、`buildSrc/src/main/kotlin/multi.sca.java-common-conventions.gradle.kts` 中的 `repositories` 元素，使用阿里云公共代理仓库。

```kotlin
repositories {
    mavenLocal()
    maven("https://maven.aliyun.com/repository/public")
    maven("https://maven.aliyun.com/repository/gradle-plugin")
    mavenCentral()
}
```

然后打开右侧的 Gradle 插件，重新加载 Gradle 项目。

![image-20231025202138578](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698236498.png)

可以将 app、list、utilities 三个子模块删除，并从 `settings.gradle.kts` 中删除 `include` 元素中的值。也可以删除多余的 `buildSrc/src/main/kotlin/multi.sca.java-application-conventions.gradle.kts`。

### 引入 SpringBoot 依赖

打开 https://start.spring.io/ ，Project 选择 Gradle-Kotlin ，Language 选择 Java，Spring Boot 选择 3.0.12，Java 版本选择 17，点击 `EXPLORE` 查看代码。

![image-20231025202928128](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698236968.png)

![image-20231025202944042](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698236984.png)

将脚手架生成的内容复制到 `multi.sca.java-common-conventions.gradle.kts` 中。

```kotlin
plugins {
    // Apply the java Plugin to add support for Java.
    java
    id("org.springframework.boot") version "3.0.12"
    id("io.spring.dependency-management") version "1.1.3"
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
}

repositories {
    // Use Maven Central for resolving dependencies.
    mavenLocal()
    maven("https://maven.aliyun.com/repository/public")
    maven("https://maven.aliyun.com/repository/gradle-plugin")
    mavenCentral()
}

dependencies {
    // 在此引入的依赖在所有使用此插件的模块生效
    implementation("org.springframework.boot:spring-boot-starter")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    // Use JUnit Jupiter for testing.
    testImplementation("org.junit.jupiter:junit-jupiter:5.9.3")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

tasks.withType<Test> {
    useJUnitPlatform()
}

```

重新加载 Gradle 项目，会发现如下错误，解决方法可以查看 [How to build import spring plugins](https://discuss.gradle.org/t/how-to-build-import-spring-plugins/40755)。

![image-20231025204047711](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698237647.png)

打开 https://plugins.gradle.org/ 查找 SpringBoot 插件对应的依赖，将其添加到 `buildSrc/build.gradle.kts` 的 `dependencies` 元素中，移除 `multi.sca.java-common-conventions.gradle.kts` 中插件的版本。

![image-20231025204716069](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698238036.png)

```kotlin
// buildSrc/build.gradle.kts
dependencies {
    implementation("org.springframework.boot:spring-boot-gradle-plugin:3.0.12")
    implementation("io.spring.gradle:dependency-management-plugin:1.1.3")
}

// multi.sca.java-common-conventions.gradle.kts
plugins {
    id("org.springframework.boot")
    id("io.spring.dependency-management")
}
```

重新加载 Gradle 项目，没有错误。

参照 `gradle init` 生成的 app 子模块结构，创建 spring-boot-app 子模块，依次修改 `spring-boot-app/build.gradle.kts` 和 `settings.gradle.kts`，重新加载 Gradle 项目即可。

![image-20231025205955766](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698238796.png)

![image-20231025210317385](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698238997.png)

```kotlin
// build.gradle.kts
plugins {
    id("multi.sca.java-common-conventions")
}

dependencies {
    // 因为在 multi.sca.java-common-conventions 中已经引入了相关依赖
}

// settings.gradle.kts
rootProject.name = "multi-sca"
include("spring-boot-app")
```

### 引入 SCA 依赖

创建三个子模块 sca-web 、sca-service、sca-api，分别作为WEB服务、业务服务和接口依赖项目。

在 sca-api 中创建接口 ScaControllerApi，定义一个方法，sca-web 、sca-service 通过将 sca-api 添加为依赖使用此接口。

```java
public interface ScaControllerApi {
    @GetMapping("/hello/{message}")
    ApiRes<String> hello(@PathVariable String message);
}
```

打开 https://start.aliyun.com/ 组件与示例选择 Nacos Service Discovery，点击浏览代码，打开 https://start.spring.io/ Dependencies 选择 OpenFeign ，点击 `EXPLORE` 浏览代码。

![image-20231025211840538](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698239920.png)

![image-20231025220525223](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698242725.png)

结合两个脚手架生成的内容，将其添加到项目中（仅列出新增的内容，需要将 groovy DSL 转为 Kotlin DSL）。

```kotlin
// multi.sca.java-common-conventions.gradle.kts
repositories {
    maven("https://repo.spring.io/milestone")
}

extra["springCloudAlibabaVersion"]="2022.0.0.0-RC2"
extra["springCloudVersion"]="2022.0.0-RC2"

dependencies {
    // nacos-discovery 在所有模块生效
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("com.alibaba.cloud:spring-cloud-starter-alibaba-nacos-discovery")
}

dependencyManagement{
    imports{
        mavenBom("com.alibaba.cloud:spring-cloud-alibaba-dependencies:${property("springCloudAlibabaVersion")}")
        mavenBom("org.springframework.cloud:spring-cloud-dependencies:${property("springCloudVersion")}")
    }
}
```

```kotlin
// sca-web/build.gradle.kts
plugins {
    id("multi.sca.java-common-conventions")
}

dependencies {
    implementation("org.springframework.cloud:spring-cloud-starter-loadbalancer")
    implementation("org.springframework.cloud:spring-cloud-starter-openfeign")
    implementation(project(":sca-api"))
}
```

```kotlin
// sca-service/build.gradle.kts
plugins {
    id("multi.sca.java-common-conventions")
}

dependencies {
    implementation(project(":sca-api"))
}
```

重新加载 Gradle 项目，没有错误。

依次实现 sca-web 和 sca-service，将服务注册到 nacos 中，使用 OpenFeign 实现服务间调用。

```yaml
// sca-service/src/main/resources/application.yml
spring:
  application:
    name: sca-service
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.114
server:
  port: 9900
// sca-web/src/main/resources/application.yml
spring:
  application:
    name: sca-web
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.114
server:
  port: 9901
```

```java
// sca-service/src/main/java/sca/service/ScaController.java
@RestController
public class ScaController implements ScaControllerApi {
    @Override
    public ApiRes<String> hello(String message) {
        return ApiRes.ok("sca-service: " + message);
    }
}
// sca-service/src/main/java/sca/service/ScaServiceApplication.java
@EnableDiscoveryClient
@SpringBootApplication
public class ScaServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ScaServiceApplication.class);
    }
}
```

```java
// sca-web/src/main/java/sca/web/ScaService.java
@FeignClient(name = "sca-service")
public interface ScaService extends ScaControllerApi {
}
// sca-web/src/main/java/sca/web/HelloController.java
@RestController
public class HelloController {
    private final ScaService scaService;

    public HelloController(ScaService scaService) {
        this.scaService = scaService;
    }

    @GetMapping("/hello/{message}")
    ApiRes<Map<String, String>> hello(@PathVariable String message) {
        var res = scaService.hello(message);
        return ApiRes.ok(Map.of("from", "sca-service", "result", res.getData()));
    }
}
// sca-web/src/main/java/sca/web/ScaWebApplication.java
@EnableFeignClients
@SpringBootApplication
@EnableDiscoveryClient(autoRegister = false)
public class ScaWebApplication {
    public static void main(String[] args) {
        SpringApplication.run(ScaWebApplication.class);
    }
}
```

启动 sca-service 和  sca-web 两个项目，从 Nacos 的服务列表中可以看到 sca-service 服务。

![image-20231025220752195](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698242872.png)

使用 HTTP 客户端测试，服务调用正常。

![image-20231025222357112](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698243837.png)

### 构建项目

单独构建 sca-web 或者 sca-service 模块正常，可以生成可执行文件。

![image-20231025222657713](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698244017.png)

如果直接使用项目顶级的构建会出现如下错误，解决方法可以查看 [4.3. Packaging Executable and Plain Archives](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/htmlsingle/#packaging-executable.and-plain-archives)。

![image-20231025222925576](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698244165.png)

在 `sca-api/build.gradle.kts` 中添加如下内容即可，此时构建跳过了 bootJar 任务。

```kotlin
tasks.named<BootJar>("bootJar") {
    enabled = false
}
```

![image-20231025223329891](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/10/1698244410.png)
