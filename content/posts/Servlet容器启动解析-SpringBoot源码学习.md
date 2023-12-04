---
title: "Servlet容器启动解析 SpringBoot源码学习"
date: 2023-11-26T12:46:13+08:00
categories:
  - SpringBoot源码学习
tags:
  - SpringBoot
summary: "Servlet 容器启动流程解析、WEB 容器工厂类加载解析、WEB 容器个性化配置解析。"
description: "Servlet 容器启动流程解析、WEB 容器工厂类加载解析、WEB 容器个性化配置解析。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## 容器启动全局流程解析

### 启动前准备

在 SpringApplication 构造方法中根据 classpath 下是否存在特定类来决定容器类型（SERVLET、REACTIVE、NONE），并赋值给 webApplicationType 属性。

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
  this.webApplicationType = WebApplicationType.deduceFromClasspath();
}

static WebApplicationType deduceFromClasspath() {
  // 存在 reactive.DispatcherHandler 且不存在 servlet.DispatcherServlet 和 org.glassfish.jersey.servlet.ServletContainer （Oracle WEB Server）时，视为 REACTIVE 环境
  if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
      && !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
    return WebApplicationType.REACTIVE;
  }
  for (String className : SERVLET_INDICATOR_CLASSES) {
    // 如果不存在 javax.servlet.Servlet 或 ConfigurableWebApplicationContext 视为 NONE （非 WEB）环境
    if (!ClassUtils.isPresent(className, null)) {
      return WebApplicationType.NONE;
    }
  }
  // 默认视为 SERVLET 环境
  return WebApplicationType.SERVLET;
}
```

在 SpringApplication 的 createApplicationContext 方法中根据 webApplicationType 属性创建对应的上下文。

```java
protected ConfigurableApplicationContext createApplicationContext() {
  Class<?> contextClass = this.applicationContextClass;
  if (contextClass == null) {
    try {
      switch (this.webApplicationType) {
      case SERVLET:
        // AnnotationConfigServletWebServerApplicationContext
        contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
        break;
      case REACTIVE:
        // AnnotationConfigReactiveWebServerApplicationContext
        contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
        break;
      default:
        // AnnotationConfigApplicationContext
        contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
      }
    }
    catch (ClassNotFoundException ex) {
  }
  return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
}
```

### WEBServer 创建

在 refresh 方法中调用由 ServletWebServerApplicationContext 实现的 onRefresh 子方法，通过 createWebServer 方法完成 WEBServer 的创建。

```java
// ServletWebServerApplicationContext.java
protected void onRefresh() {
  super.onRefresh();
  try {
    createWebServer();
  }
  catch (Throwable ex) {
    throw new ApplicationContextException("Unable to start web server", ex);
  }
}

private void createWebServer() {
  WebServer webServer = this.webServer;
  ServletContext servletContext = getServletContext();
  if (webServer == null && servletContext == null) {
    // 获取用于创建 webServer 的工厂类，一般情况是：TomcatServletWebServerFactory
    ServletWebServerFactory factory = getWebServerFactory();
    // 创建 Tomcat WEBServer
    this.webServer = factory.getWebServer(getSelfInitializer());
  }
  else if (servletContext != null) {
    try {
      getSelfInitializer().onStartup(servletContext);
    }
    catch (ServletException ex) {
      throw new ApplicationContextException("Cannot initialize servlet context", ex);
    }
  }
  initPropertySources();
}

// TomcatServletWebServerFactory.java
public WebServer getWebServer(ServletContextInitializer... initializers) {
  Tomcat tomcat = new Tomcat();
  // ...
  return getTomcatWebServer(tomcat);
}
```

### WEBServer 容器启动

在 refresh 方法中调用由 ServletWebServerApplicationContext 实现的 finishRefresh 子方法，通过 startWebServer 方法完成 WEBServer 启动。

```java
protected void finishRefresh() {
  super.finishRefresh();
  WebServer webServer = startWebServer();
  if (webServer != null) {
    publishEvent(new ServletWebServerInitializedEvent(webServer, this));
  }
}

private WebServer startWebServer() {
  WebServer webServer = this.webServer;
  if (webServer != null) {
    // 启动 webServer
    webServer.start();
  }
  return webServer;
}
```

## WEB 容器工厂类加载解析

WEBServer 创建时通过 getWebServerFactory 方法获取工厂类，默认是 TomcatServletWebServerFactory 对象，那这个对象是何时加载到容器中的呢？

### EnableAutoConfiguration

@SpringBootApplication 注解上有 @EnableAutoConfiguration 注解，并通过 @Import 引入了 AutoConfigurationImportSelector，用于处理自动配置。

```java
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
	Class<?>[] exclude() default {};
	String[] excludeName() default {};
}

public class AutoConfigurationImportSelector implements DeferredImportSelector {
  	// 返回需要导入的配置类
}
```

需要自动配置的类定义在 `spring-boot-autoconfigure/META-INF/spring.factories` 中，其中就有 ServletWebServerFactoryAutoConfiguration 。

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
```

