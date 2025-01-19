---
title: "配置类解析 SpringBoot源码学习"
date: 2023-11-23T20:01:17+08:00
categories:
  - SpringBoot源码学习
tags:
  - SpringBoot
summary: "SpringBoot 配置类解析流程、执行逻辑解析、核心方法解析。"
description: "SpringBoot 配置类解析流程、执行逻辑解析、核心方法解析。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## 全局流程解析

### 解析入口

AbstractApplicationContext 的 refresh 方法中调用 invokeBeanFactoryPostProcessors 子方法，最终调用 ConfigurationClassPostProcessor 的 postProcessBeanDefinitionRegistry 方法。

```java
public void refresh() throws BeansException, IllegalStateException {
  synchronized (this.startupShutdownMonitor) {
      invokeBeanFactoryPostProcessors(beanFactory);
  }
}
```

### postProcessBeanDefinitionRegistry 逻辑

```java
public class ConfigurationClassPostProcessor implements BeanDefinitionRegistryPostProcessor,
		PriorityOrdered, ResourceLoaderAware, BeanClassLoaderAware, EnvironmentAware {
  public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    // 获取当前 registry 的唯一 ID：registryId
    int registryId = System.identityHashCode(registry);
    // 通过 registryId 判断当前 registry 是否已处理过
    if (this.registriesPostProcessed.contains(registryId)) {
      throw new IllegalStateException(
          "postProcessBeanDefinitionRegistry already called on this post-processor against " + registry);
    }
    if (this.factoriesPostProcessed.contains(registryId)) {
      throw new IllegalStateException(
          "postProcessBeanFactory already called on this post-processor against " + registry);
    }
    // 将当前 registryId 添加到已处理集合中
    this.registriesPostProcessed.add(registryId);
    // 处理配置类
    processConfigBeanDefinitions(registry);
  }
}
```

### processConfigBeanDefinitions 逻辑

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
  List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
  String[] candidateNames = registry.getBeanDefinitionNames();
  // 遍历 BeanDefinition
  for (String beanName : candidateNames) {
    BeanDefinition beanDef = registry.getBeanDefinition(beanName);
    // 判断是否处理过：通过 configurationClass Bean 属性是否为 full 或者 lite
    if (ConfigurationClassUtils.isFullConfigurationClass(beanDef) ||
        ConfigurationClassUtils.isLiteConfigurationClass(beanDef)) {
    }
    // 检查当前 BeanDefinition 是否为配置类，如果是增加 configurationClass 属性，并添加到 configCandidates 中
    else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
      configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
    }
  }
  // Return immediately if no @Configuration classes were found
  if (configCandidates.isEmpty()) {
    return;
  }
  // Sort by previously determined @Order value, if applicable
  configCandidates.sort((bd1, bd2) -> {
    int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
    int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
    return Integer.compare(i1, i2);
  });
  // ...

  // Parse each @Configuration class
  ConfigurationClassParser parser = new ConfigurationClassParser(
      this.metadataReaderFactory, this.problemReporter, this.environment,
      this.resourceLoader, this.componentScanBeanNameGenerator, registry);

  // 遍历 configCandidates 集合进行解析处理
  Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
  Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
  do {
    parser.parse(candidates);
    parser.validate();
    // ...
  }
  while (!candidates.isEmpty());

  // 注册 importRegistry
  // Register the ImportRegistry as a bean in order to support ImportAware @Configuration classes
  if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
    sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
  }
}
```

## 执行逻辑解析

### 执行入口

通过 ConfigurationClassParser 的 parse 方法解析 candidates 配置类集合，直到没有新的配置类添加到此集合中。

```java
  do {
    // 最终调用 doProcessConfigurationClass 方法
    parser.parse(candidates);
    // 验证：配置类不能为 final，@Bean 修饰的方法能被重写（未标记为 static、final 或 private）
    parser.validate();
    // ...
    // 读取 BeanMethod 注册 BeanDefinition
    this.reader.loadBeanDefinitions(configClasses);
    alreadyParsed.addAll(configClasses);

    candidates.clear();
    // 判断是否引入新的 BeanDefinition
    if (registry.getBeanDefinitionCount() > candidateNames.length) {
      // newCandidateNames：执行 loadBeanDefinitions 后最新的 BeanDefinition
      String[] newCandidateNames = registry.getBeanDefinitionNames();
      // candidateNames：上一次（第一次）的 BeanDefinition
      Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
      Set<String> alreadyParsedClasses = new HashSet<>();
      for (ConfigurationClass configurationClass : alreadyParsed) {
        alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
      }
      for (String candidateName : newCandidateNames) {
        if (!oldCandidateNames.contains(candidateName)) {
          BeanDefinition bd = registry.getBeanDefinition(candidateName);
          if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
              !alreadyParsedClasses.contains(bd.getBeanClassName())) {
            // 如果有新注册的配置类，添加到 candidates 中，继续在下一次循环中解析
            candidates.add(new BeanDefinitionHolder(bd, candidateName));
          }
        }
      }
      candidateNames = newCandidateNames;
    }
  }
  while (!candidates.isEmpty());
