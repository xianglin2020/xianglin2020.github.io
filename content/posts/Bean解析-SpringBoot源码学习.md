---
title: "Bean解析 SpringBoot源码学习"
date: 2023-11-16T20:04:48+08:00
categories:
  - SpringBoot源码学习
tags:
  - SpringBoot
summary: "SpringBoot 中 IOC思想、Bean 的配置方式、refresh 方法解析、Bean 实例化解析。"
description: "SpringBoot 中 IOC思想、Bean 的配置方式、refresh 方法解析、Bean 实例化解析。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## Bean 的配置方式

### XML 方式配置 Bean

新建 Student 类，有 name、age、classList 三个属性，HelloService 类有 Student 属性。

```java
public class Student {
    private String name;
    private Integer age;
    private List<String> classList;

    // getter and setter
}

public class HelloService {
    private Student student;

    public Student getStudent() {
        return student;
    }

    public void setStudent(Student student) {
        this.student = student;
    }

    public String hello() {
        return student.toString();
    }
}
```

在 resources 目录下新建 Spring 配置文件 demo.xml。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <!-- student 配置 -->
  
  <bean id="helloService" class="store.xianglin.sb2.ioc.xml.HelloService">
    <property name="student" ref="student"/>
	</bean>
</beans>
```

在 test 目录下新建测试类，测试 HelloService 类的 hello 方法。

```java
@SpringBootTest
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = "classpath:ioc/demo.xml")
public class HelloServiceTest {
    @Resource
    private HelloService helloService;
    @Test
    public void testHello() {
        System.out.println(helloService.hello());
    }
}
```

方式一、通过属性注入。

```xml
<bean id="student" class="store.xianglin.sb2.ioc.xml.Student">
    <property name="name" value="zhangsan"/>
    <property name="age" value="13"/>
    <property name="classList">
        <list>
            <value>math</value>
            <value>english</value>
        </list>
    </property>
</bean>
```

方式二、通过构造器注入。在 Student 类中创建两个参数的构造器，在 demo.xml 中使用 constructor-arg 标签注入。

```java
public Student(String name, Integer age) {
    this.name = name;
    this.age = age;
}
```

```xml
<bean id="student" class="store.xianglin.sb2.ioc.xml.Student">
    <constructor-arg index="0" value="zhangsan"/>
    <constructor-arg index="1" value="13"/>
    <property name="classList">
        <list>
            <value>math</value>
            <value>english</value>
        </list>
    </property>
</bean>
```

新建抽象类 Animal，定义抽象方法 getName，定义子类 Cat 和 Dog。新建工厂类 AnimalFactory，定义静态方法 getAnimal、实例方法 getAnimalInstance，根据入参返回 Animal 的实例。

```java
public abstract class Animal {
    abstract String getName();
}

public class Cat extends Animal {
    @Override
    String getName() {
        return "cat";
    }
}

public class Dog extends Animal{
    @Override
    String getName() {
        return "dog";
    }
}

public class AnimalFactory {
    public static Animal getAnimal(String type) {
        if ("dog".equals(type)) {
            return new Dog();
        } else if ("cat".equals(type)) {
            return new Cat();
        }
        return null;
    }

    public Animal getAnimalInstance(String type) {
        if ("dog".equals(type)) {
            return new Dog();
        } else if ("cat".equals(type)) {
            return new Cat();
        }
        return null;
    }
}
```

方式三、静态工厂方法注入。

```xml
<bean id="dog" class="store.xianglin.sb2.ioc.xml.AnimalFactory" factory-method="getAnimal">
    <constructor-arg value="dog"/>
</bean>
<bean id="cat" class="store.xianglin.sb2.ioc.xml.AnimalFactory" factory-method="getAnimal">
    <constructor-arg value="cat"/>
</bean>
```

方式四、实例工厂方法注入。

```xml
<bean id="animalFactory" class="store.xianglin.sb2.ioc.xml.AnimalFactory"/>
<bean id="dog1" factory-bean="animalFactory" factory-method="getAnimalInstance">
    <constructor-arg value="dog"/>
</bean>
<bean id="cat1" factory-bean="animalFactory" factory-method="getAnimalInstance">
    <constructor-arg value="cat"/>
</bean>
```

### 注解方式配置 Bean

方式一、使用 @Component 注解，直接在 HelloService 类上添加 @Component，将其注入到容器中。

```java
@Component
public class HelloService {
    @Resource
    private Animal animal;

    public Animal getAnimal() {
        return animal;
    }

    public void setAnimal(Animal animal) {
        this.animal = animal;
    }

