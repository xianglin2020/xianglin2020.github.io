---
title: "属性配置解析 SpringBoot源码学习"
date: 2023-11-19T08:22:19+08:00
categories:
  - SpringBoot源码学习
tags:
  - SpringBoot
summary: "SpringBoot 属性配置介绍、Aware 介绍、Environment 解析、Spring profile 解析。"
description: "SpringBoot 属性配置介绍、Aware 介绍、Environment 解析、Spring profile 解析。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## 属性配置介绍

https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config

Spring 属性有三种使用方式。

* 使用 @Value 注解注入到 Bean 中
* 使用 Environment 抽象类访问
* 使用 @ConfigurationProperties 与配置对象绑定

后面的属性源可以覆盖前面属性源中定义的值，Spring 属性源优先级按如下顺序。

1. 默认属性，使用 `springApplication.setDefaultProperties(properties)` 方法指定。
2. 配置类上使用 @PropertySource 注解，Application Context 刷新后属性会添加到 Environment 中。
3. 配置文件，例如 application.properties（properties 优先级高于 yml）。
    1. jar 包内部的 application.properties 和 application.yml
    2. jar 包内部的 application-{profile}.properties 和 application-{profile}.yml
    3. jar 包外部的 application.properties 和 application.yml
    4. jar 包外部的 application-{profile}.properties 和 application-{profile}.yml
4. 随机值 RandomValuePropertySource。
5. 操作系统环境变量。
6. Java 系统属性，可以通过 `System.getProperties()` 获取。
7. 来自 java:comp/env 的 JNDI 属性。
8. ServletContext 启动参数。
9. ServletConfig 启动参数。
10. 通过 SPRING_APPLICATION_JSON 指定的属性。
11. 命令行参数。

## Spring Aware

### 常用 Aware

* BeanNameAware：获取该 Bean 在容器中的名称。
* BeanClassLoaderAware：获取加载该 Bean Class 的类加载器。
* BeanFactoryAware：获取该 Bean 的 BeanFactory。
* EnvironmentAware：获得环境变量 Environment。
* EmbeddedValueResolverAware：获取 Spring 容器加载的 properties 文件属性值。
* ResourceLoaderAware：获取资源加载器。
* ApplicationEventPublisherAware：获取应用事件发布器。
* ApplicationContextAware：获取当前应用上下文。

### Aware 调用原理

新建启动加载器 ResultCommandlineRunner 实现 EnvironmentAware 接口，在 setEnvironment 方法中打断点，观察方法调用路径。

```java
@Component
public class ResultCommandlineRunner implements CommandLineRunner, EnvironmentAware {
    private Environment environment;

    @Override
    public void run(String... args) throws Exception {
        System.out.println(environment.getProperty("property"));
    }

    @Override
    public void setEnvironment(Environment environment) {
      	// debug here
        this.environment = environment;
    }
}

```

![image-20231119102223755](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700360544.png)

通过调试发现 Aware 调用的入口在 initializeBean 方法中。

```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
  // ...
  invokeAwareMethods(beanName, bean);

  Object wrappedBean = bean;
  if (mbd == null || !mbd.isSynthetic()) {
    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
  }
  // ...
  return wrappedBean;
}
```

Aware 调用有两个入口，一个是 invokeAwareMethods，这里调用了 BeanNameAware、BeanClassLoaderAware 和 BeanFactoryAware。

```java
private void invokeAwareMethods(final String beanName, final Object bean) {
  if (bean instanceof Aware) {
    if (bean instanceof BeanNameAware) {
      ((BeanNameAware) bean).setBeanName(beanName);
    }
    if (bean instanceof BeanClassLoaderAware) {
      ClassLoader bcl = getBeanClassLoader();
      if (bcl != null) {
        ((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
      }
    }
    if (bean instanceof BeanFactoryAware) {
      ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
    }
  }
}
```

另一个是 applyBeanPostProcessorsBeforeInitialization，通过 Bean 后置处理器 ApplicationContextAwareProcessor 的 postProcessBeforeInitialization 方法调用 invokeAwareInterfaces 方法，完成剩余的 Aware 接口调用。

```java
// ApplicationContextAwareProcessor.java
public Object postProcessBeforeInitialization(final Object bean, String beanName) throws BeansException {
  // ...
  invokeAwareInterfaces(bean);
  // ...
  return bean;
}

private void invokeAwareInterfaces(Object bean) {
  if (bean instanceof Aware) {
    if (bean instanceof EnvironmentAware) {
      ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
    }
    if (bean instanceof EmbeddedValueResolverAware) {
      ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
    }
    if (bean instanceof ResourceLoaderAware) {
      ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
    }
    if (bean instanceof ApplicationEventPublisherAware) {
      ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
    }
    if (bean instanceof MessageSourceAware) {
      ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
    }
    if (bean instanceof ApplicationContextAware) {
      ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
    }
  }
}
```

