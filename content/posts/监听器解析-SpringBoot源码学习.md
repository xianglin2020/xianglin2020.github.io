---
title: "监听器解析 SpringBoot源码学习"
date: 2023-11-12T19:58:52+08:00
categories:
  - SpringBoot源码学习
tags:
  - SpringBoot
summary: "SpringBoot 监听器模式介绍、系统监听器介绍、监听事件触发机制和自定义监听器。"
description: "SpringBoot 监听器模式介绍、系统监听器介绍、监听事件触发机制和自定义监听器。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## 监听器模式介绍

![image-20231114205039935](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1699966240.png)

### 编码实现监听器模式

实现一个天气预报的监听器，当天气为下雨或下雪时，通知监听者。

定义一个表示天气事件的抽象类 WeatherEvent，有下雨事件 RainEvent 和下雪事件 SnowEvent。

```java
public abstract class WeatherEvent {
    public abstract String getWeather();
}

public class RainEvent extends WeatherEvent {
    @Override
    public String getWeather() {
        return "rain";
    }
}

public class SnowEvent extends WeatherEvent {
    @Override
    public String getWeather() {
        return "snow";
    }
}
```

定义事件监听器，分别监听下雪事件和下雨事件。

```java
public interface WeatherListener {
    void onWeatherEvent(WeatherEvent event);
}

public class RainListener implements WeatherListener{
    @Override
    public void onWeatherEvent(WeatherEvent event) {
        if (event instanceof RainEvent){
            System.out.println("hello " + event.getWeather());
        }
    }
}

public class SnowListener implements WeatherListener{
    @Override
    public void onWeatherEvent(WeatherEvent event) {
        if (event instanceof SnowEvent){
            System.out.println("hello " + event.getWeather());
        }
    }
}
```

定义事件广播器，用于广播事件和管理事件监听器。

```java
public interface EventMulticaster {
    void multicastEvent(WeatherEvent event);
    void addListener(WeatherListener weatherListener);
    void removeListener(WeatherListener weatherListener);
}

public abstract class AbstractEventMulter implements EventMulticaster {
    private List<WeatherListener> listenerList;
    @Override
    public void multicastEvent(WeatherEvent event) {
        doStart();
        listenerList.forEach(i -> i.onWeatherEvent(event));
        doEnd();
    }
    abstract void doStart();
    abstract void doEnd();

    @Override
    public void addListener(WeatherListener weatherListener) {
      	if (listenerList == null) {
            listenerList = new ArrayList<>();
        }
        listenerList.add(weatherListener);
    }
    @Override
    public void removeListener(WeatherListener weatherListener) {
        listenerList.remove(weatherListener);
    }
}

public class WeatherEventMulticaster extends AbstractEventMulter {
    @Override
    void doStart() {
        System.out.println("begin broadcast weather event");
    }

    @Override
    void doEnd() {
        System.out.println("end broadcast weather event");
    }
}
```

编写测试程序（触发机制）测试天气事件及监听器。

```java
public void testEventMulticast() {
    var eventMulticaster = new WeatherEventMulticaster();
    var weatherListener = new RainListener();
    eventMulticaster.addListener(weatherListener);
    eventMulticaster.addListener(new SnowListener());

    eventMulticaster.multicastEvent(new SnowEvent());
    eventMulticaster.multicastEvent(new RainEvent());

    eventMulticaster.removeListener(weatherListener);
    eventMulticaster.multicastEvent(new SnowEvent());
    eventMulticaster.multicastEvent(new RainEvent());
}
```

### 监听器模式的基本要素

* 监听器
* 广播器
* 事件
* 触发机制

## SpringBoot 监听器实现

Spring 系统事件监听器由接口 ApplicationListener 定义。

```java
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    // 监听到事件后的回调方法
    void onApplicationEvent(E event);
}
```

Spring 系统事件广播器由接口 ApplicationEventMulticaster 定义。

```java
// 事件广播器用于管理 ApplicationListener 和向它们广播事件
public interface ApplicationEventMulticaster {
    // 管理 ApplicationListener 的方法：新增、删除
    void addApplicationListener(ApplicationListener<?> listener);
    void removeApplicationListener(ApplicationListener<?> listener);
    void removeAllListeners();
    // 向监听器广播事件
    void multicastEvent(ApplicationEvent event);
}
```

Spring 系统事件由抽象类 ApplicationEvent 定义，主要系统事件有。

```java
public abstract class ApplicationEvent extends EventObject {
    // 事件发生的时间
    private final long timestamp;
    // 触发事件的对象（EventObject）
    protected transient Object source;
}
```

![ApplicationEvent](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1699792667.png)

### SpringBoot 系统事件发送顺序

SpringBoot 启动时系统事件和事件触发顺序由 SpringApplicationRunListener 接口定义。