    public String hello() {
        return animal.getName();
    }
}
```

方式二、使用 @Bean 注解，在 @Configuration 注解标注的配置类中通过 @Bean 注解注入 Bean。

```java
@Configuration
public class BeanConfiguration {
    @Bean("dog")
    public Animal getDog() {
        return new Dog();
    }
}
```

方式三、实现 FactoryBean 接口。

```java
@Component
public class MyCat implements FactoryBean<Animal> {
    @Override
    public Animal getObject() {
        return new Cat();
    }

    @Override
    public Class<?> getObjectType() {
        return Animal.class;
    }
}
```

方式四、实现 BeanDefinitionRegistryPostProcessor 接口，在 postProcessBeanDefinitionRegistry 方法中添加 BeanDefinition。

```java
@Component
public class MyBeanRegister implements BeanDefinitionRegistryPostProcessor, InstantiationAwareBeanPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        // 注入 BeanDefinition
        var definition = new RootBeanDefinition();
        definition.setBeanClass(Monkey.class);
        registry.registerBeanDefinition("monkey", definition);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        // 为 BeanDefinition 注入属性
        var monkey = beanFactory.getBeanDefinition("monkey");
        monkey.getPropertyValues().addPropertyValue("property", "value from postProcessBeanFactory");
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        // 自定义创建 Bean，不会执行后续的 doCreateBean 方法以及属性注入等
        if (beanClass.equals(Monkey.class)) {
            return new Monkey();
       	}
        return null;
    }
}
```

方式五、实现 ImportBeanDefinitionRegistrar，通过 @Import 导入。

```java
public class MyBeanImport implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        var definition = new RootBeanDefinition();
        definition.setBeanClass(Bird.class);
        registry.registerBeanDefinition("bird", definition);
    }
}

@Import(MyBeanImport.class)
@Configuration
public class BeanConfiguration {
}
```

## refresh 解析

refresh 方法定义在 AbstractApplicationContext，用于加载或刷新配置，调用了如下 13 个方法。

```java
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
    // Prepare this context for refreshing.
    prepareRefresh();
    // Tell the subclass to refresh the internal bean factory.
    ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
    // Prepare the bean factory for use in this context.
    prepareBeanFactory(beanFactory);
    try {
      // Allows post-processing of the bean factory in context subclasses.
      postProcessBeanFactory(beanFactory);
      // Invoke factory processors registered as beans in the context.
      invokeBeanFactoryPostProcessors(beanFactory);
      // Register bean processors that intercept bean creation.
      registerBeanPostProcessors(beanFactory);
      // Initialize message source for this context.
      initMessageSource();
      // Initialize event multicaster for this context.
      initApplicationEventMulticaster();
      // Initialize other special beans in specific context subclasses.
      onRefresh();
      // Check for listener beans and register them.
      registerListeners();
      // Instantiate all remaining (non-lazy-init) singletons.
      finishBeanFactoryInitialization(beanFactory);
      // Last step: publish corresponding event.
      finishRefresh();
    } catch (BeansException ex) {
      if (logger.isWarnEnabled()) {
        logger.warn("Exception encountered during context initialization - " +
            "cancelling refresh attempt: " + ex);
      }
      // Destroy already created singletons to avoid dangling resources.
      destroyBeans();
      // Reset 'active' flag.
      cancelRefresh(ex);
      // Propagate exception to caller.
      throw ex;
    }
    finally {
      // Reset common introspection caches in Spring's core, since we
      // might not ever need metadata for singleton beans anymore...
      resetCommonCaches();
    }
  }
}
```

### prepareRefresh

设置容器状态：启动时间、active 标记，初始化属性设置，检查必备属性是否存在。

```java
protected void prepareRefresh() {
  // Switch to active.
  this.startupDate = System.currentTimeMillis();
  this.closed.set(false);
  this.active.set(true);

  // Initialize any placeholder property sources in the context environment.
  initPropertySources();

  // Validate that all properties marked as required are resolvable:
  // see ConfigurablePropertyResolver#setRequiredProperties
  getEnvironment().validateRequiredProperties();
}
```

必备属性可以通过在 ApplicationContextInitializer 的 initialize 方法中调用 `environment.setRequiredProperties()` 方法添加，缺少必备属性时抛出 MissingRequiredPropertiesException 异常，SpringApplication 初始化失败。

### obtainFreshBeanFactory

通知子类刷新内部的 beanFactory，设置 beanFactory 序列化 ID，获取 beanFactory（DefaultListableBeanFactory）。

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
  refreshBeanFactory();
  return getBeanFactory();
}

protected final void refreshBeanFactory() throws IllegalStateException {
  if (!this.refreshed.compareAndSet(false, true)) {
    throw new IllegalStateException(
        "GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
  }
  // 设置序列化 ID，可用于通过此 ID 反序列化 BeanFactory 对象
  this.beanFactory.setSerializationId(getId());
}
```

