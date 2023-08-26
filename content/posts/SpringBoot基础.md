---
title: SpringBoot 基础
date: 2020-10-19 22:18:41
categories: [learn]
tags: [spring boot]
---

# Spring Boot

## SpringBoot 过滤器使用

### 两种注册过滤器的方式

#### 使用`FilterRegistrationBean`注册过滤器

1. 实现过滤器接口`javax.servlet.Filter`，重写`doFilter`方法

   ```java
   public class AdminFilter implements Filter {
       @Resource
       private UserService userService;
   
       @Override
       public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
           HttpServletRequest request = (HttpServletRequest) servletRequest;
           // ...
           filterChain.doFilter(servletRequest, servletResponse);
       }
   }
   ```

2. 新增配置类，注册过滤器

   ```java
   @Configuration
   public class FilterConfig {
       @Bean
       public Filter filter() {
           return new AdminFilter();
       }
   
       @Bean
       public FilterRegistrationBean<Filter> filterRegistrationBean() {
           FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
           filterRegistrationBean.setFilter(filter());
           filterRegistrationBean.addUrlPatterns("/admin/*");
           filterRegistrationBean.setName("bFilter");
           return filterRegistrationBean;
       }
   }
   ```

#### 使用`@WebFilter`和`@ServletComponentScan`注解注册过滤器

1. 实现过滤器接口`javax.servlet.Filter`，重写`doFilter`方法，并使用`WebFilter`注解指定过滤器配置

   ```java
   @WebFilter(urlPatterns = "/admin/*",filterName = "aFilter")
   public class AnnotationAdminFilter implements Filter {
   
       @Override
       public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
           log.info("使用@WebFilter注解配置过滤器！");
       }
   }
   ```

2. 在启动类上添加`@ServletComponentScan`注解

   ```java
   @SpringBootApplication
   @ServletComponentScan
   public class MallApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(MallApplication.class, args);
       }
   
   }
   
   ```

### 过滤器执行顺序

#### 使用`@WebFilter`

* 使用注解方式的过滤顺序是以类名的自然顺序排序的

#### 使用`FilterRegistrationBean`注册过滤器

1. 默认情况下，以注册器在`FilterConfig`中配置的顺序。
2. 可以使用`RegistrationBean#setOrder`指定过滤器的执行顺序，数字小的优先级高。

#### 同时使用

*测试结果是，`FilterRegistrationBean`注册的整体优先于使用`@WebFilter`的方式，具体顺序待查看源码分析*

## SpringBoot 多环境配置

* [ ] TODO