```

### doProcessConfigurationClass 逻辑

ConfigurationClassParser 的 parse 方法最终调用 doProcessConfigurationClass 方法完成配置类解析。

```java
SourceClass sourceClass = asSourceClass(configClass);
  do {
    // 如果 sourceClass 不为 null，代表父类也是需要处理的配置类
    sourceClass = doProcessConfigurationClass(configClass, sourceClass);
  }
while (sourceClass != null);

protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)
    throws IOException {

  if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
    // 递归处理内部类
    processMemberClasses(configClass, sourceClass);
  }

  // 处理 @PropertySource 属性源
  for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
      sourceClass.getMetadata(), PropertySources.class,
      org.springframework.context.annotation.PropertySource.class)) {
  }

  // 处理 @ComponentScan 注解：将 ComponentScan 指定的类添加到容器中，如果有配置类调用 parse 方法立即处理
  Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
      sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);

  // 处理 @Import 注解
  processImports(configClass, sourceClass, getImports(sourceClass), true);

  // 处理 @ImportResource 注解，解析 XML 配置
  AnnotationAttributes importResource =
      AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);

  // 处理 @Bean 注解的方法
  Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
  for (MethodMetadata methodMetadata : beanMethods) {
    configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
  }

  // 处理接口定义的默认方法，如果默认方法有 @Bean 注解，将其解析为 BeanMethod 添加到 beanMethods 集合中
  processInterfaces(configClass, sourceClass);

  // 如果父类也是未处理的配置类，则返回父类继续下一次处理
  if (sourceClass.getMetadata().hasSuperClass()) {
    String superclass = sourceClass.getMetadata().getSuperClassName();
    if (superclass != null && !superclass.startsWith("java") &&
        !this.knownSuperclasses.containsKey(superclass)) {
      this.knownSuperclasses.put(superclass, configClass);
      // Superclass found, return its annotation metadata and recurse
      return sourceClass.getSuperClass();
    }
  }

  // 如果没有父类，则完成处理
  return null;
}
```

## 核心方法解析

定义如下类用于后续演示。

```java
package store.xianglin.sb2.config.scan1;

@Component
public class ComponentClass1 {
}

package store.xianglin.sb2.config.scan2;
public class ComponentClass2 {
}

package store.xianglin.sb2.config;

@Configuration
public class MyConfiguration {
    @Bean
    public ComponentClass1 componentMyConfiguration() {
        return new ComponentClass1();
    }
}
```

### 内部类处理

在 MyConfiguration 中定义内部类，并添加 @Configuration 注解。

```java
@Configuration
public class MyConfiguration {
    @Bean
    public ComponentClass1 componentMyConfiguration() {
        return new ComponentClass1();
    }
  
    @Configuration
    static class ConfigurationA {
        @Bean
        public ComponentClass1 componentConfigurationA() {
            return new ComponentClass1();
        }
    }

    static class LiteConfiguration {
        @Bean
        public ComponentClass1 componentLiteConfiguration() {
            return new ComponentClass1();
        }
    }
}
```

解析内部类的入口方法是 `org.springframework.context.annotation.ConfigurationClassParser#processMemberClasses`。

```java
if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
  // Recursively process any member (nested) classes first
  processMemberClasses(configClass, sourceClass);
}

private void processMemberClasses(ConfigurationClass configClass, SourceClass sourceClass) throws IOException {
  // 获取 MyConfiguration 的内部类遍历
  Collection<SourceClass> memberClasses = sourceClass.getMemberClasses();
  if (!memberClasses.isEmpty()) {
    List<SourceClass> candidates = new ArrayList<>(memberClasses.size());
    for (SourceClass memberClass : memberClasses) {
      // 如果内部类是配置类，添加到 candidates 集合待处理
      if (ConfigurationClassUtils.isConfigurationCandidate(memberClass.getMetadata()) &&
          !memberClass.getMetadata().getClassName().equals(configClass.getMetadata().getClassName())) {
        candidates.add(memberClass);
      }
    }
    OrderComparator.sort(candidates);
    for (SourceClass candidate : candidates) {
      // 处理内部配置类
      processConfigurationClass(candidate.asConfigClass(configClass));
    }
  }
}
```

LiteConfiguration 虽然没有添加 @Configuration 注解，但内部声明了带有 @Bean 注解的方法，因此会被认为是 `lite configuration`，也会被处理。

![image-20231126110701895](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700968022.png)

### PropertySource 处理

在 MyConfiguration 配置类上添加 @PropertySource 注解，指定属性文件。

