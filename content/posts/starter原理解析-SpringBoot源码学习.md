---
title: "Starter原理解析 SpringBoot源码学习"
date: 2023-12-04T19:56:02+08:00
categories:
  - SpringBoot源码学习
tags:
  - SpringBoot
summary: "SpringBoot conditional 注解、starter 原理及动手搭建 starter。"
description: "SpringBoot conditional 注解、starter 原理及动手搭建 starter。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## Conditional 注解

### 简单使用

以 @ConditionalOnProperty 注解为例，新建组件 A，当配置中存在 `store.xianglin.condition` 配置项时，向容器中注册 A。

```java
@ConditionalOnProperty("store.xianglin.condition")
@Component
public class A {
}

@RunWith(SpringRunner.class)
@SpringBootTest
public class TestA {
    @Resource
    ApplicationContext context;

    @Test
    public void testA() {
        var bean = context.getBean(A.class);
        Assert.assertNotNull(bean);
    }
}
```

```properties
store.xianglin.condition=a
```

当 `store.xianglin.condition` 配置有值时，测试通过。

### 原理分析

@ConditionalOnProperty 注解上有 @Conditional(OnPropertyCondition.class)。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.TYPE, ElementType.METHOD })
@Documented
@Conditional(OnPropertyCondition.class)
public @interface ConditionalOnProperty {
	String[] value() default {};
}

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
  // 配置的所有Condition 都满足时向容器中注册组件
	Class<? extends Condition>[] value();
}
```

Condition 的定义是。

```java
@FunctionalInterface
public interface Condition {
  // 满足条件时返回 true
	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);

}
```

### 自定义 Conditional 注解

创建类 MyCondition 实现 Condition 接口，在 matches 方法中判断配置中存在相应配置项值时返回 true。

```java
public class MyCondition implements Condition {
  
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        var value = (String[]) metadata.getAnnotationAttributes("store.xianglin.sb2.condi.MyConditionAnnotation").get("value");
        for (String property : value) {
            if (!StringUtils.isEmpty(context.getEnvironment().getProperty(property))) {
                return true;
            }
        }
        return false;
    }
}
```

创建注解 MyConditionAnnotation 并使用 @Conditional 注解标注。

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(MyCondition.class)
public @interface MyConditionAnnotation {
    String[] value() default {};
}
```