### 自定义实现 Aware

通过继承 Aware 接口定义 MyAware 接口，通过实现 BeanPostProcessor 接口，在 postProcessBeforeInitialization 方法中完成 MyAware 的调用。

```java
public interface MyAware extends Aware {
    void setFlag(Flag flag);
}

@Component
public class MyAwareProcessor implements BeanPostProcessor {
    private final ConfigurableApplicationContext context;

    public MyAwareProcessor(ConfigurableApplicationContext context) {
        this.context = context;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof Aware) {
            if (bean instanceof MyAware) {
                ((MyAware) bean).setFlag(context.getBean("flag", Flag.class));
            }
        }
        return bean;
    }
}
```

使用方式同 Spring 预定义的 Aware 一样，通过实现 MyAware 的 setFlag 方法。

```java
@Component
public class ResultCommandlineRunner implements CommandLineRunner, MyAware {
    @Override
    public void run(String... args) throws Exception {
        System.out.println(flag.isCanOperate());
    }
  
    private Flag flag;

    @Override
    public void setFlag(Flag flag) {
        this.flag = flag;
    }
}
```

## Environment 解析

### 获取属性

`environment.getProperty("property")` 调用流程如下，最终遍历 propertySources 读取属性。

```java
// AbstractEnvironment.java
public String getProperty(String key) {
  return this.propertyResolver.getProperty(key);
}

// PropertySourcesPropertyResolver.java
public String getProperty(String key) {
  return getProperty(key, String.class, true);
}
protected <T> T getProperty(String key, Class<T> targetValueType, boolean resolveNestedPlaceholders) {
  if (this.propertySources != null) {
    // 从 propertySources 中获取属性
    for (PropertySource<?> propertySource : this.propertySources) {
      Object value = propertySource.getProperty(key);
      if (value != null) {
        if (resolveNestedPlaceholders && value instanceof String) {
          value = resolveNestedPlaceholders((String) value);
        }
        logKeyFound(key, propertySource, value);
        return convertValueIfNecessary(value, targetValueType);
      }
    }
  }
  return null;
}
```

### 属性加载解析

Spring 属性解析的入口在 `org.springframework.boot.SpringApplication#run(java.lang.String...)` 方法的 prepareEnvironment 子方法中。

```java
public ConfigurableApplicationContext run(String... args) {
  // ...
  ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
  // ...
}

private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
    ApplicationArguments applicationArguments) {
  // 根据容器类型创建对应的 Environment
  ConfigurableEnvironment environment = getOrCreateEnvironment();
  configureEnvironment(environment, applicationArguments.getSourceArgs());
  ConfigurationPropertySources.attach(environment);
  listeners.environmentPrepared(environment);
  bindToSpringApplication(environment);
  if (!this.isCustomEnvironment) {
    environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
        deduceEnvironmentClass());
  }
  ConfigurationPropertySources.attach(environment);
  return environment;
}
```

首先根据容器类型 webApplicationType 创建对应的 ConfigurableEnvironment 实例，Servlet WEB 容器创建 StandardServletEnvironment 实例。

```java
private ConfigurableEnvironment getOrCreateEnvironment() {
  if (this.environment != null) {
    return this.environment;
  }
  switch (this.webApplicationType) {
  case SERVLET:
    return new StandardServletEnvironment();
  case REACTIVE:
    return new StandardReactiveWebEnvironment();
  default:
    return new StandardEnvironment();
  }
}
```

![StandardReactiveWebEnvironment](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700366833.png)

StandardServletEnvironment 继承自抽象类 AbstractEnvironment，AbstractEnvironment 构造方法中调用 customizePropertySources 方法，允许子类在构造期间操作 propertySources 实例。

```java
// AbstractEnvironment.java
private final MutablePropertySources propertySources = new MutablePropertySources();
public AbstractEnvironment() {
  customizePropertySources(this.propertySources);
}

// StandardServletEnvironment.java
protected void customizePropertySources(MutablePropertySources propertySources) {
  propertySources.addLast(new StubPropertySource(SERVLET_CONFIG_PROPERTY_SOURCE_NAME));
  propertySources.addLast(new StubPropertySource(SERVLET_CONTEXT_PROPERTY_SOURCE_NAME));
  if (JndiLocatorDelegate.isDefaultJndiEnvironmentAvailable()) {
    propertySources.addLast(new JndiPropertySource(JNDI_PROPERTY_SOURCE_NAME));
  }
  super.customizePropertySources(propertySources);
}

// StandardEnvironment.java
protected void customizePropertySources(MutablePropertySources propertySources) {
  propertySources.addLast(
      new PropertiesPropertySource(SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME, getSystemProperties()));
  propertySources.addLast(
      new SystemEnvironmentPropertySource(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, getSystemEnvironment()));
}
```