```java
@Configuration
@PropertySource({"classpath:demo.properties"})
public class MyConfiguration  {

}
```

解析 PropertySource 的入口方法是 `org.springframework.context.annotation.ConfigurationClassParser#processPropertySource`。

```java
// Process any @PropertySource annotations
for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
    sourceClass.getMetadata(), PropertySources.class,
    org.springframework.context.annotation.PropertySource.class)) {
  if (this.environment instanceof ConfigurableEnvironment) {
    processPropertySource(propertySource);
  }
}

private void processPropertySource(AnnotationAttributes propertySource) throws IOException {
  // 从注解中获取 locations/value 和其它配置项
  String[] locations = propertySource.getStringArray("value");
  for (String location : locations) {
    // 替换占位符，加载资源
    String resolvedLocation = this.environment.resolveRequiredPlaceholders(location);
    Resource resource = this.resourceLoader.getResource(resolvedLocation);
    // 添加到 environment 中
    addPropertySource(factory.createPropertySource(name, new EncodedResource(resource, encoding)));
  }
}

private void addPropertySource(PropertySource<?> propertySource) {
  String name = propertySource.getName();
  MutablePropertySources propertySources = ((ConfigurableEnvironment) this.environment).getPropertySources();

  // 存在同名属性源（name 属性指定）时，合并配置项
  if (this.propertySourceNames.contains(name)) {
    // We've already added a version, we need to extend it
  }

  // 将加载的 propertySource 添加到 environment 中
  if (this.propertySourceNames.isEmpty()) {
    propertySources.addLast(propertySource);
  }
  else {
    String firstProcessed = this.propertySourceNames.get(this.propertySourceNames.size() - 1);
    propertySources.addBefore(firstProcessed, propertySource);
  }
  this.propertySourceNames.add(name);
}
```

![image-20231126112627008](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700969187.png)

### ComponentScan 处理

在 MyConfiguration 配置类上添加 @ComponentScan 注解，指定需要扫描的包路径。

```java
@Configuration
@ComponentScan(basePackages = "store.xianglin.sb2.config.scan1", basePackageClasses = ComponentClass2.class)
public class MyConfiguration {

}
```

解析 ComponentScan 的入口方法是 `org.springframework.context.annotation.ComponentScanAnnotationParser#parse`。

```java
Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
    sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
// 解析 componentScan 注解并扫描注解配置的包路径
for (AnnotationAttributes componentScan : componentScans) {
  Set<BeanDefinitionHolder> scannedBeanDefinitions =
      this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
  
  for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
    // 如果扫描得到的类中存在配置类，则立即解析配置类
    if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
      parse(bdCand.getBeanClassName(), holder.getBeanName());
    }
  }
}

public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, final String declaringClass) {
  // 从注解中获取配置项
  Set<String> basePackages = new LinkedHashSet<>();
  String[] basePackagesArray = componentScan.getStringArray("basePackages");
  for (String pkg : basePackagesArray) {
    String[] tokenized = StringUtils.tokenizeToStringArray(this.environment.resolvePlaceholders(pkg),
        ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);
    Collections.addAll(basePackages, tokenized);
  }
  // 从 basePackageClasses 定义的类中获取包路径
  for (Class<?> clazz : componentScan.getClassArray("basePackageClasses")) {
    basePackages.add(ClassUtils.getPackageName(clazz));
  }
  // 如果未配置扫描路径，则将配置类所在路径添加到扫描路径中
  if (basePackages.isEmpty()) {
    basePackages.add(ClassUtils.getPackageName(declaringClass));
  }
  // 扫描配置的包路径，返回 BeanDefinitionHolder
  return scanner.doScan(StringUtils.toStringArray(basePackages));
}
```

### Import 处理

@Import 注解允许导入 ImportSelector 和 ImportBeanDefinitionRegistrar 的实现。

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        System.out.println("MyImportSelector");
        return new String[]{"store.xianglin.sb2.config.scan1.ComponentClass1"};
    }
}

public class MyDeferredImportSelector implements DeferredImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        System.out.println("MyDeferredImportSelector");
        return new String[]{"store.xianglin.sb2.config.scan2.ComponentClass2"};
    }
}

public class MyBeanImport implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        var definition = new RootBeanDefinition();
        definition.setBeanClass(Bird.class);
        registry.registerBeanDefinition("bird", definition);
    }
}

@Configuration
@Import(value = {MyImportSelector.class, MyDeferredImportSelector.class})
public class MyConfiguration {

}
```

解析 Import 的入口方法是 `org.springframework.context.annotation.ConfigurationClassParser#processImports`。