```java
// 实现类：EventPublishingRunListener
public interface SpringApplicationRunListener {
    // ApplicationStartingEvent：SpringApplication run 方法执行时触发
    void starting();
    // ApplicationEnvironmentPreparedEvent：环境准备好 ApplicationContext 还未创建时
    void environmentPrepared(ConfigurableEnvironment environment);
    // ApplicationContextInitializedEvent：
    void contextPrepared(ConfigurableApplicationContext context);
    // ApplicationPreparedEvent：ApplicationContext 已加载未刷新
    void contextLoaded(ConfigurableApplicationContext context);
    // ApplicationStartedEvent
    void started(ConfigurableApplicationContext context);
    // ApplicationReadyEvent
    void running(ConfigurableApplicationContext context);
    // ApplicationFailedEvent：应用发送错误时
    void failed(ConfigurableApplicationContext context, Throwable exception);
}
```

### 监听器注册

系统监听器和系统初始化器的注册流程基本一致，即在 SpringApplication 的构造方法中通过 SpringFactoriesLoader 加载 ApplicationListener 的实现类，具体流程参照[初始化器解析-springboot源码学习/#springfactoriesloader-介绍](https://blog.xianglin.store/posts/%E5%88%9D%E5%A7%8B%E5%8C%96%E5%99%A8%E8%A7%A3%E6%9E%90-springboot%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0/#springfactoriesloader-%E4%BB%8B%E7%BB%8D)。

## 监听器事件触发机制

### 系统事件触发机制

以 ApplicationStartingEvent 事件为例，最初是在 ConfigurableApplicationContext 的 run 方法中调用 starting 方法触发。

```java
public ConfigurableApplicationContext run(String... args) {
  // ...
  SpringApplicationRunListeners listeners = getRunListeners(args);
  listeners.starting();
  // ...
  return context;
}
```

SpringApplicationRunListeners 类是 SpringApplicationRunListener 的集合，对每一个 RunListener 调用 starting  方法。

```java
class SpringApplicationRunListeners {
    private final List<SpringApplicationRunListener> listeners;
    // ...
    public void starting() {
      for (SpringApplicationRunListener listener : this.listeners) {
        listener.starting();
      }
    }
}
```

SpringApplicationRunListener 最终通过 SimpleApplicationEventMulticaster 事件广播器将 ApplicationStartingEvent 事件广播给事件监听器。

```java
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {
    private final SpringApplication application;
    private final String[] args;
    private final SimpleApplicationEventMulticaster initialMulticaster;

    public EventPublishingRunListener(SpringApplication application, String[] args) {
      this.application = application;
      this.args = args;
      this.initialMulticaster = new SimpleApplicationEventMulticaster();
      for (ApplicationListener<?> listener : application.getListeners()) {
        // 将所有监听器注册到广播器中
        this.initialMulticaster.addApplicationListener(listener);
      }
    }
    // ...
    @Override
    public void starting() {
      this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(this.application, this.args));
    }
}
```

### 改造监听器模式

参照 SpringApplicationRunListener 改造上述天气预报的监听器，引入 WeatherRunListener ，定义 snow 和 rain 方法。通过 Spring 管理广播器和监听器实现自动注入。

```java
@Component
public class WeatherRunListener {
    @Resource
    private WeatherEventMulticaster eventMulticaster;

    public void snow(){
        eventMulticaster.multicastEvent(new SnowEvent());
    }

    public void rain(){
        eventMulticaster.multicastEvent(new RainEvent());
    }
}

@Component
public class WeatherEventMulticaster extends AbstractEventMulter {
  // ...
}

@Component
public class RainListener implements WeatherListener{
  // ...
}

// 测试
@RunWith(SpringRunner.class)
@SpringBootTest
public class WeatherRunListenerTest {
    @Resource
    private WeatherRunListener weatherRunListener;

    @Test
    public void testSnow() {
        weatherRunListener.snow();
    }

    @Test
    public void testRain() {
        weatherRunListener.rain();
    }
}
```

### multicastEvent 方法解析

上面已经分析到 ApplicationStartingEvent 事件最终是通过 SimpleApplicationEventMulticaster 的 multicastEvent 方法广播给监听器，multicastEvent 方法的主要逻辑是获取监听器，触发监听器的 onApplicationEvent 事件。

```java
@Override
public void multicastEvent(ApplicationEvent event) {
  // 获取事件的 class 包装，调用重载方法
  multicastEvent(event, resolveDefaultEventType(event));
}

@Override
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
  ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
  // 未指定线程池，返回 null
  Executor executor = getTaskExecutor();
  // getApplicationListeners 获取对当前事件感兴趣的监听器
  for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
    if (executor != null) {
      executor.execute(() -> invokeListener(listener, event));
    }
    else {
      // 调用事件监听器的 onApplicationEvent 方法：listener.onApplicationEvent(event);
      invokeListener(listener, event);
    }
  }
}
```

通过 getApplicationListeners 方法获取对当前事件感兴趣的监听器。首先使用事件源 source 和事件类型 sourceType 构造缓存 cacheKey，避免重复查找，如果 retrieverCache 缓存中不存在，则调用 retrieveApplicationListeners 方法获取对当前事件感兴趣的监听器。

```java
protected Collection<ApplicationListener<?>> getApplicationListeners(
			ApplicationEvent event, ResolvableType eventType) {

  Object source = event.getSource();
  Class<?> sourceType = (source != null ? source.getClass() : null);
  ListenerCacheKey cacheKey = new ListenerCacheKey(eventType, sourceType);

  // Quick check for existing entry on ConcurrentHashMap...
  ListenerRetriever retriever = this.retrieverCache.get(cacheKey);
  if (retriever != null) {
    return retriever.getApplicationListeners();
  }
  // ... 
  return retrieveApplicationListeners(eventType, sourceType, null)
}
```

重载的 retrieveApplicationListeners 方法才是真正获取监听器的方法。遍历所有 SpringApplication 初始化时加载的监听器（EventPublishingRunListener 构造方法中添加到 SimpleApplicationEventMulticaster 中），依次调用 supportsEvent 方法，将返回为 true 的监听器添加到目标集合中，最后对目标监听器排序返回。

```java
private Collection<ApplicationListener<?>> retrieveApplicationListeners(
			ResolvableType eventType, @Nullable Class<?> sourceType, @Nullable ListenerRetriever retriever) {

  List<ApplicationListener<?>> allListeners = new ArrayList<>();
  Set<ApplicationListener<?>> listeners;
  Set<String> listenerBeans;
  synchronized (this.retrievalMutex) {
    // this.defaultRetriever.applicationListeners：EventPublishingRunListener 构造方法中调用 initialMulticaster.addApplicationListener(listener) 方法注册
    listeners = new LinkedHashSet<>(this.defaultRetriever.applicationListeners);
    listenerBeans = new LinkedHashSet<>(this.defaultRetriever.applicationListenerBeans);
  }
  for (ApplicationListener<?> listener : listeners) {
    if (supportsEvent(listener, eventType, sourceType)) {
      if (retriever != null) {
        retriever.applicationListeners.add(listener);
      }
      allListeners.add(listener);
    }
  }
  if (!listenerBeans.isEmpty()) {
    // ...
  }
  AnnotationAwareOrderComparator.sort(allListeners);
  if (retriever != null && retriever.applicationListenerBeans.isEmpty()) {
    retriever.applicationListeners.clear();
    retriever.applicationListeners.addAll(allListeners);
  }
  return allListeners;
}
```

supportsEvent 确定给定监听器是否支持给定事件，首先将 listener 包装为 GenericApplicationListener 的实现类，然后调用 supportsEventType 方法和 supportsSourceType 方法确定给定监听器是否支持当前事件。

```java
protected boolean supportsEvent(
			ApplicationListener<?> listener, ResolvableType eventType, @Nullable Class<?> sourceType) {

GenericApplicationListener smartListener = (listener instanceof GenericApplicationListener ?
      (GenericApplicationListener) listener : new GenericApplicationListenerAdapter(listener));
  return (smartListener.supportsEventType(eventType) && smartListener.supportsSourceType(sourceType));
}
```

![GenericApplicationListener](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700060884.png)

GenericApplicationListener 和 SmartApplicationListener 都是 ApplicationListener 的子接口，增加了 supportsEventType 方法判断监听器是否支持给定的事件类型和 supportsSourceType 方法判断监听器是否支持给定的事件源类型。

如果监听器不是实现自 SmartApplicationListener 或 GenericApplicationListener 则使用 GenericApplicationListenerAdapter 适配器包装，通过 resolveDeclaredEventType 方法解析出监听器支持的事件类型。

## 自定义监听器

### 监听器定义方式

方式一、实现 ApplicationListener 接口，通过泛型参数指定监听的事件。

```java
@Order(1)
public class FirstListener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("hello first");
    }
}
```

方式二、实现 SmartApplicationListener 接口，通过 supportsEventType 方法指定监听的事件。

```java
@Order(4)
public class FourthListener implements SmartApplicationListener {
    @Override
    public boolean supportsEventType(Class<? extends ApplicationEvent> eventType) {
        return ApplicationStartedEvent.class.isAssignableFrom(eventType)
                || ApplicationPreparedEvent.class.isAssignableFrom(eventType);
    }

    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("hello fourth");
    }
}
```

### 监听器注册方式

方式一、通过 META-INF/spring.factories 文件注册监听器。

```properties
org.springframework.context.ApplicationListener=store.xianglin.sb2.listener.FirstListener,store.xianglin.sb2.listener.FourthListener
```

方式二、通过调用 `SpringApplication.addListeners()` 方法注册 SecondListener。

```java
@SpringBootApplication
@MapperScan("store.xianglin.sb2.mapper")
public class SpringBoot2Application {
    public static void main(String[] args) {
//        SpringApplication.run(SpringBoot2Application.class, args);
        var springApplication = new SpringApplication(SpringBoot2Application.class);
        springApplication.addInitializers(new SecondInitializer());
        springApplication.addListeners(new SecondListener());
        springApplication.run(args);
    }
}
```

方式三、通过配置文件注册 ThirdListener，委托给 DelegatingApplicationListener 监听器处理。

```properties
context.listener.classes=store.xianglin.sb2.listener.ThirdListener
```
