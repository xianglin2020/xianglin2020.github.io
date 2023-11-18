---
title: "Banner解析 SpringBoot源码学习"
date: 2023-11-18T11:59:42+08:00
categories:
  - SpringBoot源码学习
tags:
  - SpringBoot
summary: "SpringBoot Banner解析、效果演示。"
description: "SpringBoot Banner解析、效果演示。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## Banner 演示

### 通过配置文件指定

通过配置 `spring.banner.location` 指定文字 Banner 文件，通过配置 `spring.banner.image.location` 指定图片 Banner 文件，通过配置 `spring.main.banner-mode` 设置 Banner 展示模式：OFF —关闭 Banner，CONSOLE —输出到 `System.out`，LOG—打印到日志文件 。

```properties
spring.banner.location=favorite.txt
spring.banner.image.location=favorite.jpg
spring.main.banner-mode=console
```

可以同时配置文字 Banner 和图片 Banner。

![image-20231118165621890](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700297782.png)

### 通过编码指定

在未使用配置文件指定和没有默认配置时生效。

```java
@SpringBootApplication
@MapperScan("store.xianglin.sb2.mapper")
public class SpringBoot2Application {
    public static void main(String[] args) {
        var springApplication = new SpringApplication(SpringBoot2Application.class);
        springApplication.setBanner(new ResourceBanner(new ClassPathResource("favorite.txt")));
        springApplication.setBannerMode(Banner.Mode.CONSOLE);
        springApplication.run(args);
    }
}
```

## Banner 打印原理

Banner 打印是通过在 `SpringApplication#run` 方法中调用 printBanner 方法，printBanner 的逻辑为：获取 Banner，根据 Banner.Mode 打印 Banner。

```java
Banner printedBanner = printBanner(environment);
// 根据 Banner.Mode 选择打印方式
private Banner printBanner(ConfigurableEnvironment environment) {
  if (this.bannerMode == Banner.Mode.OFF) {
    return null;
  }
  ResourceLoader resourceLoader = (this.resourceLoader != null) ? this.resourceLoader
      : new DefaultResourceLoader(getClassLoader());
  SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(resourceLoader, this.banner);
  if (this.bannerMode == Mode.LOG) {
    return bannerPrinter.print(environment, this.mainApplicationClass, logger);
  }
  return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
}
// 打印逻辑：获取 Banner、打印 Banner
public Banner print(Environment environment, Class<?> sourceClass, PrintStream out) {
  Banner banner = getBanner(environment);
  banner.printBanner(environment, sourceClass, out);
  return new PrintedBanner(banner, sourceClass);
}
```

### Banner 内容获取

以 Banner.Mode.CONSOLE 为例，获取 Banner 的逻辑见 `org.springframework.boot.SpringApplicationBannerPrinter#getBanner`方法。

```java
private Banner getBanner(Environment environment) {
  // Banners：Banner 容器，private final List<Banner> banners = new ArrayList<>();
  Banners banners = new Banners();
  // 添加图片 Banner
  banners.addIfNotNull(getImageBanner(environment));
  // 添加文字 Banner
  banners.addIfNotNull(getTextBanner(environment));
  if (banners.hasAtLeastOneBanner()) {
    return banners;
  }
  // fallbackBanner 为编码指定的 Banner
  if (this.fallbackBanner != null) {
    return this.fallbackBanner;
  }
  // 默认 Banner，输出 Spring 横幅
  return DEFAULT_BANNER;
}
```

getImageBanner 和 getTextBanner 的逻辑基本一致，首先判断是否通过配置文件指定 Banner location，然后判断是否存在默认配置。

```java
private Banner getImageBanner(Environment environment) {
  // BANNER_IMAGE_LOCATION_PROPERTY : spring.banner.image.location
  String location = environment.getProperty(BANNER_IMAGE_LOCATION_PROPERTY);
  if (StringUtils.hasLength(location)) {
    Resource resource = this.resourceLoader.getResource(location);
    return resource.exists() ? new ImageBanner(resource) : null;
  }
  // IMAGE_EXTENSION : "gif", "jpg", "png" 
  for (String ext : IMAGE_EXTENSION) {
    Resource resource = this.resourceLoader.getResource("banner." + ext);
    if (resource.exists()) {
      return new ImageBanner(resource);
    }
  }
  return null;
}
```

### Banner 内容输出

方法 `org.springframework.boot.Banner#printBanner` 的实现有 5 个，其中 PrintedBanner 和 Banners 不涉及打印逻辑，其余类的打印逻辑比较简单。

![image-20231118173145624](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700299905.png)