StandardServletEnvironment 中添加三种属性源：servletConfigInitParams、servletContextInitParams 和 jndiProperties。StandardEnvironment 中添加两种属性源：systemProperties（System.getProperties()） 和 systemEnvironment（System.getenv()）。即在 StandardServletEnvironment 构造时添加了五种属性源（下图没有 JNDI）。

![image-20231119121945100](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700367585.png)

ConfigurableEnvironment 实例创建后，调用 configureEnvironment 方法，configureEnvironment 中调用 configurePropertySources 和 configureProfiles 两个模板方法。

```java
protected void configureEnvironment(ConfigurableEnvironment environment, String[] args) {
  configurePropertySources(environment, args);
  // 配置激活的 profiles
  configureProfiles(environment, args);
}
```

configurePropertySources 方法中添加两种属性源：defaultProperties 和 commandLineArgs。

```java
protected void configurePropertySources(ConfigurableEnvironment environment, String[] args) {
  MutablePropertySources sources = environment.getPropertySources();
  if (this.defaultProperties != null && !this.defaultProperties.isEmpty()) {
    sources.addLast(new MapPropertySource("defaultProperties", this.defaultProperties));
  }
  if (this.addCommandLineProperties && args.length > 0) {
    String name = CommandLinePropertySource.COMMAND_LINE_PROPERTY_SOURCE_NAME;
    // ...
    sources.addFirst(new SimpleCommandLinePropertySource(args));
  }
}
```

接着发布 ApplicationEnvironmentPreparedEvent 事件，对此事件感兴趣的监听器有如下几种。

![image-20231119123409067](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700368449.png)

其中 ConfigFileApplicationListener 用于调用所有的 EnvironmentPostProcessor，LoggingApplicationListener 用于初始化日志系统。

```java
// ConfigFileApplicationListener.java
private void onApplicationEnvironmentPreparedEvent(ApplicationEnvironmentPreparedEvent event) {
  List<EnvironmentPostProcessor> postProcessors = loadPostProcessors();
  // ConfigFileApplicationListener 也实现了 EnvironmentPostProcessor
  postProcessors.add(this);
  AnnotationAwareOrderComparator.sort(postProcessors);
  for (EnvironmentPostProcessor postProcessor : postProcessors) {
    postProcessor.postProcessEnvironment(event.getEnvironment(), event.getSpringApplication());
  }
}

List<EnvironmentPostProcessor> loadPostProcessors() {
  return SpringFactoriesLoader.loadFactories(EnvironmentPostProcessor.class, getClass().getClassLoader());
}
```

实现了 EnvironmentPostProcessor 接口的类和主要逻辑有

* SystemEnvironmentPropertySourceEnvironmentPostProcessor：将 SystemEnvironmentPropertySource 包装为 OriginAwareSystemEnvironmentPropertySource 属性源，增加 OriginLookup 接口的支持。
* SpringApplicationJsonEnvironmentPostProcessor：从 `spring.application.json` 或 `SPRING_APPLICATION_JSON` 解析 JSON，向 Environment 添加 `spring.application.json` 属性源。
* CloudFoundryVcapEnvironmentPostProcessor：如果是 SpringCloud 环境，则添加 VCAP 属性源。
* ConfigFileApplicationListener：向 Environment 添加 random 属性源和配置文件属性源。

`ConfigurationPropertySources.attach(environment);` 方法用于将 MutablePropertySources 转换为 ConfigurationPropertySources，向 Environment 添加 configurationProperties 属性源。

使用 @PropertySource 添加到属性源是在 ConfigurationClassParser 类中解析的。

```java
protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)
    throws IOException {
  // Process any @PropertySource annotations
  for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
      sourceClass.getMetadata(), PropertySources.class,
      org.springframework.context.annotation.PropertySource.class)) {
    if (this.environment instanceof ConfigurableEnvironment) {
      processPropertySource(propertySource);
    }
  }
}
```

## Spring Profile

### 解析入口

当 ConfigFileApplicationListener 收到 ApplicationEnvironmentPreparedEvent 事件后，调用 addPropertySources 方法解析配置文件，即 Spring Profile 的处理入口。

```java
protected void addPropertySources(ConfigurableEnvironment environment, ResourceLoader resourceLoader) {
  RandomValuePropertySource.addToEnvironment(environment);
  new Loader(environment, resourceLoader).load();
}
```

Loader 的 load 方法如下。

