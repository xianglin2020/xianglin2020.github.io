---
title: "初始化器解析 SpringBoot源码学习"
date: 2023-11-09T19:48:24+08:00
categories:
  - SpringBoot源码学习
tags:
  - SpringBoot
summary: "SpringBoot 系统初始化器解析。"
description: "SpringBoot 系统初始化器解析。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## 系统初始化器介绍

实现 ApplicationContextInitializer 接口的类称为系统初始化器，在容器刷新前（refresh()）执行其回调方法（initialize()），用于向 SpringBoot 容器中注册属性（通常用于 WEB 环境中）。

### 自定义初始化器注入环境变量

新建三个类 FirstInitializer、SecondInitializer 和 ThirdInitializer 实现 ApplicationContextInitializer，使用 @Order 注解标记其顺序，分别为 1、2 和 3， 在 initialize 方法中为环境添加自定义属性。

```java
@Order(1)
public class FirstInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        var environment = applicationContext.getEnvironment();
        var map = Map.<String, Object>of("key1", "value1");
        var mapPropertySource = new MapPropertySource("firstInitializer", map);
        environment.getPropertySources().addLast(mapPropertySource);
        System.out.println("run firstInitializer");
    }
}
```

注册方式一，在 resources 文件夹中新建文件 META-INF/spring.factories，注册 FirstInitializer。

```properties
org.springframework.context.ApplicationContextInitializer=store.xianglin.sb2.Initializer.FirstInitializer
```

注册方式二，通过调用 `SpringApplication.addInitializers()` 方法注册 SecondInitializer。

```java
@SpringBootApplication
@MapperScan("store.xianglin.sb2.mapper")
public class SpringBoot2Application {
    public static void main(String[] args) {
//        SpringApplication.run(SpringBoot2Application.class, args);
        var springApplication = new SpringApplication(SpringBoot2Application.class);
        springApplication.addInitializers(new SecondInitializer());
        springApplication.run(args);
    }
}
```

注册方式三，通过配置文件注册 ThirdInitializer。

```properties
context.initializer.classes=store.xianglin.sb2.Initializer.ThirdInitializer
```

## SpringFactoriesLoader 介绍

`org.springframework.core.io.support.SpringFactoriesLoader` 类介绍如下。

* 它是框架内部使用的通用工厂加载机制
* 它从 classpath 下的多个 jar 包中读取 META-INF/spring.factories 文件，并加载和初始化文件中定义的类
* spring.factories 文件内容必须是 Properties 格式（key-value）
* key 是接口或者抽象类的全限定名，value 是实现类，多个实现类用 `,` 分隔

### loadFactories 流程