### ServletWebServerFactoryAutoConfiguration

ServletWebServerFactoryAutoConfiguration 用于处理 Servlet WEB 容器的配置类，它引入了 EmbeddedTomcat 配置类，并配置了 ServletWebServerFactoryCustomizer 对象用于配置 ServletWebServerFactory。

```java
@Configuration
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
		ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
		ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
		ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
public class ServletWebServerFactoryAutoConfiguration {

	@Bean
	public ServletWebServerFactoryCustomizer servletWebServerFactoryCustomizer(ServerProperties serverProperties) {
		return new ServletWebServerFactoryCustomizer(serverProperties);
	}

	@Bean
	@ConditionalOnClass(name = "org.apache.catalina.startup.Tomcat")
	public TomcatServletWebServerFactoryCustomizer tomcatServletWebServerFactoryCustomizer(
			ServerProperties serverProperties) {
		return new TomcatServletWebServerFactoryCustomizer(serverProperties);
	}
}
```

因为当前是 Servlet 环境，且引入了 `org.springframework.boot:spring-boot-starter-tomcat` 依赖，故此处会创建 TomcatServletWebServerFactory 对象注册到 BeanFactory 中，用于后续创建 Tomcat WEBServer。

```java
@Configuration
class ServletWebServerFactoryConfiguration {

	@Configuration
	@ConditionalOnClass({ Servlet.class, Tomcat.class, UpgradeProtocol.class })
	@ConditionalOnMissingBean(value = ServletWebServerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedTomcat {
		@Bean
		public TomcatServletWebServerFactory tomcatServletWebServerFactory() {
			return new TomcatServletWebServerFactory();
		}
	}
}
```

## WEB 容器配置解析

### ServerProperties

以 Tomcat 容器为例，调整如下两个配置，WEBServer 属性配置保存在 ServerProperties 中。

```properties
server.port=8082
server.tomcat.max-threads=201
```

```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {
    private Integer port;
    private final Tomcat tomcat = new Tomcat();
    public static class Tomcat {
        private int maxThreads = 200;
    }
    // ...
}
```

### WebServerFactoryCustomizer

在 ServletWebServerFactoryAutoConfiguration 中向容器中注册了 ServletWebServerFactoryCustomizer 和 TomcatServletWebServerFactoryCustomizer 对象用于调整 WebServer 配置。

```java
@FunctionalInterface
public interface WebServerFactoryCustomizer<T extends WebServerFactory> {
	void customize(T factory);
}
```

WebServerFactoryCustomizer 实现类在 WebServerFactoryCustomizerBeanPostProcessor 的 postProcessBeforeInitialization 方法中调用。

```java
public class WebServerFactoryCustomizerBeanPostProcessor implements BeanPostProcessor, BeanFactoryAware {

	private ListableBeanFactory beanFactory;

	private List<WebServerFactoryCustomizer<?>> customizers;

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		if (bean instanceof WebServerFactory) {
			postProcessBeforeInitialization((WebServerFactory) bean);
		}
		return bean;
	}

	@SuppressWarnings("unchecked")
	private void postProcessBeforeInitialization(WebServerFactory webServerFactory) {
    	// getCustomizers()：获取所有 WebServerFactoryCustomizer 实现类对象
    	LambdaSafe.callbacks(WebServerFactoryCustomizer.class, getCustomizers(), webServerFactory)
        .withLogger(WebServerFactoryCustomizerBeanPostProcessor.class)
        // 调用 customize 方法
        .invoke((customizer) -> customizer.customize(webServerFactory));
	}
}
```

### ServletWebServerFactoryCustomizer

以 ServletWebServerFactoryCustomizer 为例，介绍 `server.port` 属性是如何设置到 WebServer 中的。

ServletWebServerFactoryAutoConfiguration 自动配置类会注册 ServletWebServerFactoryCustomizer Bean 实例，ServerProperties 对象中保存了配置项 `server.port` 的值。

![image-20231203192924320](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/12/1701602964.png)

WebServer 容器创建时会调用 getWebServerFactory 方法获取已注册的 TomcatServletWebServerFactory Bean 实例，Bean 创建时通过 WebServerFactoryCustomizerBeanPostProcessor 后置处理器调用 WebServerFactoryCustomizer 实现，对 TomcatServletWebServerFactory 做配置。

![image-20231203192735355](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/12/1701602855.png)

![image-20231203193003592](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/12/1701603003.png)

ServletWebServerFactoryCustomizer 在 customize 方法中将 `server.port` 属性设置到 ConfigurableServletWebServerFactory 中，其它的 Customizer 也基本是同样的处理方式。

![image-20231203193042578](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/12/1701603042.png)

最终调用 TomcatServletWebServerFactory 的 getWebServer 创建 WebServer 时，将保存在自身的 `server.port`属性设置到 Connector 中。

![image-20231203193304079](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/12/1701603184.png)

![image-20231203193343879](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/12/1701603224.png)
