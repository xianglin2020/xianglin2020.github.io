---
title: Java 线程池学习
date: 2020-11-29 17:10:44
categories: [learn]
tags: [java]
description: 线程池是一种基于池化管理线程的工具，经常出现在多线程服务器中。既避免处理任务时创建销毁线程的开销，也避免了线程数据膨胀导致过分调度的问题。
featuredImagePreview: https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202102/Executors.png
---

# 线程池学习

## 线程池概念

### 使用线程池的优势

线程池是一种基于池化管理线程的工具，经常出现在多线程服务器中。既避免处理任务时创建销毁线程的开销，也避免了线程数据膨胀导致过分调度的问题。使用线程池的好处有：

* 降低资源消耗：通过池化技术重复利用已创建的线程，降低线程创建和销毁造成的损耗。
* 提高响应速度：任务到达时，无需等待线程创建即可执行。
* 提高线程的可管理性：线程是稀缺资源，无限制的创建既会销毁系统资源，也会导致调度失衡，减低系统稳定性。使用线程池可以进行统一的分配、调优和监控。
* 提供更多更强大的功能：基于线程池提供的钩子函数，可以扩展线程池的行为。

## 线程池的设计与实现

### 继承关系及类作用介绍

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202011/195501.png" alt="Executor" style="zoom:50%;" />

#### `Executor`

`Executor` 接口提供了一种思想：将任务提交和任务执行分离。用户无需关注线程的创建、调度和任务执行过程，只需要将表示任务的 `Runnable` 对象提交到执行器 `Executor` 中，由 `Executor` 完成线程的调配和任务的执行部分。

#### `ExecutorService`

`ExecutorService` 接口增加了一些能力：

1. 扩展执行任务的能力：补充可以为一个或一批异步任务生成 `Future` 的方法。
2. 提供了管理线程池的方法：停止线程池、查看任务数量等。

#### `AbstractExecutorService`

`AbstractExecutorService` 是上层抽象类，将任务执行的流程串联起来，保证下层的实现只需要关注一个执行任务的方法即可。

#### `ThreadPoolExecutor`

`ThreadPoolExecutor` 一方面维护自身的生命周期，另一方面管理线程和任务。

线程池在内部构建了一个生产者消费者模型，将线程和任务两者解耦，并不直接关联，从而良好的缓冲任务，复用线程。线程池的运行主要分为两部分：任务管理和线程管理。

任务管理充当生产者角色，当任务提交后，线程会判断该任务的后续流转：（1）直接申请线程执行该任务；（2）缓冲到队列中等待线程执行；（3）拒绝该任务。

线程管理部分是消费者，它们被维护在线程池内，根据任务请求进行线程的分配，当线程执行完任务后则继续获取新的任务去执行，最终当线程获取不到任务时就会被回收。

### 线程池生命周期管理

线程池内部使用一个变量来维护两个值：运行状态（runState）和线程数量（workCount）：

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

高 3 位保存 runState，低 29 位保存 workCount，用一个变量存储两个值，可以避免使用锁维护两者的一致。

线程池的运行状态有 5 种，分别是：

|     状态     | 含义                                                         |
| :----------: | ------------------------------------------------------------ |
|  `RUNNING`   | 能接收新任务，也能处理排队任务                               |
|  `SHUTWODN`  | 关闭状态，不接受新任务，但处理排队任务                       |
|    `STOP`    | 不接受新任务，不处理排队任务，并中断正在进行的任务           |
|  `TIDYING`   | 所有任务都已经终止，workerCount 为零，并调用`terminated`钩子方法 |
| `TERMINATED` | `terminated()`方法执行完成后进入该状态                       |

其生命周期转换关系如下：

![线程池生命周期转换](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202102/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E8%BD%AC%E6%8D%A2.png)

### 任务执行机制

所有任务的调度都是由 `execute` 方法完成的，这部分完成的工作是：检查现在线程池的运行状态、运行线程数、运行策略，决定接下来执行的流程，是直接申请线程执行，或是缓冲到队列中执行，亦或是直接拒绝该任务。其执行流程如下：

1. 首先检测线程池运行状态，如果不是 `RUNNING` ，则直接拒绝，线程池要保证在 `RUNNING` 状态下执行任务；
2. 如果 `workerCount < corePoolSize`，则创建并启动一个线程来执行新提交的任务；
3. 如果 `workerCount >= corePoolSize`，且线程池内的阻塞队列未满，则将该任务添加到阻塞队列中；
4. 如果 `workCount >= corePoolSize && workerCount < maximumPoolSize`，且线程池内的阻塞队列已满，则创建一个新线程来执行新提交的任务；
5. 如果 `workerCount >= maximumPoolSize`，并且线程池内的阻塞队列已满，则根据拒绝策略来处理该任务，默认的处理方式是直抛出异常。

其执行流程如图所示：

