---
title: "Java多线程"
date: 2021-05-05
draft: false
tags: ["Java", "面向菜鸟编程"]
slug: "rookie-multi-thread"
---

## 概述

## 创建线程

### 继承Thread类

### 实现Runnable接口

### 实现Callable接口

### 线程池创建

## 锁

### 公平锁

### 非公平锁

### 可重入锁

### 不可重入锁

### 自旋锁

### 独占锁

### 共享锁


## volatile

## synchronized

## ReentrantLock

## 并发集合不安全

### List

### Set

### Map


## 线程池
线程池做的工作主要是控制运行的线程的数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量超出数量的线程排队等候，等其它线程执行完毕，再从队列中取出任务来执行。

它的主要特点为：线程复用，控制最大并发数，管理线程。

优点：
- 降低资源消耗。通过重复利用己创建的线程降低线程创建和销毁造成的消耗。
- 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

### 常用方式
通过`Executors`线程池工具类来使用：
- `Executors.newSingleThreadExecutor()`：创建只有一个线程的线程池
- `Executors.newFixedThreadPool(int)`：创建固定线程的线程池
- `Executors.newCachedThreadPool()`：创建一个可缓存的线程池，线程数量随着处理业务数量变化

底层代码都是用`ThreadPoolExecutor`创建的。
#### SingleThreadExecutor
- 使用`Executors.newSingleThreadExecutor()`创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序执行。
- `newSingleThreadExecutor` 将 `corePoolSize` 和 `maximumPoolSize` 都设置为1，它使用的 `LinkedBlockingQueue`。

源代码
```
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

代码演示
```
public class MainTest {
    public static void main(String[] args) {
        ExecutorService executor1 = null;
        try {
            executor1 = Executors.newSingleThreadExecutor();
            for (int i = 1; i <= 10; i++) {
                executor1.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "执行了");
                });
            }
        } finally {
            executor1.shutdown();
        }
    }
}
```

#### FixedThreadPool
- 使用`Executors.newFixedThreadPool(int)`创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待
- `newFixedThreadPool` 创建的线程池 `corePoolSize` 和 `maximumPoolSize` 值是相等的，它使用的 `LinkedBlockingQueue`。

源代码
```
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```
代码演示
```
public class MainTest {
    public static void main(String[] args) {
        ExecutorService executor1 = null;
        try {
            executor1 = Executors.newFixedThreadPool(10);
            for (int i = 1; i <= 10; i++) {
                executor1.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "执行了");
                });
            }
        } finally {
            executor1.shutdown();
        }
    }
}
```

#### CachedThreadPool
- 使用`Executors.newCachedThreadPool()`创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
- `newCachedThreadPool` 将 `corePoolSize` 设置为0，将 `maximumPoolSize` 设置为 `Integer.MAX_VALUE`，使用的 `SynchronousQueue`，也就是说来了任务就创建线程运行，当线程空闲超过60秒，就销毁线程。

源代码
```
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

代码演示
```
public class MainTest {
    public static void main(String[] args) {
        ExecutorService executor1 = null;
        try {
            executor1 = Executors.newCachedThreadPool();
            for (int i = 1; i <= 10; i++) {
                executor1.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "执行了");
                });
            }
        } finally {
            executor1.shutdown();
        }
    }
}
```

### 阻塞队列

### 线程池参数
```
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
                 // ...
    }
```
- `corePoolSize`: 线程池中的常驻核心线程数,可理解为初始化线程数
- `maximumPoolSize`：线程池能够容纳同时执行的最大线程数，此值必须大于等于1
- `threadFactory`：线程工厂；表示生成线程池中工作线程的线程工厂，用于创建线程，一般用默认的即可
- `workQueue`：任务队列；随着业务量的增多，线程开始慢慢处理不过来，这时候需要放到任务队列中去等待线程处理
- `rejectedExecutionHandler`：拒绝策略；如果业务越来越多，线程池首先会扩容，扩容后发现还是处理不过来，任务队列已经满了，这时候拒绝接收新的请求
- `keepAliveTime`：多余的空闲线程的存活时间；如果线程池扩容后，能处理过来，而且数据量并没有那么大，用最初的线程数量就能处理过来，剩下的线程被叫做空闲线程
- `unit`：多余的空闲线程的存活时间的单位

### 线程池工作原理
![线程池工作原理](/myblog/posts/images/essays/线程池工作原理.png)

