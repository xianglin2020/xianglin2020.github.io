---
title: "异常报告器解析 SpringBoot源码学习"
date: 2023-11-21T21:59:41+08:00
categories:
  - SpringBoot源码学习
tags:
  - SpringBoot
summary: "SpringBoot 异常报告器类介绍、异常处理流程分析、案例分析。"
description: "SpringBoot 异常报告器类介绍、异常处理流程分析、案例分析。"
author: ["XiangLin"]
cover:
  image: ""
  caption: ""
  alt: ""
  relative: false
---

## 核心类解析

![SpringBootExceptionReporter](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/2023/11/1700655979.png)

FailureAnalyzer：用于分析故障并提供可显示给用户的诊断信息 FailureAnalysis。

```java
@FunctionalInterface
public interface FailureAnalyzer {
	FailureAnalysis analyze(Throwable failure);
}
```

FailureAnalysisReporter：用于向用户报告 FailureAnalyzer 的分析结果 FailureAnalysis。

```java
@FunctionalInterface
public interface FailureAnalysisReporter {
	void report(FailureAnalysis analysis);
}
```

SpringBootExceptionReporter：用于自定义报告 SpringApplication 启动异常的回调接口。

```java
@FunctionalInterface
public interface SpringBootExceptionReporter {
	boolean reportException(Throwable failure);
}
```

FailureAnalyzers：用于触发 FailureAnalyzer 和 FailureAnalysisReporter。

```java
final class FailureAnalyzers implements SpringBootExceptionReporter {
  @Override
	public boolean reportException(Throwable failure) {
    // 调用 FailureAnalyzer 的 analyzer 方法，得到 FailureAnalysis
		FailureAnalysis analysis = analyze(failure, this.analyzers);
    // 调用 FailureAnalysisReporter 的 report 方法，报告分析结果
		return report(analysis, this.classLoader);
	}
}
```

## 异常处理流程解析

SpringBoot 启动异常处理流程的入口在 `org.springframework.boot.SpringApplication#run(java.lang.String...)` 方法中。

```java
public ConfigurableApplicationContext run(String... args) {
  try {
  }
  catch (Throwable ex) {
    handleRunFailure(context, ex, exceptionReporters, listeners);
    throw new IllegalStateException(ex);
  }
  try {
    listeners.running(context);
  }
  catch (Throwable ex) {
    handleRunFailure(context, ex, exceptionReporters, null);
    throw new IllegalStateException(ex);
  }
  return context;
}
```

当启动过程中出现异常或调用 SpringApplicationRunListeners 出现异常时会调用 handleRunFailure 方法，完成异常处理流程。

```java
private void handleRunFailure(ConfigurableApplicationContext context, Throwable exception,
    Collection<SpringBootExceptionReporter> exceptionReporters, SpringApplicationRunListeners listeners) {
  try {
    try {
      // 处理 exitCode
      handleExitCode(context, exception);
      if (listeners != null) {
        // 发送 ApplicationFailedEvent 事件
        listeners.failed(context, exception);
      }
    }
    finally {
      // 调用 FailureAnalyzers 的 reportException 方法报告异常
      reportFailure(exceptionReporters, exception);
      if (context != null) {
        // 关闭应用程序上下文
        context.close();
      }
    }
  }
  catch (Exception ex) {
    logger.warn("Unable to close ApplicationContext", ex);
  }
  // 重新抛出此异常
  ReflectionUtils.rethrowRuntimeException(exception);
}
```

关闭应用程序上下文的逻辑主要有。

```java
public void close() {
  synchronized (this.startupShutdownMonitor) {
    doClose();
    if (this.shutdownHook != null) {
      try {
        // 移除 ShutdownHooks，因为 doClose 方法已经明确关闭了 context，不需要借助 ShutdownHooks 来完成这些操作
        Runtime.getRuntime().removeShutdownHook(this.shutdownHook);
      }
      catch (IllegalStateException ex) {
      }
    }
  }
}

protected void doClose() {
  // 更改应用上下文状态
  if (this.active.get() && this.closed.compareAndSet(false, true)) {
    // 销毁单例 Bean
    destroyBeans();
    // 将 context 中的 beanFactory 置空
    closeBeanFactory();
    // 留给子类实现：WEB 环境下关闭 WEB 容器
    onClose();
    this.active.set(false);
  }
}
```