```java
// 保存从外部配置或配置文件中解析的 profile
private Deque<Profile> profiles;
// 保存已完成解析的 profile
private List<Profile> processedProfiles;
private boolean activatedProfiles;
// 保存每个 profile 中加载的配置
private Map<Profile, MutablePropertySources> loaded;
public void load() {
  this.profiles = new LinkedList<>();
  this.processedProfiles = new LinkedList<>();
  this.activatedProfiles = false;
  this.loaded = new LinkedHashMap<>();
  // 初始化 profile 信息
  initializeProfiles();
  while (!this.profiles.isEmpty()) {
    Profile profile = this.profiles.poll();
    if (profile != null && !profile.isDefaultProfile()) {
      addProfileToEnvironment(profile.getName());
    }
    load(profile, this::getPositiveProfileFilter, addToLoaded(MutablePropertySources::addLast, false));
    this.processedProfiles.add(profile);
  }
  resetEnvironmentProfiles(this.processedProfiles);
  load(null, this::getNegativeProfileFilter, addToLoaded(MutablePropertySources::addFirst, true));
  addLoadedPropertySources();
}
```

### 初始化 profile 信息

```java
private void initializeProfiles() {
  // 添加 null，用于处理 application.properties 或 application.yml 配置文件
  this.profiles.add(null);
  // 从环境变量中查找 spring.profiles.active 和 spring.profiles.include 配置项，如果存在将其加入 profiles 中
  Set<Profile> activatedViaProperty = getProfilesActivatedViaProperty();
  this.profiles.addAll(getOtherActiveProfiles(activatedViaProperty));
  // Any pre-existing active profiles set via property sources (e.g.
  // System properties) take precedence over those added in config files.
  addActiveProfiles(activatedViaProperty);
  if (this.profiles.size() == 1) { // only has null profile
    // 如果没有指定 profile，默认会添加一个名为 default 的 profile（可通过 spring.profiles.default 配置修改）
    for (String defaultProfileName : this.environment.getDefaultProfiles()) {
      Profile defaultProfile = new Profile(defaultProfileName, true);
      this.profiles.add(defaultProfile);
    }
  }
}
```

### Profile 处理逻辑

```java
private void load(Profile profile, DocumentFilterFactory filterFactory, DocumentConsumer consumer) {
  // 获取配置文件的搜索路径
  // 1. 如果指定了 spring.config.location 则从指定位置查找
  // 2. 如果指定了 spring.config.additional-location，则将指定位置和默认位置（classpath:/,classpath:/config/,file:./,file:./config/）合并查找
  getSearchLocations().forEach((location) -> {
    boolean isFolder = location.endsWith("/");
    // 是否通过 spring.config.name 指定配置文件名字，默认为 application
    Set<String> names = isFolder ? getSearchNames() : NO_SEARCH_NAMES;
    names.forEach((name) -> load(location, name, profile, filterFactory, consumer));
  });
}
```

### PropertySourceLoader

PropertiesPropertySourceLoader 用于加载 properties 和 xml 文件，YamlPropertySourceLoader 用于加载 yml 和 yaml 文件。

### load 逻辑

在上一步获取到配置文件搜索路径和名称后，load 方法主要是判断配置文件是否存在，存在即读取配置文件。

```java
private void load(PropertySourceLoader loader, String location, Profile profile, DocumentFilter filter,
    DocumentConsumer consumer) {
  try {
    // 读取 location/name.extension 文件，默认 classpath:/application.properties 等。
    Resource resource = this.resourceLoader.getResource(location);
    // 如果资源存在，则读取并解析文件。
    String name = "applicationConfig: [" + location + "]";
    List<Document> documents = loadDocuments(loader, name, resource);

    List<Document> loaded = new ArrayList<>();
    for (Document document : documents) {
      if (filter.match(document)) {
        // 将此配置内激活的 profile 添加到 profiles 集合中，待后续解析
        addActiveProfiles(document.getActiveProfiles());
        addIncludedProfiles(document.getIncludeProfiles());
        loaded.add(document);
      }
    }
    Collections.reverse(loaded);
    if (!loaded.isEmpty()) {
      // 将此配置文件存放于 loaded 中（Map<Profile, MutablePropertySources> loaded）
      loaded.forEach((document) -> consumer.accept(profile, document));
    }
  }
  catch (Exception ex) {
  }
}
```

### 将配置添加到 environment

```java
private void addLoadedPropertySources() {
  MutablePropertySources destination = this.environment.getPropertySources();
  List<MutablePropertySources> loaded = new ArrayList<>(this.loaded.values());
  Collections.reverse(loaded);
  String lastAdded = null;
  Set<String> added = new HashSet<>();
  // 遍历从 profiles 中解析出的配置信息
  for (MutablePropertySources sources : loaded) {
    // 一个 profile 可能对应多个配置文件，即 application-default.properties | application-default.yml
    for (PropertySource<?> source : sources) {
      if (added.add(source.getName())) {
        // 将属性集添加到 MutablePropertySources 中
        addLoadedPropertySource(destination, lastAdded, source);
        lastAdded = source.getName();
      }
    }
  }
}
```

![image-20231121213114617](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700573475.png)