在创建了线程池后，等待提交过来的任务请求;
当调用`execute`方法添加一个请求任务时，线程池会做如下判断：
1. 如果当前运行的线程数小于`corePoolSize`，那么马上创建线程运行该任务
2. 如果当前运行的线程数大于等于`corePoolSize`,那么该任务会被放入任务队列
3. 如果这时候任务队列满了且正在运行的线程数还小于`maximumPoolSize`，那么要创建非核心线程立刻运行这个任务(扩容)
4. 如果任务队列满了且正在运行的线程数等于`maximumPoolSize`，那么线程池会启动饱和拒绝策略来执行
5. 随着时间的推移，业务量越来越少，线程池中出现了空闲线程，当一个线程无事可做超过一定的时间时，线程池会进行判断：
如果当前运行的线程数大于 `corePoolSize`，那么这个线程就被停掉，所以线程池的所有任务完成后它最终会收缩到 `corePoolSize` 的大小

### 四种拒绝策略
在线程池中，如果任务队列满了并且正在运行的线程个数等于允许运行的最大线程数，那么线程池会启动拒绝策略来执行，具体分为下列四种：
- `AbortPolicy`: 默认拒绝策略；直接抛出`java.util.concurrent.RejectedExecutionException`异常，组织系统的正常运行；
- `CallerRunsPolicy`：调用这运行，一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者,从而降低新任务的流量；
- `DiscardOldestPolicy`：抛弃队列中等待最久的任务，然后把当前任务加入到队列中；
- `DiscardPolicy`：直接丢弃任务，不给予任何处理也不会抛出异常；如果允许任务丢失，这是一种最好的解决方案；


### 自定义线程池

**在实际开发中用哪个线程池？**

上面的三种一个都不用，我们生产上只能使用自定义的。

**Executors 中JDK已经给你提供了，为什么不用?**

以下内容摘自[《阿里巴巴开发手册》](https://developer.aliyun.com/topic/download?spm=a2c6h.15028928.J_5293118740.2&id=805)
> 【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。
  说明：线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销，解决资源不足的问题。 如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。
  【强制】线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
  
>说明：Executors 返回的线程池对象的弊端如下：
  1） FixedThreadPool 和 SingleThreadPool： 允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
  2） CachedThreadPool： 允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。


自定义线程池代码演示
```
public class MainTest {
    public static void main(String[] args) {
        ExecutorService executor1 = null;
        try {
            executor1 = new ThreadPoolExecutor(
                    2,
                    5,
                    1L,
                    TimeUnit.SECONDS,
                    new LinkedBlockingQueue<>(3),
                    Executors.defaultThreadFactory(),
                    new ThreadPoolExecutor.CallerRunsPolicy());
            for (int i = 1; i <= 20; i++) {
                executor1.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "执行了");
                });
            }
        } finally {
            executor1.shutdown();
        }
    }
}
```
### 合理配置线程池参数
可以分为以下两种情况
- CPU密集型：CPU密集的意思是该任务需要大量的运算，而没有阻塞，CPU一直全速运行；
  CPU密集型任务配置尽可能少的线程数量：`参考公式：（CPU核数+1）`

- IO密集型：即该任务需要大量的IO，即大量的阻塞；
  在IO密集型任务中使用多线程可以大大的加速程序运行，故需要多配置线程数：参考公式：`CPU核数/ (1-阻塞系数) 阻塞系数在0.8~0.9之间`

代码演示
```
public class MainTest {
    public static void main(String[] args) {
        ExecutorService executor1 = null;
        try {
            // 获取cpu核心数
            int coreNum = Runtime.getRuntime().availableProcessors();
            /*
             * 1. IO密集型: CPU核数/ (1-阻塞系数) 阻塞系数在0.8~0.9之间
             * 2. CPU密集型: CPU核数+1
             */
//            int maximumPoolSize = coreNum + 1;
            int maximumPoolSize = (int) (coreNum / (1 - 0.9));
            System.out.println("当前线程池最大允许存放：" + maximumPoolSize + "个线程");
            executor1 = new ThreadPoolExecutor(
                    2,
                    maximumPoolSize,
                    1L,
                    TimeUnit.SECONDS,
                    new LinkedBlockingQueue<>(3),
                    Executors.defaultThreadFactory(),
                    new ThreadPoolExecutor.CallerRunsPolicy());
            for (int i = 1; i <= 20; i++) {
                executor1.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "执行了");
                });
            }
        } finally {
            executor1.shutdown();
        }
    }
}
```