![线程执行流程图](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202102/%E7%BA%BF%E7%A8%8B%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

任务拒绝模块是线程池的保护部分，线程池有一个最大的容量，当线程池的任务缓存队列已满，并且线程池中的线程数达到 `maximumPoolSize` 时，就需要拒绝掉该任务，采取任务拒绝策略，保护线程池。

拒绝策略是一个接口，设计如下：

```java
public interface RejectedExecutionHandler {
    
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}

```

用户可以通过实现这个接口定制拒绝策略，也可以选择 JDK 提供的四种拒绝策略：

* `AbortPolicy`：抛出`RejectedExecutionException`

  ```java
  public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    throw new RejectedExecutionException("Task " + r.toString() +
                                         " rejected from " +
                                         e.toString());
  }
  ```

* `DiscardPolicy`：什么也不做，直接忽略

  ```java
  public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
  }
  ```

* `DiscardOldestPolicy`：丢弃执行队列中最老的任务，尝试为当前提交的任务腾出位置

  ```java
  public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    if (!e.isShutdown()) {
      e.getQueue().poll();
      e.execute(r);
    }
  }
  ```

* `CallerRunsPolicy`：直接由提交任务者执行这个任务

  ```java
  public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    if (!e.isShutdown()) {
      r.run();
    }
  }
  ```
  

线程池的默认拒绝策略是：

```java
  private static final RejectedExecutionHandler defaultHandler = new AbortPolicy();
```

### Worker 线程管理

线程池为了掌握线程的状态并维护线程的生命周期，设计了线程池内部工作线程 Worker，部分代码如下：

```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
    // worker 持有的线程
    final Thread thread;
    // 初始化的任务
    Runnable firstTask;
    
    public void run() {
       runWorker(this);
    }
}
```

Worker 线程实现了`Runnable` 接口，并持有一个线程 `thread`，一个初始化任务 `firstTask`。`thread` 是在调用构造方式时通过 `ThreadFactory` 来创建的线程，可以用来执行任务；`firstTask` 用来保存传入的第一个任务。

线程池需要管理线程的生命周期，需要在线程长时间不允许的时候进行回收。线程池使用 `HashSet` 持有线程的引用，这样可以通过添加引用、移除引用的操作来控制线程的生命周期。

Worker 通过继承 AQS，使用 AQS 来实现独占锁这个功能。

1. `lock` 方法一旦获取了独占锁，表示当前线程正在执行任务中。
2. 如果正在执行任务，则不应该中断线程，如果该线程现在不是独占锁状态，也就是空闲状态，说明它没有在处理任务，这时可以对该线程进行中断。
3. 线程池在执行 `shutdown` 方法或 `tryTerminate` 方法时会调用 `interruptIdleWorkers` 方法来中断空闲线程，`interruptIdleWorkers` 方法会使用 `tryLock` 方法来判断线程池中的线程是否是空闲状态，如果线程是空闲状态则可以安全回收。

增加线程是通过 `addWorker` 方法完成的。`addWorker` 方法有两个参数：`firstTask`、`core` 。`firstTask` 参数用于指定新增的线程执行的第一个任务，该参数可以为空；`core` 参数为 `true` 时表示新增线程时会判断当前活动的线程数是否少于 `corePoolSize` ，`false` 表示新增线程前需要判断当前活动线程数是否少于 `maximumPoolSize` ，其增加流程如下：

![线程池增加Worker步骤](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202102/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%A2%9E%E5%8A%A0Worker%E6%AD%A5%E9%AA%A4.png)

Worker 类中的 `run()` 方法调用了 `runWorker()` 方法来执行任务，其执行过程如下：

1. 循环通过 `getTask()` 方法获取任务；
2. 如果线程池正在停止，那么保证当前线程是中断状态，否则保证当前线程不是中断状态；
3. 执行任务。
4. 获取不到任务时，执行 `processWorkerExit()` 方法主动销毁线程。

![runWorker步骤](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202102/runWorker%E6%AD%A5%E9%AA%A4.png)

线程池中线程的销毁依赖 JVM 自动的回收，线程池做的工作就是根据当前线程池状维护一定数量的线程引用，防止这部分线程被 JVM 回收，当线程池决定哪些线程需要回收时，只需要将其引用消除即可。Worker 被创建出来后，就会不断循环获取任务执行，核心线程可以无限等待获取任务，非核心线程要限时获取任务。当 Worker 无法获取到任务时，循环会结束，Worker 会主动消除自身在线程池内的引用。

```java
final void runWorker(Worker w) {
    try {
        while (task != null || (task = getTask()) != null) {
            // 执行任务
        }
    } finally {
        // 获取不到任务时，主动回收自己
        processWorkerExit(w, completedAbruptly);
    }
}
```

```java
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        completedTaskCount += w.completedTasks;
        // 将线程引用移除线程池
        workers.remove(w);
    } finally {
        mainLock.unlock();
    }
}
```



## 创建和停止线程池

### 线程池的构造器的参数

| 参数名            | 类型                       | 含义                              |
| ----------------- | -------------------------- | --------------------------------- |
| `corePoolSize`    | `int`                      | 核心线程数                        |
| `maximumPoolSize` | `int`                      | 最大线程数                        |
| `keepAliveTime`   | `long`                     | 保持存活时间                      |
| `workQueue`       | `BlockingQueue`            | 任务存储队列                      |
| `threadFactory`   | `ThreadFactory`            | 使用 `threadFactory` 创建新的线程 |
| `handler`         | `RejectedExecutionHandler` | 拒绝策略                          |

* `corePoolSize`：指的是核心线程数。

* `maxPoolSize`：指的是最大线程数。

	1. 如果线程数小于`corePoolSize`，创建一个新的线程来运行新任务；
	2. 如果线程数等于或大于`corePoolSize`但小于`maximumPoolSize`，则将任务添加到任务队列中；
	3. 如果队列已满，且线程数小于`maximumPoolSize`，则创建新线程；
	4. 如果队列已满，且线程数达到`maximumPoolSize`，则调用`handler`执行拒绝策略。
	
	<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202011/174715.jpg" alt="线程池创建线程步骤" style="zoom:50%;" />
	
* `keepAliveTime`：如果线程池当前的线程数多于`corePoolSize`，那么多余的线程闲置超过指定时间会被终止。

* `threadFactory`：创建线程的工厂。

  <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202011/175254.png" alt="DefaultThreadFactory" style="zoom:50%;" />

  默认线程池创建的线程的属性为：

  1. 属于同一个`ThreadGroup`
  2. 线程池和线程的名称按序号递增
  3. 不是 daemon 线程
  4. 线程的优先级为`NORM_PRIORITY`

  ```java
  private final String namePrefix = "pool-" +
                            poolNumber.getAndIncrement() +
                           "-thread-";
  
  
  public Thread newThread(Runnable r) {
    Thread t = new Thread(group, r,
                          namePrefix + threadNumber.getAndIncrement(),
                          0);
    if (t.isDaemon())
      t.setDaemon(false);
    if (t.getPriority() != Thread.NORM_PRIORITY)
      t.setPriority(Thread.NORM_PRIORITY);
    return t;
  }
  ```

* `workQueue`：任务队列

  1. 直接交接：`SynchronousQueue`
  2. 无界队列：`LinkedBlockingQueue`
  3. 有界队列：`ArrayBlockingQueue`

### Executors创建线程池

| 方法名                    | 功能|缺点|
| ------------------------- | -------------------------------------------------- | -------------------------------------------------- |
| `newFixedThreadPool`      | 创建固定大小的线程池                               |容易造成大量内存占用，导致 OOM|
| `newSingleThreadExecutor` | 创建只有一个线程的线程池                           |当请求堆积的时候，占用大量内存|
| `newCachedThreadPool`     | 创建一个不设线程上限的线程池，任何任务都将立即执行 |创建数量非常多的线程，导致 OOM|

1. `newFixedThreadPool`

   ```java
   public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory){
     return new ThreadPoolExecutor(nThreads, nThreads,
                                   0L, TimeUnit.MILLISECONDS,
                                   new LinkedBlockingQueue<Runnable>(),
                                   threadFactory);
   }
   ```

2. `newSingleThreadExecutor`

   ```java
   public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
     return new FinalizableDelegatedExecutorService
       (new ThreadPoolExecutor(1, 1,
                               0L, TimeUnit.MILLISECONDS,
                               new LinkedBlockingQueue<Runnable>(),
                               threadFactory));
   }
   ```

3. `newCachedThreadPool`

   ```java
   public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
     return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                   60L, TimeUnit.SECONDS,
                                   new SynchronousQueue<Runnable>(),
                                   threadFactory);
   }
   ```

   

### 线程池的线程数量如何设定

* CPU 密集型（加密、计算 hash）：最佳线程数为CPU 核心数的 1-2 倍左右；
* 耗时 IO 型（读写数据库、文件、网络）：最大线程数一般会大于 CPU 核心数很多倍。
* 线程数 = CPU 核心数 * (1 + 平均等待时间 / 平均工作时间)

### 停止线程池的方法

* `shutdown`
  1. 将线程池的状态置为`SHUTDOWN`
  2. 调用此方法后，不允许继续提交任务，即调用指定的拒绝策略拒绝任务
  3. 所有在调用此方法前提交的任务都会被执行
  4. 所有任务被执行完毕，`ExecutorService`才会真正关闭
* `shutdownNow`
  1. 将线程池的状态置为`STOP`
  2. 使用中断操作尝试停止运行中的任务
  3. 返回未尚未执行的任务

* `isShutdown`：线程池是否关闭
* `isTerminated`：判断线程池关闭后所有的任务是否都执行完了
* `awaitTermination`：阻塞，直到出现以下情况
  1. `shutdown`调用后所有任务执行完成
  2. 超时返回
  3. 当前线程中断

### 线程池钩子`beforeExecute`

实现一个可以暂停的线程池

```java
public class PauseableThreadPool extends ThreadPoolExecutor {
    public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    private boolean isPaused;
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();


    @Override
    protected void beforeExecute(Thread t, Runnable r) {
        lock.lock();
        try {
            while (isPaused) {
                condition.await();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }


    private void pause() {
        lock.lock();
        try {
            isPaused = true;
        } finally {
            lock.unlock();
        }
    }

    private void resume() {
        lock.lock();
        try {
            isPaused = false;
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }
    
}
```