以 ApplicationContextInitializer 加载为例，loadFactories 流程见 SpringApplication 中 `org.springframework.boot.SpringApplication#getSpringFactoriesInstances(java.lang.Class<T>, java.lang.Class<?>[], java.lang.Object...)` 方法。

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
  ClassLoader classLoader = getClassLoader();
  // Use names and ensure unique to protect against duplicates
  // 从类路径下加载所有 ApplicationContextInitializer 的实现类
  Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
  // 实例化所有 ApplicationContextInitializer 对象
  List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
  // 按 Order 接口或注解定义的顺序排序
  AnnotationAwareOrderComparator.sort(instances);
  return instances;
}
```

在 loadFactoryNames 方法中获取 ApplicationContextInitializer 实现类分为两步，首先加载所有 META-INF/spring.factories 文件的内容，以 factoryClassName 分组保存，然后获取 ApplicationContextInitializer 对应的实现类。

```java
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
  String factoryClassName = factoryClass.getName();
  return loadSpringFactories(classLoader)
    .getOrDefault(factoryClassName, Collections.emptyList());
}
```

```java
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
  // 缓存已加载的结果
  MultiValueMap<String, String> result = cache.get(classLoader);
  if (result != null) {
    return result;
  }

  try {
    Enumeration<URL> urls = (classLoader != null ?
        classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
        ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
    result = new LinkedMultiValueMap<>();
    while (urls.hasMoreElements()) {
      URL url = urls.nextElement();
      // 遍历找到的 spring.factories 文件，读取并转成 Properties 对象
      UrlResource resource = new UrlResource(url);
      Properties properties = PropertiesLoaderUtils.loadProperties(resource);
      // 遍历 Properties 中的属性，将 value 按逗号拆分，并添加到 result 中
      for (Map.Entry<?, ?> entry : properties.entrySet()) {
        String factoryClassName = ((String) entry.getKey()).trim();
        for (String factoryName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
          result.add(factoryClassName, factoryName.trim());
        }
      }
    }
    // 缓存加载结果
    cache.put(classLoader, result);
    return result;
  }
  catch (IOException ex) {
    throw new IllegalArgumentException("Unable to load factories from location [" +
        FACTORIES_RESOURCE_LOCATION + "]", ex);
  }
}
```

获取到 ApplicationContextInitializer 的实现类后，通过全限定名加载类，然后通过反射创建对象。

```java
private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes,
			ClassLoader classLoader, Object[] args, Set<String> names) {
  List<T> instances = new ArrayList<>(names.size());
  for (String name : names) {
    try {
      // 通过类名加载类
      Class<?> instanceClass = ClassUtils.forName(name, classLoader);
      // 确保 spring.factories 中定义的类是 ApplicationContextInitializer 的实现类
      Assert.isAssignable(type, instanceClass);
      // 获取默认构造器，创建实例对象
      Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
      T instance = (T) BeanUtils.instantiateClass(constructor, args);
      instances.add(instance);
    }
    catch (Throwable ex) {
      throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
    }
  }
  return instances;
}
```

## 系统初始化器原理

系统初始化器有三种注册方法，通过 spring.factories 文件注册和编码调用 addInitializers() 方法注册都会将实现类对象添加到 `private List<ApplicationContextInitializer<?>> initializers;` 中，其调用是在 `org.springframework.boot.SpringApplication#applyInitializers` 方法完成。

```java
protected void applyInitializers(ConfigurableApplicationContext context) {
  // 遍历所有注册的初始化器
  for (ApplicationContextInitializer initializer : getInitializers()) {
    Class<?> requiredType = GenericTypeResolver.resolveTypeArgument(initializer.getClass(),
        ApplicationContextInitializer.class);
    Assert.isInstanceOf(requiredType, context, "Unable to call initializer.");
    // 调用初始化器的 initialize 方法，前面实现的注入环境变量在这里执行
    initializer.initialize(context);
  }
}
```

![image-20231109213739838](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1699537060.png)

从上图可以发现，通过配置文件注册的 ThirdInitializer 并不在 initializers 中，其加载调用是通过 DelegatingApplicationContextInitializer 类完成的。

```java
public class DelegatingApplicationContextInitializer
		implements ApplicationContextInitializer<ConfigurableApplicationContext>, Ordered {
  // 配置文件中的属性名
	private static final String PROPERTY_NAME = "context.initializer.classes";
	private int order = 0;
	@Override
	public void initialize(ConfigurableApplicationContext context) {
		ConfigurableEnvironment environment = context.getEnvironment();
    // 获取配置文件中注册的初始化器
		List<Class<?>> initializerClasses = getInitializerClasses(environment);
		if (!initializerClasses.isEmpty()) {
      // 执行初始化器逻辑，由此可发现：配置文件注册的初始化器执行顺序取决于 DelegatingApplicationContextInitializer 的顺序
			applyInitializerClasses(context, initializerClasses);
		}
	}
}
```

![image-20231109214519210](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1699537519.png)

而 DelegatingApplicationContextInitializer 是在 `spring-boot-x.x.x.jar!/META-INF/spring.factories` 中注册的。

![image-20231109215539892](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1699538140.png)