### prepareBeanFactory

设置 beanFactory 的属性：ClassLoader、PropertyEditorRegistrar、BeanPostProcessor，设置忽略的自动装配接口，注册默认环境相关的 Bean。

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
  // Tell the internal bean factory to use the context's class loader etc.
  beanFactory.setBeanClassLoader(getClassLoader());
  beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
  beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

  // Configure the bean factory with context callbacks.
  beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
  beanFactory.ignoreDependencyInterface(EnvironmentAware.class);

  // BeanFactory interface not registered as resolvable type in a plain factory.
  // MessageSource registered (and found for autowiring) as a bean.
  beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);

  // Register early post-processor for detecting inner beans as ApplicationListeners.
  beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

  // Register default environment beans.
  if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
    beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
  }
}
```

### postProcessBeanFactory

子类重写以在 beanFactory 完成创建后做进一步设置，WEB 环境中注册 ServletContextAwareProcessor，注册 WEB 环境特有的 scope：request、session、application。

```java
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
  beanFactory.addBeanPostProcessor(new WebApplicationContextServletContextAwareProcessor(this));
  beanFactory.ignoreDependencyInterface(ServletContextAware.class);
  registerWebApplicationScopes();
}
```

### invokeBeanFactoryPostProcessors

按顺序实例化并调用所有已注册的 BeanDefinitionRegistryPostProcessor 和 BeanFactoryPostProcessor，上述注解配置方式四定义的 MyBeanRegister 在这里被调用，此方法主要作用有：

* 调用 BeanDefinitionRegistryPostProcessor 实现向容器内添加 Bean 的定义。
* 调用 BeanFactoryPostProcessor 实现向容器内 Bean 的定义添加属性。

```java
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
  PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
}
```

### registerBeanPostProcessors

按顺序实例化并注册所有 BeanPostProcessor ，但不调用。

### initMessageSource

初始化国际化相关属性。

### initApplicationEventMulticaster

初始化事件广播器。

### onRefresh

在 WEB 环境中调用 createWebServer 方法创建 WEB 容器。

```java
// ServletWebServerApplicationContext
protected void onRefresh() {
  super.onRefresh();
  try {
    createWebServer();
  }
  catch (Throwable ex) {
    throw new ApplicationContextException("Unable to start web server", ex);
  }
}
```

### registerListeners

添加容器内事件监听器至事件广播器中，广播早期事件。

### finishBeanFactoryInitialization

初始化所有剩余的单例 Bean。

### finishRefresh

初始化生命周期处理器，调用生命周期处理器的 onRefresh 方法，发布 ContextRefreshedEvent 事件。

## Bean 实例化解析

### BeanDefinition

一个对象在 Spring 中使用 BeanDefinition 来描述，RootBeanDefinition 是其常见实现，可以通过操作 BeanDefinition 来完成 Bean 实例化和属性注入。

![RootBeanDefinition](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700270150.png)

### 自定义创建 Bean

逻辑见 `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])`，在 createBean 方法中调用 resolveBeforeInstantiation 方法，随后调用 applyBeanPostProcessorsBeforeInstantiation 方法，最终调用 InstantiationAwareBeanPostProcessor 的 postProcessBeforeInstantiation 方法，该方法在实例化之前调用，如果返回 Bean 对象，则不再执行 Bean 的默认实例化逻辑（doCreateBean 方法以及属性注入等）。

![自定义创建 Bean](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700273192.png)

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {
  
  try {
    // Bean 实例化之前调用，如果返回 Bean 对象，不执行后续方法
    Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
    if (bean != null) {
      return bean;
    }
  }
  catch (Throwable ex) {
    throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
        "BeanPostProcessor before instantiation of bean failed", ex);
  }
  // 调用默认实例化方法
  Object beanInstance = doCreateBean(beanName, mbdToUse, args);
}

protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd){
  Object bean = null;
  bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
  if (bean != null) {
    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
  }
  return bean;
}

protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
  for (BeanPostProcessor bp : getBeanPostProcessors()) {
    if (bp instanceof InstantiationAwareBeanPostProcessor) {
      // 实现 InstantiationAwareBeanPostProcessor 自定义创建 Bean
      InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
      Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
      if (result != null) {
        return result;
      }
    }
  }
  return null;
}
```

### Bean 实例化流程

单例 Bean 实例化入口为 `org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons` 方法，主要的方法调用如图。

![自定义创建 Bean.drawio](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700279017.png)