```java
// getImports 方法解析 @Import 注解配置的实现类
processImports(configClass, sourceClass, getImports(sourceClass), true);

private void processImports(ConfigurationClass configClass, ConfigurationClassParser.SourceClass currentSourceClass,
                            Collection<ConfigurationClassParser.SourceClass> importCandidates, boolean checkForCircularImports) {

  // 遍历 @Import 指定的配置类
  for (ConfigurationClassParser.SourceClass candidate : importCandidates) {
      if (candidate.isAssignable(ImportSelector.class)) {
          Class<?> candidateClass = candidate.loadClass();
          ImportSelector selector = BeanUtils.instantiateClass(candidateClass, ImportSelector.class);
          // DeferredImportSelector 实现由 deferredImportSelectorHandler 延迟处理
          if (selector instanceof DeferredImportSelector) {
              this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector) selector);
          } else {
              // 执行 selectImports 方法得到配置的类名称
              String[] importClassNames = selector.selectImports(currentSourceClass.getMetadata());
              Collection<ConfigurationClassParser.SourceClass> importSourceClasses = asSourceClasses(importClassNames);
              // 递归调用 processImports 方法，进入 else 分支，作为配置类解析
              processImports(configClass, currentSourceClass, importSourceClasses, false);
          }
      } else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {
          // Candidate class is an ImportBeanDefinitionRegistrar ->
          // delegate to it to register additional bean definitions
          Class<?> candidateClass = candidate.loadClass();
          ImportBeanDefinitionRegistrar registrar =
                  BeanUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class);
          // 处理 ImportBeanDefinitionRegistrar 实现类中注册的 BeanDefinition
          configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
      } else {
          // 处理普通的配置类
          processConfigurationClass(candidate.asConfigClass(configClass));
      }
  }
}
```

### ImportResource 处理

使用 `@ImportResource(locations = "classpath:config/config.xml")` 方式引入 XML 配置文件，解析方法如下。

```java
AnnotationAttributes importResource =
    AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);
if (importResource != null) {
  // 获取注解中的配置项：XML 文件路径
  String[] resources = importResource.getStringArray("locations");
  Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
  for (String resource : resources) {
    // 替换占位符，加载资源
    String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
    configClass.addImportedResource(resolvedResource, readerClass);
  }
}

// 将 resolvedResource 保存到 importedResources 中，后续在 loadBeanDefinitions 方法中解析
public void addImportedResource(String importedResource, Class<? extends BeanDefinitionReader> readerClass) {
  this.importedResources.put(importedResource, readerClass);
}
```

### BeanMethod 处理

```java
// 获取配置类中所有添加 @Bean 注解的方法，包装为 BeanMethod
Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
for (MethodMetadata methodMetadata : beanMethods) {
  configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
}

// 将 BeanMethod 添加到 beanMethods 中，后续在 loadBeanDefinitions 方法中解析
public void addBeanMethod(BeanMethod method) {
  this.beanMethods.add(method);
}
```

### 默认方法处理

在接口 InterfaceConfiguration 中定义默认方法，MyConfiguration 配置类实现此接口。

```java
public interface InterfaceConfiguration {
    @Bean
    default ComponentClass1 componentInterfaceConfiguration() {
        return new ComponentClass1();
    }
}

@Configuration
public class MyConfigurationimplements InterfaceConfiguration {

}
```

```java
// 处理接口中的默认方法
processInterfaces(configClass, sourceClass);

private void processInterfaces(ConfigurationClass configClass, SourceClass sourceClass) throws IOException {
  for (SourceClass ifc : sourceClass.getInterfaces()) {
    // 获取所有添加 @Bean 注解的方法
    Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(ifc);
    for (MethodMetadata methodMetadata : beanMethods) {
      if (!methodMetadata.isAbstract()) {
        // 将 BeanMethod 添加到 beanMethods 中，后续在 loadBeanDefinitions 方法中解析
        configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
      }
    }
    // 递归处理父接口
    processInterfaces(configClass, ifc);
  }
}
```

![image-20231126122025976](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700972426.png)

### 父类处理

定义父类 SuperConfiguration，MyConfiguration 配置类继承此父类。

```java
public abstract class SuperConfiguration {
    @Bean
    public ComponentClass1 componentSuperConfiguration() {
        return new ComponentClass1();
    }
}

@Configuration
public class MyConfiguration extends SuperConfiguration{

}
```

```java
if (sourceClass.getMetadata().hasSuperClass()) {
  String superclass = sourceClass.getMetadata().getSuperClassName();
  // 父类不为 null、不是 java 开头的包、没有处理过
  if (superclass != null && !superclass.startsWith("java") &&
      !this.knownSuperclasses.containsKey(superclass)) {
    this.knownSuperclasses.put(superclass, configClass);
    // 返回父类，再次调用 doProcessConfigurationClass 方法处理
    return sourceClass.getSuperClass();
  }
}
```

![image-20231126122356785](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700972637.png)

![image-20231126122704373](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700972824.png)

