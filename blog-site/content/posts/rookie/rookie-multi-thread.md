---
title: "Java多线程"
date: 2021-05-05
draft: false
tags: ["Java", "面向菜鸟编程"]
slug: "rookie-multi-thread"
---

## 相关概念

### 线程与进程
> **进程**是一个具有一定独立功能的程序关于某个数据集合的一次运行活动。例如，一个正在运行的程序的实例就是一个进程。

> **线程**是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。
一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。

进程是操作系统资源分配的基本单位，而线程是处理器任务调度和执行的基本单位。
一个进程至少有一个线程，一个进程可以运行多个线程，多个线程可共享数据。

Java 程序是多线程程序，每启动一个Java程序至少我们知道的都会包含一个主线程和一个垃圾回收线程。
而且启动的时候，每条线程可以并行执行不同的任务。

![线程与进程的关系](/myblog/posts/images/essays/线程与进程的关系.jpg)

### 并行、并发、串行
- 并行：前提是在多核CPU或多个CPU条件下，多个线程同时被多个CPU执行，同时执行的线程并不会抢占CPU资源。
- 串行：前提是在单核CPU条件下，单线程程序执行，不能同时执行，也不能去切换执行。也就是在同一时间段只能做一件事。
- 并发：前提是多线程条件下，多个线程抢占一个CPU资源，多个线程被交替执行。因为CPU运算速度很快，所以用户感觉不到线程切换的卡顿。

无论并行、并发，都可以有多个线程执行，如果是多个线程抢占一个CPU就成了并发，多个CPU同时执行多个线程就是并行。

|           | 单线程 | 多线程 |
| --------- | :----: | ------ |
| **单CPU** |  串行  | 并发   |
| **多CPU** |  串行  | 并行   |

对于单CPU的计算机来说，同一时间是只能干一件事儿的，如果是单线程线程就是串行；如果是多个线程就是并发。
而对于多CPU的计算机说，同一时间能干多个事，如果多个CPU同时执行多个线程就是并行；如果一个CPU同时执行多个线程就是并行。

并行与并发区别图解
![并行与并发区别](/myblog/posts/images/essays/并行与并发区别.png)

并发是两个队列交替使用一台咖啡机，并行是两个队列同时使用两台咖啡机；
如果串行，一个队列使用一台咖啡机，那么哪怕前面那个人便秘了去厕所呆半天，后面的人也只能死等着他回来才能去接咖啡，这效率无疑是最低的。

### 同步与异步
同步、异步一般是相对与方法来说的。

- 同步方法调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为；
- 异步方法调用更像一个消息传递，一旦开始，方法调用就会立即返回，调用者就可以继续后续的操作；
异步方法通常会在另外一个线程中执行着。整个过程，不会阻碍调用者的后续工作。

只有多线程环境下才会异步调用方法，换言之异步调用方法则需要单独创建一个线程。
|    | 单线程 | 多线程 |
| --------- | :----: | ------ |
| **同步** |  只能同步  | 可以同步   |
| **异步** |  不能异步  | 可以异步  |

> PS 线程中的同步(synchronized)机制
在多线程环境下，一旦一个方法或一段代码被synchronized修饰，也就意味着被同步了；
在synchronized修饰的作用域中，某一段时间内只允许一个线程进行操作数据，如果有多个线程需要操作则要排队等待。

### 守护线程
> 守护线程是指为其他线程服务的线程。在JVM中，所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出。

守护线程也称“服务线程”，在没有用户线程可服务时会自动离开。因为主要是服务其他线程所以在程序中的优先级比较低。

举例：垃圾回收线程就是一个经典的守护线程，当我们的程序中不再有任何运行的线程,程序就不会再产生垃圾，垃圾回收器也就无事可做；
所以当垃圾回收线程是JVM上仅剩的线程时，垃圾回收线程会自动离开。它始终在低级别的状态中运行，用于实时监控和管理系统中的可回收资源。

守护线程在程序中的操作演示
```
public class MainTest {
    public static void main(String[] args) {
        int i = 0;
        while (true) {
            Thread daemon = new Thread(() -> {
                System.out.println("启动线程--->" + Thread.currentThread().getName());
            });
            daemon.setDaemon(i==3);
            daemon.start();
            boolean isDaemon = daemon.isDaemon();
            System.out.println("当前线程是否是守护线程：" + isDaemon);
            if (isDaemon) {
                break;
            }
            i++;
        }

    }
}
```

## 线程的状态
线程状态共包含6种，6中状态又可以互相的转换。
![线程状态转换](/myblog/posts/images/essays/线程转换图.jpg)

- 新建状态(New): 创建了线程后尚未启动；
- 可运行状态(Runnable): 可能正在运行，也可能正在等待 CPU 时间片。包含了运行中(Running)和 就绪（Ready)状态；
    - 就绪（Ready）:线程对象创建后，其他线程(比如main线程）调用了该对象的 `start()`方法。该状态的线程位于可运行线程池中，等待被线程调度选中并分配cpu使用权 。
    - 运行中（Runing）：就绪的线程获得了cpu 时间片，开始执行程序代码。
- 阻塞状态(Blocked): 等待获取一个排它锁，如果其线程释放了锁就会结束此状态；
- 无限期等待(Wating): 等待其它线程显式地唤醒，否则不会被分配 CPU 时间片；
- 限期等待(Timed Wating): 无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒;
- 死亡(Terminated): 可以是线程结束任务之后自己结束，或者产生了异常而结束。

> 补充：
> 睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。
> - 调用 `Thread.sleep()` 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。
> - 调用 `Object.wait()` 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。
> - 阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁,而等待是主动的，通过调用 `Thread.sleep()` 和 `Object.wait()` 等方法进入等待。


## 创建线程
在Java中，创建一个线程，有且仅有一种方式: 创建一个`Thread`类实例，并调用它的`start`方法。

### Thread类
通过继承`Thread`类，重写`run()`方法来创建线程。
```
public class MainTest {
    public static void main(String[] args) {
        ThreadDemo thread1 = new ThreadDemo();
        thread1.start();
    }
}
class ThreadDemo extends Thread {
    @Override
    public void run() {
        System.out.printf("通过继承Thread类的方式创建线程,线程 %s 启动",Thread.currentThread().getName());
    }
}
```
### Runnable接口
实现 `Runnale` 接口，将它作为 `target` 参数传递给 `Thread` 类构造函数的方式创建线程。
```
public class MainTest {
    public static void main(String[] args) {
        new Thread(() -> {
            System.out.printf("通过实现Runnable接口的方式，重写run方法创建线程；线程 %s 启动",Thread.currentThread().getName());
        }).start();
    }
}
```
### Callable接口
通过实现 `Callable`接口，来创建一个带有返回值的线程。

在Callable执行完之前的这段时间，主线程可以先去做一些其他的事情，事情都做完之后，再获取Callable的返回结果。可以通过isDone()来判断子线程是否执行完。
```
public class MainTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> futureTask = new FutureTask<>(() -> {
            System.out.printf("通过实现Callable接口的方式，重写call方法创建线程；线程 %s 启动", Thread.currentThread().getName());
            System.out.println();
            Thread.sleep(10000);
            return "我是call方法返回值";
        });
        new Thread(futureTask).start();

        System.out.println("主线程工作中 ...");
        String callRet = null;
        while (callRet == null){
            if(futureTask.isDone()){
                callRet = futureTask.get();
            }
            System.out.println("主线程继续工作 ...");
        }
        System.out.println("获取call方法返回值："+ callRet);
    }
}
```

### 线程池
线程池做的工作主要是控制运行的线程的数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量超出数量的线程排队等候，等其它线程执行完毕，再从队列中取出任务来执行。

它的主要特点为：线程复用，控制最大并发数，管理线程。

优点：
- 降低资源消耗。通过重复利用己创建的线程降低线程创建和销毁造成的消耗。
- 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

#### 常用方式
通过`Executors`线程池工具类来使用：
- `Executors.newSingleThreadExecutor()`：创建只有一个线程的线程池
- `Executors.newFixedThreadPool(int)`：创建固定线程的线程池
- `Executors.newCachedThreadPool()`：创建一个可缓存的线程池，线程数量随着处理业务数量变化

这三种常用创建线程池的方式，底层代码都是用`ThreadPoolExecutor`创建的。

##### SingleThreadExecutor
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

##### FixedThreadPool
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

##### CachedThreadPool
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

#### 阻塞队列
阻塞队列，顾名思义，首先它是一个队列：

![阻塞队列](/myblog/posts/images/essays/阻塞队列结构.jpeg)

一个阻塞队列在数据结构中所起的作用：
- 当阻塞队列是空时，从队列中获取元素的操作将会被阻塞。
- 当阻塞队列是满时，往队列里添加元素的操作将会被阻塞。  

`blockQueue`作为线程容器、阻塞队列，多用于生产者、消费者的关系模式中，保障并发编程线程同步，线程池中被用于当作存储任务的队列，还可以保证线程执行的有序性.**fifo先进先出**

在多线程领域:所谓阻塞,在某些情况下会挂起线程(即线程阻塞),一旦条件满足,被挂起的线程优惠被自动唤醒

##### 为什么使用阻塞队列
我们不需要关心什么时候需要阻塞线程,什么时候需要唤醒线程,因为`BlockingQueue`都一手给你包办好了
在`concurrent`包 发布以前,在多线程环境下,我们每个程序员都必须自己去控制这些细节,尤其还要兼顾效率和线程安全,而这会给我们的程序带来不小的复杂度.

##### 阻塞队列种类
- `ArrayBlockingQueue`: 由数组结构组成的有界阻塞队列.
- `LinkedBlockingDeque`: 由链表结构组成的有界(但大小默认值`Integer>MAX_VALUE`大约21亿)阻塞队列.
- `PriorityBlockingQueue`:支持优先级排序的无界阻塞队列.
- `DelayQueue`: 使用优先级队列实现的延迟无界阻塞队列.
- `SynchronousQueue`:不存储元素的阻塞队列,也即是单个元素的队列.
- `LinkedTransferQueue`:由链表结构组成的无界阻塞队列.
- `LinkedBlockingDeque`:由了解结构组成的双向阻塞队列.

###### ArrayListBlockingQueue
- `add() `:相对列里边添加元素,返回值了类型`boolean`,当超出队列大小时会抛出异常`java.lang.IllegalStateException: Queue full`
- `remove`:清除元素,默认清除队列最上边的元素,可指定元素进行清除,如果清除一个不存在的元素会报异常`java.util.NoSuchElementException`
- `element`:查看队首元素,检查队列为不为空

```
public static void arrayBlockDemo() {
        // 与ArrayList类似,但需要设置队列大小
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<>(3);
        System.out.println(queue.add("c"));
        System.out.println(queue.add("b"));
        System.out.println(queue.add("a"));
        // 当add第四个元素到队列时会抛异常
        queue.add("f");
        //查看队首元素,检查队列为不为空
        System.out.println(queue.element());
        System.out.println(queue.remove());
        System.out.println(queue.remove());
        System.out.println(queue.remove());
        // 如果多清除一个不存在的元素会报异常
        System.out.println(queue.remove());
    }

```
- `offer`:与`add()`类似,但如果添加失败,不会报异常.会返回`false`
- `poll`:与`remove`类似,如果没有元素可取,不会报异常,会返回`null`
- `peek`:与`element`类似
```
public static void arrayBlockDemo2(){
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<>(3);
        System.out.println(queue.offer("1"));
        System.out.println(queue.offer("2"));
        System.out.println(queue.offer("3"));
        // 不会抛异常
        System.out.println(queue.offer("4"));
        System.out.println(queue.peek());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
    }
```

- `put`:当阻塞队列满时,生产者继续往队列里面put元素,队列会一直阻塞直到put数据or响应中断退出
- `take`:获取并移除此队列头元素，若没有元素则一直阻塞.当阻塞队列空时,消费者试图从队列take元素,队列会一直阻塞消费者线程直到队列可用.当阻塞队列满时,队列会阻塞生产者线程一定时间,超过后限时后生产者线程就会退出

###### SynchronousQueue
`SynchronousQueue`，实际上它不是一个真正的队列，因为它不会为队列中元素维护存储空间。与其他队列不同的是，它维护一组线程，这些线程在等待着把元素加入或移出队列.

`SynchronousQueue`支持支持生产者和消费者等待的公平性策略。默认情况下，不能保证生产消费的顺序。
如果是公平锁的话可以保证当前第一个队首的线程是等待时间最长的线程，这时可以视`SynchronousQueue`为一个FIFO队列

```
public class SynchronousQueueDemo {

    public static void main(String[] args) {
        SynchronousQueue<Integer> synchronousQueue = new SynchronousQueue<>();
        new Thread(() -> {
            try {
                synchronousQueue.put(1);
                Thread.sleep(3000);
                synchronousQueue.put(2);
                Thread.sleep(3000);
                synchronousQueue.put(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(() -> {
            try {
                Integer val = synchronousQueue.take();
                System.out.println(val);
                Integer val2 = synchronousQueue.take();
                System.out.println(val2);
                Integer val3 = synchronousQueue.take();
                System.out.println(val3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```
使用场景:
- 生产者消费者模式
- 线程池
- 消息中间件


#### 线程池参数
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

#### 线程池工作原理
![线程池工作原理](/myblog/posts/images/essays/线程池工作原理.png)

在创建了线程池后，等待提交过来的任务请求;
当调用`execute`方法添加一个请求任务时，线程池会做如下判断：
1. 如果当前运行的线程数小于`corePoolSize`，那么马上创建线程运行该任务
2. 如果当前运行的线程数大于等于`corePoolSize`,那么该任务会被放入任务队列
3. 如果这时候任务队列满了且正在运行的线程数还小于`maximumPoolSize`，那么要创建非核心线程立刻运行这个任务(扩容)
4. 如果任务队列满了且正在运行的线程数等于`maximumPoolSize`，那么线程池会启动饱和拒绝策略来执行
5. 随着时间的推移，业务量越来越少，线程池中出现了空闲线程，当一个线程无事可做超过一定的时间时，线程池会进行判断：
如果当前运行的线程数大于 `corePoolSize`，那么这个线程就被停掉，所以线程池的所有任务完成后它最终会收缩到 `corePoolSize` 的大小

#### 四种拒绝策略
在线程池中，如果任务队列满了并且正在运行的线程个数大于等于允许运行的最大线程数，那么线程池会启动拒绝策略来执行，具体分为下列四种：
- `AbortPolicy`: 默认拒绝策略；直接抛出`java.util.concurrent.RejectedExecutionException`异常，阻止系统的正常运行；
- `CallerRunsPolicy`：调用这运行，一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者,从而降低新任务的流量；
- `DiscardOldestPolicy`：抛弃队列中等待最久的任务，然后把当前任务加入到队列中；
- `DiscardPolicy`：直接丢弃任务，不给予任何处理也不会抛出异常；如果允许任务丢失，这是一种最好的解决方案；


#### 自定义线程池

**在实际开发中用哪个线程池？**

上面的三种一个都不用，我们生产上只能使用自定义的。

**`Executors` 中JDK已经给你提供了，为什么不用?**

以下内容摘自[《阿里巴巴开发手册》](https://developer.aliyun.com/topic/download?spm=a2c6h.15028928.J_5293118740.2&id=805)
> 【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。
  说明：线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销，解决资源不足的问题。 如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。
  【强制】线程池不允许使用 `Executors` 去创建，而是通过 `ThreadPoolExecutor` 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
  
>说明：`Executors` 返回的线程池对象的弊端如下：
  1） `FixedThreadPool` 和 `SingleThreadPool`： 允许的请求队列长度为 `Integer.MAX_VALUE`，可能会堆积大量的请求，从而导致 OOM。
  2） `CachedThreadPool`： 允许的创建线程数量为 `Integer.MAX_VALUE`，可能会创建大量的线程，从而导致 OOM。


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
#### 合理配置线程池参数
合理配置线程池参数，可以分为以下两种情况
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

## 锁

### 公平锁与非公平锁


### 可重入锁与不可重入锁

### 悲观锁与乐观锁
乐观锁与悲观锁是一种广义上的概念，可以理解为一种标准类似于Java中的接口。

对于多线程并发操作，加了悲观锁的线程认为每一次修改数据时都会有其他线程来跟它一起修改数据，所以在修改数据之前先会加锁，确保其他线程不会修改该数据。
由于悲观锁在修改数据前先加锁的特性，能保证写操作时数据正确，所以悲观锁更适合写多读少的场景。

乐观锁则与悲观锁相反，每一次修改数据时，都认为没有其他线程来跟它一起修改，所以在修改数据之前不会去添加锁，如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作。
由于乐观锁是一种无锁操作，所以在使用乐观锁的场景中读的性能会大幅度提升，适合读多写少。

![悲观锁与乐观锁](/myblog/posts/images/essays/悲观锁与乐观锁.png)

在Java中悲观锁的实现有：`synchronized`、`Lock`实现类，乐观锁的实现有`CAS`。

### 自旋锁、适应性自旋锁
![自旋锁与非自旋锁](/myblog/posts/images/essays/自旋锁与非自旋锁.png)

#### 自旋锁
当一个线程尝试获取某个锁时，如果该锁已被其他线程占用，就一直循环检测锁是否被释放，而不是进入线程挂起或睡眠状态，直到获取到某个锁。

自旋锁本身是有缺点的，它不能代替阻塞。如果锁被占用的时间很长，那么自旋的线程只会白白浪费处理器资源，带来性能上的浪费。

自旋锁的实现原理是[CAS](#CAS),例如`AtomicInteger`中`getAndUpdate()`方法
```
    public final int getAndUpdate(IntUnaryOperator updateFunction) {
        int prev, next;
        do {
            prev = get();
            next = updateFunction.applyAsInt(prev);
        } while (!compareAndSet(prev, next));
        return prev;
    }
```
源码中的`do-while`循环就是一个自旋操作，如果修改数值失败则通过循环来执行自旋，直至修改成功。

**为什么要使用自旋锁？**

在许多场景中，同步资源的锁定时间很短，为了这一小段时间去切换线程，线程挂起和恢复现场的花费可能会让系统得不偿失。
简单来说就是，避免切换线程带来的开销。

自旋等待虽然避免了线程切换的开销，但它要占用处理器时间。如果锁被占用的时间很短，自旋等待的效果就会非常好。
反之，如果锁被占用的时间很长，那么自旋的线程只会白浪费处理器资源。
所以，自旋等待的时间必须要有一定的限度，如果自旋超过了限定次数（默认是10次，可以使用`-XX:PreBlockSpin`来更改）没有成功获得锁，就应当挂起线程。

自旋锁在JDK 1.4中引入，默认关闭，但是可以使用`-XX:+UseSpinning`开开启;在JDK1.6中默认开启,同时自旋的默认次数为10次，可以通过参数`-XX:PreBlockSpin`来调整。

如果通过参数`-XX:PreBlockSpin`来调整自旋锁的自旋次数，会带来诸多不便。
假如将参数调整为10，但是系统很多线程都是等你刚刚退出的时候就释放了锁（假如多自旋一两次就可以获取锁)。于是JDK1.6引入**适应性自旋锁**。

#### 适应性自旋锁
>自适应意味着自旋的时间（次数）不再固定，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。
如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。
如果对于某个锁，自旋很少成功获得过，那在以后尝试获取这个锁时将可能省略掉自旋过程，直接阻塞线程，避免浪费处理器资源。

### 共享锁与独占锁
### 轻量级锁与重量级锁

### 偏向锁
### 读写锁
### 互斥锁


## 线程安全
线程不安全，在多线程并发的环境中，多个线程共同操作同一个数据，如果最后数据的值和期待值不一样，这时候就出现了线程不安全问题。

线程安全，就是在多线程并发中，多个线程共同操作同一个数据，在Java中最常用的就是加锁；
当一个线程修改某个数据的时候，其他线程不能进行访问直到该线程操作该数据结束释放锁，其他线程才可以继续操作该数据。

一个对象是否安全取决于它是否被多个线程访问，要使对象线程安全，那么需要采用同步的机制来协同对对象可变状态的访问。

### 三大特性
在保证线程安全之前要先弄明白线程安全的三大特性，即原子性、可见性、有序性。

- 原子性,即不可分割,完整性.当某个线程正在做某个业务时,中间不能被打断、加塞,不能被分割.需要整体完整。要么同时成功,要么同时失败.与数据库中的原子性类似。
- 可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即获取到修改的值。
- 有序性即程序执行的顺序按照代码的先后顺序执行。

>**关于有序性肯定会有人有疑问，程序执行的顺序难道不是从上到下按照顺序来执行吗？**
>
>在多线程环境下,Java语句可能会不按照顺序执行,所以要注意数据的依赖性。计算机在执行程序时,为了提高性能,编译器和处理器常常会做指令重排,一把分为以下两种：
>- 单线程环境里面确保程序最终执行结果和代码顺序执行的结果一致。处理器在进行重新排序是必须要考虑指令之间的数据依赖；
>- 多线程环境中线程交替执行,由于编译器优化重排的存在,两个线程使用的变量能否保持一致性是无法确定的,结果无法预测；

### 内存模型
>为了保证并发编程中可以满足原子性、可见性及有序性。内存模型定义了共享内存系统中多线程程序读写操作行为的规范。
通过这些规则来规范对内存的读写操作，从而保证指令执行的正确性。它与处理器有关、与缓存有关、与并发有关、与编译器也有关。
他解决了CPU多级缓存、处理器优化、指令重排等导致的内存访问问题，保证了并发场景下的一致性、原子性和有序性。

内存模型解决并发问题主要采用两种方式：限制处理器优化和使用内存屏障。

**[Java内存模型](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr133.pdf)**

Java内存模型，即JMM（Java Memory Model）本身是一种抽象的概念,并不真实存在,它描述的是一组规则或规范通过规范定制了程序中各个变量(包括实例字段,静态字段和构成数组对象的元素)的访问方式。
屏蔽了各种硬件和操作系统的访问差异的，保证了Java程序在各种平台下对内存的访问都能保证效果一致的机制及规范。

JMM就作用于工作内存和主存之间数据同步过程。他规定了如何做数据同步以及什么时候做数据同步。

![Java内存模型](/myblog/posts/images/essays/Java内存模型.jpg)

**那么在Java中是如何保证原子性、可见性及有序性的呢？**

- 原子性；
在Java中，为了保证原子性，提供了两个高级的字节码指令 `monitorenter` 和 `monitorexit`。对应的就是Java中的关键字 `synchronized`,在Java中只要被`synchronized`修饰就能保证原子性。

- 可见性；
在Java中，为了保证线程间的可见性，可以使用`volatile`、`synchronized`、`final`来修饰。

- 有序性；
在Java中，可以使用 `synchronized` 和 `volatile` 来保证多线程之间操作的有序性。其中，`volatile` 关键字会禁止编译器指令重排，来保证；
`synchronized` 关键字保证同一时刻只允许一条线程操作，而不能禁止指令重排，指令重排并不会影响单线程的顺序，它影响的是多线程并发执行的顺序性，从而保证了有序性。

### volatile
`volatile`通常被比喻成轻量级的锁，也是Java并发编程中比较重要的一个关键字。

`volatile`特点：
- 保证线程之间的可见性；
- 禁止指令重排；
- **不保证原子性，也就是线程不安全；**

#### 使用案例
在Java中`volatile` 是一个变量修饰符，只能用来修饰变量。

`volatile` 典型的使用就是单例模式中的DCL双重检查锁。
```
/**
多线程下的单例模式 DCL(double check lock)
**/
class SingletonDemo {

    // volatile 此处作用 禁止指令重排
    public static volatile SingletonDemo singleton = null;

    private SingletonDemo() {
    }

    public static SingletonDemo getInstance() {
        if (singleton == null) {
            synchronized (SingletonDemo.class) {
                if (singleton == null) {
                    singleton = new SingletonDemo();
                }
            }
        }
        return singleton;
    }

}
```
**为什么在此处要使用`volatile`修饰`singleton`？**

多线程下的DCL单例模式,如果不加 `volatile` 修饰不是绝对安全的,因为在创建对象的时候JVM底层会进行三个步骤:
1. 分配对象的内存空间
2. 初始化对象
3. 设置对象指向刚刚分配的内存地址

其中步骤2和步骤3是没有数据依赖关系的,而且无论重排前还是重排后的程序执行结果在单线程中并没有改变,因此这种重排优化是允许的。
所以有可能先执行步骤3在执行步骤2,导致分配的对象不为 `null`,但对象没有被初始化;

所以当一个线程获取对象不为 `null` 时,由于对象未必已经完成初始化,会存在线程不安全的风险。

#### 原理
《深入理解JVM》中对 `volatile` 的描述: 
>一旦一个共享变量（类的成员变量、类的静态成员变量）被 `volatile` 修饰之后，那么就具备了两层语义：
>- 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的；
>- 禁止进行指令重排序，即有序性；
>
>`volatile`只提供了保证访问该变量时，每次都是从内存中读取最新值，并不会使用寄存器缓存该值——每次都会从内存中读取。
而对该变量的修改，`volatile` 并不提供原子性(线程不安全)的保证;由于及时更新，很可能导致另一线程访问最新变量值，无法跳出循环的情况,多线程下计数器必须使用锁保护.

将[上面的代码](#使用案例)用`javap -v SingletonDemo.class >test.txt`命令执行，将反编译后的字节码指令写入到test文件中,可以看到`ACC_VOLATILE`
```
  public static volatile content.posts.rookie.SingletonDemo singleton;
    descriptor: Lcontent/posts/rookie/SingletonDemo;
    flags: ACC_PUBLIC, ACC_STATIC, ACC_VOLATILE
```
`volatile` 在字节码层面，就是使用访问标志：`ACC_VOLATILE` 来表示，供后续操作此变量时判断访问标志是否为 `ACC_VOLATILE`，来决定是否遵循 `volatile` 的语义处理。

可以从`openjdk8`中找到对应的源码文件
```
路径：openjdk8/hotspot/src/share/vm/interpreter/bytecodeInterpreter.cpp
```
![volitile字节码](/myblog/posts/images/essays/volitile字节码.png)

重点是`cache->is_volatile()`方法，调用栈
```
bytecodeInterpreter.cpp>is_volatile() 
==> accessFlags.hpp>is_volatile 
==> bytecodeInterpreter.cpprelease_byte_field_put
==> oop.inline.hpp>(oopDesc::byte_field_acquire、oopDesc::release_byte_field_put)
==> orderAccess.hpp
>> orderAccess_linux_x86.inline.hpp.OrderAccess::release_store
```
最终调用了`OrderAccess::release_store`
```
inline void     OrderAccess::release_store(volatile jbyte*   p, jbyte   v) { *p = v; }
inline void     OrderAccess::release_store(volatile jshort*  p, jshort  v) { *p = v; }
```
可以从上面看到，到C++的实现层面，又使用C++中的 `volatile` 关键字，用来修饰变量，通常用于建立语言级别的内存屏障`memory barrier`。
在《C++ Programming Language》一书中对 `volatile` 修饰词的解释：
>A volatile specifier is a hint to a compiler that an object may change its value in ways not specified by the language so that aggressive optimizations must be avoided.

- `volatile` 修饰的类型变量表示可以被某些编译器未知的因素更改;
- 使用 `volatile` 变量时，避免激进的优化; 系统总是重新从内存读取数据，即使它前面的指令刚从内存中读取被缓存，防止出现未知更改和主内存中不一致。

其在64位系统的实现`orderAccess_linux_x86.inline.hpp.OrderAccess::release_store`
```
inline void OrderAccess::fence() {
  if (os::is_MP()) {
    // always use locked addl since mfence is sometimes expensive
#ifdef AMD64
    __asm__ volatile ("lock; addl $0,0(%%rsp)" : : : "cc", "memory");
#else
    __asm__ volatile ("lock; addl $0,0(%%esp)" : : : "cc", "memory");
#endif
  }
}
```
代码`lock; addl $0,0(%%rsp)`就是lock前缀；

> lock前缀，会保证某个处理器对共享内存的独占使用。
它将本处理器缓存写入内存，该写入操作会引起其他处理器或内核对应的缓存失效。
通过独占内存、使其他处理器缓存失效，达到了“指令重排序无法越过内存屏障”的作用。


对于 `volatile`修饰的变量，当对 `volatile` 修饰的变量进行写操作的时候，JVM会向处理器发送一条带有 `lock` 前缀的指令，将这个缓存中的变量回写到系统主存中。
但是就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题，所以在多处理器下，为了保证各个处理器的缓存是一致的，就会实现 **缓存一致性协议**

>缓存一致性协议: 每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器要对这个数据进行修改操作的时候，会强制重新从系统内存里把数据读到处理器缓存里。

为了提高CPU处理器的执行速度，在处理器和内存之间增加了多级缓存来提升。但是由于引入了多级缓存，就存在缓存数据不一致问题。

![CPU多级缓存](/myblog/posts/images/essays/CPU多级缓存.jpg)

所以，如果一个变量被 `volatile` 所修饰的话，在每次数据变化之后，其值都会被强制刷入主存。而其他处理器的缓存由于遵守了缓存一致性协议，也会把这个变量的值从主存加载到自己的缓存中。这就保证了一个 `volatile` 在并发编程中，其值在多个缓存中是可见的。

#### volatile与可见性
各个线程对主内存中共享变量的操作，都是各个线程各自拷贝到自己的工作内存操作后再写回主内存中的。
这就可能存在一个线程AAA修改了共享变量X的值还未写回主内存中时 ,另外一个线程BBB又对内存中的一个共享变量X进行操作,但此时A线程工作内存中的共享比那里X对线程B来说并不不可见。
这种工作内存与主内存同步延迟现象就造成了可见性问题。

这种变量的可见性问题可以用`volatile`来解决。`volatile`的作用简单来说就是当一个线程修改了数据,并且写回主物理内存,其他线程都会得到通知获取最新的数据。

`volatile`可见性，代码演示
```
public class MainTest {
    public static void main(String[] args) {
        A a = new A();
        // thread1
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " is come in");
            try {
                // 模拟执行其他业务
                Thread.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 用该线程改变A类中 number 变量的值
            a.numberTo100();
        }, "thread1").start();
        
        // 如果number 等于0，则其他线程会一直等待 则证明 volatile 没有保证变量的可见性；相反则保证了变量的可见性
        while (a.number == 0) {
        }
        System.out.println(Thread.currentThread().getName() + " thread is over");
    }
}
class A {
    // 注意: 此时变量要加 volatile 关键字修饰； 可以去掉 volatile 来进行对比测试
    volatile int number = 0;

    public void numberTo100() {
        System.out.println(Thread.currentThread().getName() + " update number");
        this.number = 100;
    }
}
```
#### volatile与原子性
`volatile`原子性，代码演示
```
public class MainTest {

    public static void main(String[] args) {
        A a = new A();
        /**
         * 创建20个线程 每个线程让 number++ 1000次；
         * number 变量用 volatile 修饰
         * 如果 volatile 保证变量的原子性，则最后结果为 20 * 1000，反之则不保证。
         * 当然不排除偶然事件，建议反复多试几次。
         */
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    a.addPlusplus();
                }
            }, String.valueOf(i)).start();
        }
        // 如果当前存活线程大于 2 个(包括main线程) 礼让线程继续执行上边的线程
        while (Thread.activeCount() > 2) {
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName() + " Thread is over\t" + a.number);

    }

}

class A {
    volatile int number = 0;

    public void addPlusplus() {
        this.number++;
    }
}
```
**不保证原子性的原因**

由于各个线程之间都是复制主内存的数据到自己的工作空间里边修改数据,CPU的轮询反复切换线程,会导致数据丢失。
即某个线程修改了数据,准备回主内存,此时CPU切换到另一个线程修改了数据,并且写回到了主内存,此时其他的线程不知道主内存的数据已经被更改,还会执行将之前从主内存复制的数据修改后的,写到主内存,这就导致了数据被覆盖,丢失。

**解决**

如果要解决原子性的问题,在Java中只能控制线程，在修改的时候不能被中断，即加锁。
上面的例子可以使用[CAS](#CAS)的实现`AtomicInteger`来解决。

```
public class MainTest {

    public static void main(String[] args) {
        A a = new A();
        /**
         * 创建20个线程 每个线程让 number++ 1000次；
         * number 变量用 volatile 修饰
         * 如果 volatile 保证变量的原子性，则最后结果为 20 * 1000，反之则不保证。
         * 当然不排除偶然事件，建议反复多试几次。
         */
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    a.addPlusplus();
                }
            }, String.valueOf(i)).start();
        }
        // 如果当前存活线程大于 2 个(包括main线程) 礼让线程继续执行上边的线程
        while (Thread.activeCount() > 2) {
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName() + " Thread is over\t" + a.number);

    }

}

class A {

    int number = 0;

    /**
     * 如果要解决原子性的问题可以用synchronized 关键字(这种太浪费性能)
     * 可用JUC下的 AtomicInteger 来解决
     **/
    AtomicInteger atomicInteger = new AtomicInteger(number);

    public void addPlusplus() {
        number = atomicInteger.incrementAndGet();
    }
}
```

对于`AtomicInteger.incrementAndGet`方法来说，原理就是`volatile` + `do...while()` + `CAS`;

`AtomicInteger.incrementAndGet`源码
```
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
//=========================
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

用`volatile`修饰该变量，保证该变量被某个线程修改时，保证其他线程中的这个变量的可见性；
在多线程环境下，CPU轮流切换线程执行，有可能某个线程修改了数据,准备回主内存,此时CPU切换到另一个线程修改了数据,并且写回到了主内存，此时就导致数据的不准确；
`do...while()` + `CAS`的作用就是，当某个线程工作内存中的值与主内存中的值，如果不相同就会一直`while`循环下去,之所以用`do..while`是考虑到做自增操作。

#### volatile与有序性
有序性，指的就是代码按照顺序执行，这个就是对比指令重排来说的；计算机在执行程序时,为了提高性能,编译器和处理器常常会做指令重排。

在上面的[使用案例](#使用案例)中的代码，DCL就是一个使用禁止指令重排的案例。

**`volatile` 禁止指令重排原因**

由于编译器和处理器都能执行指令重排的优化,如果在指令键加入一条内存屏障(`Memory barrier`),就会告诉编译器和CPU不管什么指令都不能和这条加入`Memory barrier`指令键重新排序,也就是说**通过内存屏障禁止在内存屏障前后的指令重新排序优化**。
内存屏障的另一个作用就是强制刷出各种CPU缓存数据,因此任何CPU上的线程都能读取到这些数据的最新值，即可见性。


### CAS
CAS全称为`Compare and Swap`被译为比较并交换。是一种无锁算法。在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。
`java.util.concurrent.atomic` 并发包下的所有原子类都是基于 `CAS` 来实现的。

以 `AtomicInteger` 原子整型类为例。
```
public class MainTest {
    public static void main(String[] args) {
        new AtomicInteger().compareAndSet(1,2);
    }
}
```
以上面的代码为例，调用栈如下：
```
compareAndSet --> unsafe.compareAndSwapInt ---> unsafe.compareAndSwapInt --> (C++) cmpxchg
```
`AtomicInteger` 内部方法都是基于 `Unsafe` 类实现的。
```
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```
参数:
- `this`: `Unsafe` 对象本身，需要通过这个类来获取 `value` 的内存偏移地址;
- `valueOffset`: `value` 变量的内存偏移地址;
- `expect`: 期望更新的值;
- `update`: 要更新的最新值;

偏移量`valueOffset`
```
// setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
```

1. `Unsafe` 是CAS的核心类，Java无法直接访问底层操作系统，而是通过 `native` 方法来访问。不过尽管如此，JVM还是开了一个后门，JDK中有一个类 `Unsafe`，它提供了硬件级别的原子操作。
2. `valueOffset` 表示的是变量值在内存中的偏移地址，因为 `Unsafe` 就是根据内存偏移地址获取数据的原值的。
3. `value` 是用 `volatile` 修饰的，保证了多线程之间看到的 `value` 值是同一份。

继续向底层深入，就会看到`Unsafe`类中的一些方法，同时也是CAS的核心方法：
```
public final class Unsafe {

    // ...

    public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

    public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

    public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
    
    // ...
}
```
上面的三个方法的原理，可以对应去查看 `openjdk` 的 `hotspot`源码：`src/share/vm/prims/unsafe.cpp`
```
#define FN_PTR(f) CAST_FROM_FN_PTR(void*, &f)

{CC"compareAndSwapObject", CC"("OBJ"J"OBJ""OBJ")Z",  FN_PTR(Unsafe_CompareAndSwapObject)},

{CC"compareAndSwapInt",  CC"("OBJ"J""I""I"")Z",      FN_PTR(Unsafe_CompareAndSwapInt)},

{CC"compareAndSwapLong", CC"("OBJ"J""J""J"")Z",      FN_PTR(Unsafe_CompareAndSwapLong)},
``` 
最终在 `hotspot` 源码实现中都会调用统一的 `cmpxchg` 函数，`/src/share/vm/runtime/Atomic.cpp`
```
jbyte Atomic::cmpxchg(jbyte exchange_value, volatile jbyte*dest, jbyte compare_value) {
         assert (sizeof(jbyte) == 1,"assumption.");
         uintptr_t dest_addr = (uintptr_t) dest;
         uintptr_t offset = dest_addr % sizeof(jint);
         volatile jint*dest_int = ( volatile jint*)(dest_addr - offset);
         // 对象当前值
         jint cur = *dest_int;
         // 当前值cur的地址
         jbyte * cur_as_bytes = (jbyte *) ( & cur);
         // new_val地址
         jint new_val = cur;
         jbyte * new_val_as_bytes = (jbyte *) ( & new_val);
          // new_val存exchange_value，后面修改则直接从new_val中取值
         new_val_as_bytes[offset] = exchange_value;
         // 比较当前值与期望值，如果相同则更新，不同则直接返回
         while (cur_as_bytes[offset] == compare_value) {
          // 调用汇编指令cmpxchg执行CAS操作，期望值为cur，更新值为new_val
             jint res = cmpxchg(new_val, dest_int, cur);
             if (res == cur) break;
             cur = res;
             new_val = cur;
             new_val_as_bytes[offset] = exchange_value;
         }
         // 返回当前值
         return cur_as_bytes[offset];
}
```
从上述源码可以看出CAS的原理就是调用了汇编指令 `cmpxchg` ,最终其实也就调用了CPU的某些指令。


CAS作用也一目了然，在多线程环境中，就是比较当前线程工作内存中的值和主内存中的值，如果相同则执行规定操作，否则继续比较，直到主内存和当前线程工作内存中的值一致为止。

例如代码：
```
public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
        return var5;
}
```

#### 如何保证数据一致性

从源码可以看出，CAS是通过`Unsafe`调用CPU指令,当CPU中某个处理器对缓存中的共享变量进行了操作，其他处理器会有个嗅探机制，将其他处理器的该共享变量的缓存失效，待其他线程读取时会重新从主内存中读取最新的数据，基于 `MESI` 缓存一致性协议来实现的。

简述，就是通过CPU的缓存一致性协议来保证线程之间的数据一致性的。

> CPU 处理器速度远远大于在主内存中的，为了解决速度差异，在他们之间架设了多级缓存，如 L1、L2、L3 级别的缓存，这些缓存离CPU越近就越快，将频繁操作的数据缓存到这里，加快访问速度。

![CPU多级缓存](/myblog/posts/images/essays/CPU多级缓存.jpg)


#### CAS与Unsafe关系
CAS的作用是比较并交换,就是先拿这个期望值,与主内存的值比较,判断主内存中该位置是否存在期望值,
如果存在,则改为新的值,这个修改的过程是具有原子性的.
因为CAS是cpu并发源语,并发源语体现在`Java sun.misc.Unsafa`类上.
调用Unsafe类中的CAS方法，JVM会帮我们实现CAS汇编指令。这是一种完全依赖于硬件的功能，通过他实现了原子操作。
由于CAS是一种系统原语，原语属于操作系统用语范畴，是由若干条指令组成的，用于完成某个功能的一个过程，并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU的原子指令，不会造成数据不一致问题。

> PS Unsafe类
CAS其实是调用了 `Unsafe` 类的方法 `Unsafa` 类是CAS核心类，由于Java方法无法直接访问底层系统，需要通过本地（`native`）方法来访问，`Unsafe` 相当于一个后门，基于该类可以直接操作特定内存数据。
Unsafe类存在于sun.misc包中，其内部方法操作可以像C的指针(内存地址)一样直接操作内存，因此Java中CAS操作的执行依赖于Unsafe类的方法。
Unsafe类中的所有方法都是**native**修饰的，也就是说Unsafe类中的方法都直接**调用操作系统底层资源执行相应任务**。

#### 缺点
##### 循环时间长开销
因为是采用自旋锁的方式来实现所以，自然有自旋锁的缺点，循环时间长开销大,例如：`getAndAddInt` 方法执行，有个`do while`循环，如果CAS失败，一直会进行尝试，如果CAS长时间不成功，可能会给CPU带来很大的开销。
```
public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
        return var5;
}
```
##### 多个变量原子性
只能保证一个共享变量的原子操作，对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁来保证原子性。
但是Java从1.5开始JDK提供了`AtomicReference`类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。

##### ABA问题

ABA问题示例代码:
````
public class MainTest {
    static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
    public static void main(String[] args) {
        new Thread(() -> {
            // 先改到101在改回来，CAS会认为value没有被修改过
            atomicReference.compareAndSet(100, 101);
            atomicReference.compareAndSet(101, 100);
        }, "Thread 1").start();

        new Thread(() -> {
            try {
                //保证线程1完成一次ABA操作
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(atomicReference.compareAndSet(100, 2019) + "\t" + atomicReference.get());
        }, "Thread 2").start();
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
````
CAS算法实现一个重要前提是，需要去除内存中某个时刻的数据并在当下时刻比较并替换，那么在这个时间差类会导致数据的变化。

比如，线程1从内存位置V取出A，线程2同时也从内存取出A，并且线程2进行一些操作将值改为B，然后线程2又将V位置数据改成A，这时候线程1进行CAS操作发现内存中的值依然时A，然后线程1操作成功。
尽管线程1的CAS操作成功，但是不代表这个过程没有问题。

简单说，如果一个线程改了一个值，最后又改回到初始值了，这时候CAS会认为它没有被修改过。简而言之就是只比较结果,不比较过程。

**ABA问题解决**

利用 `AtomicReference` 类进行原子引用
````
public class AtomicRefrenceDemo {
    public static void main(String[] args) {
        User z3 = new User("张三", 22);
        User l4 = new User("李四", 23);
        AtomicReference<User> atomicReference = new AtomicReference<>();
        atomicReference.set(z3);
        System.out.println(atomicReference.compareAndSet(z3, l4) + "\t" + atomicReference.get().toString());
        System.out.println(atomicReference.compareAndSet(z3, l4) + "\t" + atomicReference.get().toString());
    }
}

@Getter
@ToString
@AllArgsConstructor
class User {
    String userName;
    int age;
}
````
````
// 输出结果
true	User(userName=李四, age=23)
false	User(userName=李四, age=23)
````

使用时间戳的原子引用`AtomicStampedReference`修改版本号。主要是在对象中额外再增加一个标记来标识对象是否有过变更
````
static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(100, 1);

public static void main(String[] args) {
    new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t第1次版本号" + stamp);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            atomicStampedReference.compareAndSet(100, 101, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "\t第2次版本号" + atomicStampedReference.getStamp());
            atomicStampedReference.compareAndSet(101, 100, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "\t第3次版本号" + atomicStampedReference.getStamp());
        }, "Thread 3").start();

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t第1次版本号" + stamp);
            try {
                TimeUnit.SECONDS.sleep(4);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean result = atomicStampedReference.compareAndSet(100, 2019, stamp, stamp + 1);

            System.out.println(Thread.currentThread().getName() + "\t修改是否成功" + result + "\t当前最新实际版本号：" + atomicStampedReference.getStamp());
            System.out.println(Thread.currentThread().getName() + "\t当前最新实际值：" + atomicStampedReference.getReference());
        }, "Thread 4").start();
}
````
````
Thread 3	第1次版本号1
Thread 4	第1次版本号1
Thread 3	第2次版本号2
Thread 3	第3次版本号3
Thread 4	修改是否成功false	当前最新实际版本号：3
Thread 4	当前最新实际值：100
````


### Lock
#### 使用
#### 原理
#### trylock、lock
### AQS

### synchronized
`synchronized`是Java提供的关键字，可译为同步。可用来给对象、方法或者代码块加锁，当它锁定一个方法或者一个代码块的时候，同一时刻最多只有一个线程执行这段代码。

> `synchronized`关键字在需要原子性、可见性和有序性这三种特性的时候都可以作为其中一种解决方案，看起来是“万能”的。的确，大部分并发控制操作都能使用`synchronized`来完成。

#### 使用
| 修饰的对象       | 作用范围     | 作用对象           |
| ---------------- | ------------ | ------------------ |
| 同步一个实例方法     | 整个实例方法     | 调用此方法的对象   |
| 同步一个静态方法 | 整个静态方法 | 此类的所有对象     |
| 同步代码块-对象  | 整个代码块   | 调用此代码块的对象 |
| 同步代码块-类   | 整个代码块   | 此类的所有对象     |

##### 同步一个方法

代码演示
```
public class MainTest {

    //共享资源
    static int i = 0;

    public static void main(String[] args) throws InterruptedException {
        MainTest mainTest = new MainTest();
        Thread thread1 = new Thread(() -> {
            for (int j = 0; j < 1000000; j++) {
                mainTest.increase();
            }
        }, "线程1");
        Thread thread2 = new Thread(() -> {
            for (int j = 0; j < 1000000; j++) {
                mainTest.increase();
            }
        }, "线程2");

        thread1.start();
        thread2.start();

        // join方法的作用是调用线程等待该线程完成后，才能继续用下运行。
        thread1.join();
        thread2.join();
        System.out.println(i);
    }

    public synchronized void increase() {
        i++;
    }

    // 通过是否使用synchronized来体会
//    public  void increase() {
//        i++;
//    }
}
```
对于上面的代码如果加上`synchronized`最后输出的结果为2000000；
如果没有加，最后的结果很大程度上是小于2000000的，当然不排除偶然情况，所以这里不是肯定句。

由此可见，当某个线程运行到这个方法时,都要检查有没有其它线程正在用这个方法(或者该类的其他同步方法)，有的话要等待正在使用 `synchronized` 方法的线程运行完这个方法后再运行此线程,没有的话,锁定调用者,然后直接运行。

##### 同步一个静态方法
当 `synchronized` 作用于静态方法时，其锁就是当前类的class对象锁。由于静态成员不专属于任何一个实例对象，是类成员，因此通过class对象锁可以控制静态 成员的并发操作。

需要注意的是如果一个线程A调用一个实例对象的非`static synchronized`方法，而线程B需要调用这个实例对象所属类的静态 `synchronized` 方法，是允许的，不会发生互斥现象;
因为访问静态 `synchronized` 方法占用的锁是当前类的class对象，而访问非静态 `synchronized` 方法占用的锁是当前实例对象锁。

代码演示
```
public class MainTest {

    //共享资源
    static int i = 0;

    public static void main(String[] args) throws InterruptedException {
        MainTest mainTest = new MainTest();
        Thread thread1 = new Thread(() -> {
            for (int j = 0; j < 1000000; j++) {
//                increase();
                mainTest.increaseNoneStatic();
            }
        }, "线程1");
        Thread thread2 = new Thread(() -> {
            for (int j = 0; j < 1000000; j++) {
//                increase();
//                mainTest.increaseNoneStatic();
            }
        }, "线程2");

        thread1.start();
        thread2.start();

        // join方法的作用是调用线程等待该线程完成后，才能继续用下运行。
        thread1.join();
        thread2.join();
        System.out.println(i);
    }

    // static修饰 锁住的是类对象
    public static synchronized void increase() {
        i++;
    }

    // 无static修饰 锁住的是调用该方法的 当前对象
    public synchronized void increaseNoneStatic() {
        i++;
    }
}
```
同步一个静态方法，作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁。也就是给当前类加锁，会作用于类的所有对象实例，因为静态成员不属于任何一个实例对象，是类成员（static表明这是该类的一个静态资源，不管new了多少个对象，只有一份，所以对该类的所有对象都加了锁）。
所以如果一个线程A调用一个实例对象的非静态`synchronized`方法，而线程B需要调用这个实例对象所属类的静态`synchronized`方法，是允许的，不会发生互斥现象，因为访问静态`synchronized`方法占用的锁是当前类的锁，而访问非静态`synchronized`方法占用的锁是当前实例对象锁。

##### 同步代码块
在某些情况下，我们编写的方法体可能比较大，同时存在一些比较耗时的操作，而需要同步的代码又只有一小部分，如果直接对整个方法进行同步操作,这样做就有点浪费；
此时我们可以使用同步代码块的方式对需要同步的代码进行包裹。

代码演示
```
public class MainTest {

    //共享资源
    static int i = 0;

    public static void main(String[] args) throws InterruptedException {
        MainTest mainTest = new MainTest();
        Thread thread1 = new Thread(() -> {
            for (int j = 0; j < 1000000; j++) {
                mainTest.increase();
            }
        }, "线程1");
        Thread thread2 = new Thread(() -> {
            for (int j = 0; j < 1000000; j++) {
                mainTest.increase();
            }
        }, "线程2");

        thread1.start();
        thread2.start();

        // join方法的作用是调用线程等待该线程完成后，才能继续用下运行。
        thread1.join();
        thread2.join();
        System.out.println(i);
    }

    public  void increase() {
        synchronized (this){
            i++;
        }
    }

//    public  void increase() {
//        i++;
//    }

}
```
除了使用`synchronized (this)`锁定，当然静态方法是没有this对象的；也可以使用`class`对象,和程序中创建的一些对象来做为锁。
```
// class类对象锁
synchronized(MainTest.class){
    // ...
}

// 
```
当没有明确的对象作为锁，只是想让一段代码同步时，可以创建一个特殊的对象来充当锁;
```
private byte[] lock = new byte[0];
public void method(){
  synchronized(lock) {
     // .....
  }
}
```
零长度的`byte`数组对象创建起来将比任何对象都经济――查看编译后的字节码：生成零长度的`byte[]`对象只需3条操作码，而`Object lock = new Object()`则需要7行操作码。

当一个线程访问对象的一个`synchronized(this)`同步代码块时，另一个线程仍然可以访问该对象中的非`synchronized(this)`同步代码块。 
```
public class MainTest {
    public static void main(String[] args) {
        Counter counter = new Counter();
        Thread thread1 = new Thread(counter, "A");
        Thread thread2 = new Thread(counter, "B");
        thread1.start();
        thread2.start();
    }
}

class Counter implements Runnable{
    private int count;

    public Counter() {
        count = 0;
    }

    public void countAdd() {
        synchronized(this) {
            for (int i = 0; i < 5; i ++) {
                try {
                    System.out.println(Thread.currentThread().getName() + ":" + (count++));
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    //非synchronized代码块，未对count进行读写操作，所以可以不用synchronized
    public void printCount() {
        for (int i = 0; i < 5; i ++) {
            try {
                System.out.println(Thread.currentThread().getName() + " count:" + count);
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void run() {
        String threadName = Thread.currentThread().getName();
        if (threadName.equals("A")) {
            countAdd();
        } else if (threadName.equals("B")) {
            printCount();
        }
    }
}
```

#### 原理
参考：
- https://www.hollischuang.com/archives/2030
- https://blog.csdn.net/javazejian/article/details/72828483
- https://blog.csdn.net/luoweifu/article/details/46613015

通过上面的使用，可以体会到被`synchronized`修饰的代码块及方法，在同一时间，只能被单个线程访问。

用`javap -v MainTest.class` 命令反编译下面代码，我们就能了解到JVM对`synchronized`是怎么处理的了。
```
public class MainTest {

    synchronized void demo01() {
        System.out.println("demo 01");
    }

    void demo02() {
        synchronized (MainTest.class) {
            System.out.println("demo 02");
        }
    }

}
```
```
  synchronized void demo01();
    descriptor: ()V
    flags: ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String demo 01
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
// ...
void demo02();
    descriptor: ()V
    flags:
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #5                  // class content/posts/rookie/MainTest
         2: dup
         3: astore_1
         4: monitorenter
         5: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         8: ldc           #6                  // String demo 02
        10: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        13: aload_1
        14: monitorexit
        15: goto          23
        18: astore_2
        19: aload_1
        20: monitorexit
        21: aload_2
        22: athrow
        23: return
// ...
```
通过反编译后代码可以看出：
- 对于同步方法，JVM采用`ACC_SYNCHRONIZED`标记符来实现同步；
- 对于同步代码块，JVM采用`monitorenter`、`monitorexit`两个指令来实现同步;

其中同步代码块，有两个`monitorexit`指令的原因是,为了保证抛异常的情况下也能释放锁，所以`javac`为同步代码块添加了一个隐式的`try-finally`，在`finally`中会调用`monitorexit`命令释放锁。

[官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10)中关于同步方法和同步代码块的实现原理描述
> 方法级的同步是隐式的。同步方法的常量池中会有一个 `ACC_SYNCHRONIZED` 标志。当某个线程要访问某个方法的时候，会检查是否有 `ACC_SYNCHRONIZED`，如果有设置，则需要先获得监视器锁，然后开始执行方法，方法执行之后再释放监视器锁。这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。值得注意的是，如果在方法执行过程中，发生了异常，并且方法内部并没有处理该异常，那么在异常被抛到方法外面之前监视器锁会被自动释放。

> 同步代码块使用 `monitorenter` 和 `monitorexit` 两个指令实现。可以把执行 `monitorenter` 指令理解为加锁，执行 `monitorexit` 理解为释放锁。 每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为0，当一个线程获得锁（执行 `monitorenter`）后，该计数器自增变为 1 ，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行 `monitorexit` 指令）的时候，计数器再自减。当计数器为0的时候。锁将被释放，其他线程便可以获得锁。

其实无论是`ACC_SYNCHRONIZED`还是`monitorenter`、`monitorexit`都是基于`Monitor`实现的，每一个锁都对应一个`monitor`对象;
在Java虚拟机(HotSpot)中，`Monitor` 是基于C++实现的，由`ObjectMonitor`实现。

在`/hotspot/src/share/vm/runtime/objectMonitor.hpp`中有`ObjectMonitor`的实现
```
// initialize the monitor, exception the semaphore, all other fields
// are simple integers or pointers
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; //记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```
>- `_owner`：指向持有`ObjectMonitor`对象的线程
>- `_WaitSet`：存放处于`wait`状态的线程队列
>-`_EntryList`：存放处于等待锁`block`状态的线程队列
>- `_recursions`：锁的重入次数
>-`_count`：用来记录该线程获取锁的次数

若持有`monitor`的线程调用`wait()`方法，将释放当前持有的`monitor`，`_owner`变量恢复为null，`_count`自减1，同时该线程进入`_WaitSet`集合中等待被唤醒。若当前线程执行完毕也将释放`monitor`(锁)并复位变量的值，以便其他线程进入获取`monitor`(锁)。

当多个线程同时访问一段同步代码时，首先会进入`_EntryList`队列中，当某个线程获取到对象的`monitor`后进入_Owner区域并把`monitor`中的`_owner`变量设置为当前线程，同时`monitor`中的计数器`_count`加1。即获得对象锁。

`ObjectMonitor`中方法
```
  bool      try_enter (TRAPS) ;
  void      enter(TRAPS);
  void      exit(bool not_suspended, TRAPS);
  void      wait(jlong millis, bool interruptable, TRAPS);
  void      notify(TRAPS);
  void      notifyAll(TRAPS);
```
`sychronized`加锁的时候，会调用`objectMonitor`的`enter`方法，解锁的时候会调用`exit`方法。
在JDK1.6之前，`synchronized` 的实现才会直接调用 `ObjectMonitor`的`enter`和`exit`，这种锁被称之为重量级锁。

> 重量级锁:
> Java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统的帮忙，这就要从用户态转换到核心态，因此状态转换需要花费很多的处理器时间;
对于代码简单的同步块（如被`synchronized`修饰的`get`、`set`方法）状态转换消耗的时间有可能比用户代码执行的时间还要长，所以说`synchronized`是java语言中一个重量级的操纵。

所以，在JDK1.6中出现对锁进行了很多的优化，进而出现轻量级锁，偏向锁，锁消除，适应性自旋锁，锁粗化。

#### 锁的升级
阅读之前了解[Java对象头](http://localhost:1313/myblog/posts/jvm/java-object/#对象头)效果更佳哦。




#### [synchronized与可见性](http://hollischuang.gitee.io/tobetopjavaer/#/basics/concurrent-coding/synchronized?id=synchronized与可见性)
可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

Java内存模型规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中是用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存。
不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步进行。所以，就可能出现线程1改了某个变量的值，但是线程2不可见的情况。
被`synchronized`修饰的代码，在开始执行时会加锁，执行完成后会进行解锁。而为了保证可见性，有一条规则是这样的：对一个变量解锁之前，必须先把此变量同步回主存中。这样解锁后，后续线程就可以访问到被修改后的值。

所以，`synchronized`关键字锁住的对象，其值是具有可见性的.


#### [synchronized与原子性](http://hollischuang.gitee.io/tobetopjavaer/#/basics/concurrent-coding/synchronized?id=synchronized与原子性)
原子性是指一个操作是不可中断的，要全部执行完成，要不就都不执行。

线程是CPU调度的基本单位。CPU有时间片的概念，会根据不同的调度算法进行线程调度。当一个线程获得时间片之后开始执行，在时间片耗尽之后，就会失去CPU使用权。所以在多线程场景下，由于时间片在线程间轮换，就会发生原子性问题。

在Java中，为了保证原子性，提供了两个高级的字节码指令``monitorenter``和``monitorexit``。
这两个字节码指令，在Java中对应的关键字就是``synchronized``。

通过下``monitorexit``和``monitorexit``指令，可以保证被``synchronized``修饰的代码在同一时间只能被一个线程访问，在锁未释放之前，无法被其他线程访问到。因此，在Java中可以使用``synchronized``来保证方法和代码块内的操作是原子性的。

>例如： 线程1在执行``monitorenter``指令的时候，会对Monitor进行加锁，加锁后其他线程无法获得锁，除非线程1主动解锁。
即使在执行过程中，由于某种原因，比如CPU时间片用完，线程1放弃了CPU，但是，他并没有进行解锁。
而由于``synchronized``的锁是可重入的，下一个时间片还是只能被他自己获取到，还是会继续执行代码。直到所有代码执行完。这就保证了原子性。


#### [synchronized与有序性](http://hollischuang.gitee.io/tobetopjavaer/#/basics/concurrent-coding/synchronized?id=synchronized与有序性)
有序性即程序执行的顺序按照代码的先后顺序执行。

除了引入了时间片以外，由于处理器优化和指令重排等，CPU还可能对输入代码进行乱序执行，比如``load->add->save`` 有可能被优化成``load->save->add``这就是可能存在有序性问题。

这里需要注意的是，``synchronized``是无法禁止指令重排和处理器优化的。也就是说，``synchronized``无法避免上述提到的问题。

那么，为什么还说``synchronized``也提供了有序性保证呢？

这就要再把有序性的概念扩展一下了。Java程序中天然的有序性可以总结为一句话,《深入理解Java虚拟机》:
> 如果在本线程内观察，所有操作都是天然有序的。如果在一个线程中观察另一个线程，所有操作都是无序的。

以上这句话也是，但是怎么理解呢？简单扩展一下，这其实和``as-if-serial``语义有关。

``as-if-serial``语义的意思指：不管怎么重排序（编译器和处理器为了提高并行度），单线程程序的执行结果都不能被改变。
编译器和处理器无论如何优化，都必须遵守``as-if-serial``语义。

这里不对``as-if-serial``语义详细展开了，简单说就是``as-if-serial``语义保证了单线程中，指令重排是有一定的限制的，而只要编译器和处理器都遵守了这个语义，那么就可以认为单线程程序是按照顺序执行的。当然，实际上还是有重排的，只不过我们无须关心这种重排的干扰。

所以呢，由于``synchronized``修饰的代码，同一时间只能被同一线程访问。那么也就是单线程执行的。所以，可以保证其有序性。

## 常用的线程安全的集合
| 线程不安全 | 线程不安全解决方案                                           |
| ---------- | ------------------------------------------------------------ |
| ArrayList  | 使用Vector、Collections.synchronizedArrayList、CopyOnWriteArrayList |
| HashSet    | 使用Collections.synchronizedSet、CopyOnWriteArraySet         |
| HashMap    | 使用HashTable、Collections.synchronizedMap、ConcurrentHashMap |

### ArrayList线程不安全
`ArrayList`线程不安全代码演示
```
public class MainTest {
    public static void main(String[] args) {
        ArrayList<String> arrayList = new ArrayList<>();
        for(int i=0; i< 10; i++) {
            new Thread(() -> {
                arrayList.add(UUID.randomUUID().toString());
                System.out.println(arrayList);
            },String.valueOf(i)).start();
        }
    }
}
```
为避免偶然事件，请重复多试几次上面的代码,很大情况会出现`ConcurrentModificationException`"同步修改异常"
```
java.util.ConcurrentModificationException
```
出现该异常的原因是，当某个线程正在执行 `add()`方法时,被某个线程打断,添加到一半被打断,没有被添加完。

#### 解决ArrayList线程不安全问题
- 可以使用 `Vector` 来代替 `ArrayList`,`Vector` 是线程安全的 `ArrayList`,但是由于,并发量太小,被淘汰;
- 使用 `Collections.synchronizedArrayList()` 来创建 `ArrayList`；使用 `Collections` 工具类来创建 `ArrayList` 的思路是,在 `ArrayList` 的外边套了一个`synchronized`外壳,来使 `ArrayList` 线程安全;
- 使用 `CopyOnWriteArrayList()`来保证 `ArrayList` 线程安全；

下面详细说明`CopyOnWriteArrayList()`；使用`CopyOnWriteArrayList`演示代码
```
public class MainTest {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> arrayList = new CopyOnWriteArrayList<>();
        for(int i=0; i< 10; i++) {
            new Thread(() -> {
                arrayList.add(UUID.randomUUID().toString());
                System.out.println(arrayList);
            },String.valueOf(i)).start();
        }
    }
}
```
#### CopyWriteArrayList原理
`CopyWriteArrayList` 字面意思就是在写的时候复制,思想就是读写分离的思想。以下是 `CopyOnWriteArrayList` 的 `add()` 方法源码
```
/** The array, accessed only via getArray/setArray. */
    private transient volatile Object[] array;

/** The lock protecting all mutators */
    final transient ReentrantLock lock = new ReentrantLock();

  /**
     * Gets the array.  Non-private so as to also be accessible
     * from CopyOnWriteArraySet class.
     */
    final Object[] getArray() {
        return array;
    }

/**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```
`CopyWriteArrayList`之所以线程安全的原因是在源码里面使用 `ReentrantLock`，所以保证了某个线程在写的时候不会被打断；
可以看到源码开始先是复制了一份数组(因为同一时刻只有一个线程写,其余的线程会读),在复制的数组上边进行写操作,写好以后在返回 `true`。
这样写的就把读写进行了分离.写好以后因为 `array` 加了 `volatile` 关键字,所以该数组是对于其他的线程是可见的,就会读取到最新的值.


### HashSet
`HashSet` 和 `ArrayList` 类似,也是线程不安全的集合类。代码演示线程不安全示例，与`ArrayList`类似
```
public class MainTest {
    public static void main(String[] args) {
        HashSet<String> set = new HashSet<>();
        for(int i=0; i< 10; i++) {
            new Thread(() -> {
                set.add(UUID.randomUUID().toString());
                System.out.println(set);
            },String.valueOf(i)).start();
        }
    }
}
```
也会报 `java.util.ConcurrentModificationException` 异常。

参照`ArrayList`解决方案,`HashSet`有两种解决方案：
- `Collections.synchronizedSet()`使用集合工具类解决;
- 使用 `CopyOnWriteArraySet()`来保证集合线程安全;

使用 `CopyOnWriteArraySet()`代码演示
```
public class MainTest {
    public static void main(String[] args) {
        CopyOnWriteArraySet<String> set = new CopyOnWriteArraySet<>();
        for(int i=0; i< 10; i++) {
            new Thread(() -> {
                set.add(UUID.randomUUID().toString());
                System.out.println(set);
            },String.valueOf(i)).start();
        }
    }
}
```

`CopyOnWriteArraySet`底层调用的就是`CopyOnWriteArrayList`。
```
private final CopyOnWriteArrayList<E> al;
/**
 * Creates an empty set.
 */
public CopyOnWriteArraySet() {
    al = new CopyOnWriteArrayList<E>();
}
```
参照[CopyWriteArrayList原理。](#CopyWriteArrayList原理)

### HashMap
`HashMap` 也是线程不安全的集合类;
在多线程环境下使用同样会出现`java.util.ConcurrentModificationException`。
```
public class MainTest {
    public static void main(String[] args) {
        HashMap<String,Object> map = new HashMap<>();
        for(int i=0; i< 10; i++) {
            new Thread(() -> {
                map.put(UUID.randomUUID().toString(),Thread.currentThread().getName());
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }
}
```
再多线程环境下`HashMap`不仅会出现`ConcurrentModificationException`问题；
更严重的是，当多个线程中的 `HashMap` 同时扩容时，再使用put方法添加元素，如果hash值相同，可能出现同时在同一数组下用链表表示，造成闭环，导致在get时会出现死循环，CPU飙升到100%。

解决方案：
- 使用 `HashTable`来保证线程安全;
- `Collections.synchronizedMap()` 使用集合工具类;
- `ConcurrentHashMap<>()` 来保证线程安全;

上面的`HashTable`、`Collections.synchronizedMap()`因为性能的原因，在多线程环境下很少使用，一般都会使用`ConcurrentHashMap<>()`。

`HashTable`性能低的原因，就是直接加了`synchronized`修饰；
当使用put方法时，通过hash算法判断应该分配到哪一个数组上，如果分配到同一个数组上，即发生hash冲突，这个时候加锁是没问题的；但是一旦不发生hash冲突，再去加锁，性能就不太好了。

可理解为`HashTable`性能不好的原因就是锁的粒度太粗了。

`HashTable`put方法源码
```
public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
```

#### ConcurrentHashMap原理
`ConcurrentHashMap`原理简单理解为：`HashMap` + 分段锁。

因为`HashMap`在jdk1.7与jdk1.8结构上做了调整，所以`ConcurrentHashMap`在jdk1.7与jdk1.8结构上也有所不同。

在阅读之前建议掌握`HashMap`基本原理、CAS、`synchronized`、lock以及对多线程并发有一定了解。

##### jdk1.7ConcurrentHashMap
JDK1.7采用`segment`的分段锁机制实现线程安全，其中`segment`类继承自`ReentrantLock`。用`ReentrantLock`、CAS来保证线程安全。

![jdk1.7ConcurrentHashMap](/myblog/posts/images/essays/jdk1.7ConcurrentHashMap.png)

jdk1.7的`ConcurrentHashMap`结构：
- `segment`: 每一个`segment`数组就相当于一个`HashMap`；
- `HashEntry`: 等同于`HashMap`中`Entry`,用于存放K,V键值对；
- 节点：每个节点对应`ConcurrentHashMap`存放的值；

jdk1.7`ConcurrentHashMap`之所以能够保证线程安全，主要原因是在每个`segment`数组上加了锁，俗称分段锁，细化了锁的粒度。

jdk1.7`ConcurrentHashMap.put`方法源码
```
    public V put(K key, V value) {
        Segment<K,V> s;
        if (value == null)
            throw new NullPointerException();
        int hash = hash(key.hashCode());
        int j = (hash >>> segmentShift) & segmentMask;
        if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
             (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
            s = ensureSegment(j);
        return s.put(key, hash, value, false);
    }
```
首先判空，计算hash值，计算put进来的元素分配到哪个`segment`数组上，判断当前`segments`数组上的元素是否为空，如果为空就会使用`ensureSegment`方法创建`segment`对象；
最后调用`Segment.put`方法，存放到对应的节点中。

`Segment.ensureSegment`方法源码
```
/**
 * Returns the segment for the given index, creating it and
 * recording in segment table (via CAS) if not already present.
 *
 * @param k the index
 * @return the segment
 */
private Segment<K,V> ensureSegment(int k) {
        final Segment<K,V>[] ss = this.segments;
        long u = (k << SSHIFT) + SBASE; // raw offset
        Segment<K,V> seg;
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
            Segment<K,V> proto = ss[0]; // use segment 0 as prototype
            int cap = proto.table.length;
            float lf = proto.loadFactor;
            int threshold = (int)(cap * lf);
            HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
            if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                == null) { // recheck
                Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
                while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                       == null) {
                    if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                        break;
                }
            }
        }
        return seg;
    }
```
通过文档注释可以看到`ensureSegment`方法作用
> 返回指定索引的segment对象，通过CAS判断，如果还没有则创建它并记录在segment表中。

当多个线程同时执行该方法，同时通过`ensureSegment`方法创建`segment`对象时,只有一个线程能够创建成功；
其中创建的新`segment`对象中的加载因子、存放位置、扩容阈值与`segment[0]`元素保持一致。这样做性能更高，因为不用在计算了。

为了保证线程安全，在`ensureSegment`方法中用`Unsafe`类中的一些方法做了三次判断，其中最后一次也就是该方法保证线程安全的关键，用到了CAS操作;

当多个线程并发执行下面的代码，先执行CAS的线程，判断`segment`数组中某个位置是空的，然后就把这个线程自己创建的`segment`数组赋值给seg，即`seg = s`;然后`break`跳出循环；
后执行的线程会再次判断seg是否为空，因先执行的线程已经`seg = s`不为空了，所以循环条件不成立，也就不再执行了。
```
while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
       == null) {
    if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
        break;
}
```


`Segment.put`方法源码；为了保证线程安全，执行put方法要保证要加到锁，如果没加到锁就会执行`scanAndLockForPut`方法；
这个方法就会保证一定要加到锁；
```
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    HashEntry<K,V> node = tryLock() ? null :
        scanAndLockForPut(key, hash, value);
    // ... 插入节点操作 最后释放锁
}
```
`scanAndLockForPut`方法的主要作用就是加锁，如果没有获取锁，就会一致遍历`segment`数组，直到遍历到最后一个元素；
每次遍历完都会尝试获取锁，如果还是获取不到锁，就会重试，最大次数为`MAX_SCAN_RETRIES`在CPU多核下为64次，如果大于64次就会强制加锁。
```
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1; // negative while locating node
    while (!tryLock()) {
        HashEntry<K,V> f; // to recheck first below
        if (retries < 0) {
            if (e == null) {
                if (node == null) // speculatively create node
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                retries = 0;
            else
                e = e.next;
        }
        else if (++retries > MAX_SCAN_RETRIES) {
            lock();
            break;
        }
        else if ((retries & 1) == 0 &&
                 (f = entryForHash(this, hash)) != first) {
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    return node;
}

static final int MAX_SCAN_RETRIES =
            Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;

```

##### [jdk1.8ConcurrentHashMap](https://www.jianshu.com/p/c0642afe03e0)
JDK1.8的实现已经摒弃了 `Segment` 的概念，而是直接用 `Node数组+链表/红黑树`的数据结构来实现，并发控制使用 `synchronized` 和CAS来操作，整个看起来就像是优化过且线程安全的`HashMap`;
虽然在JDK1.8中还能看到 `Segment` 的数据结构，但是已经简化了属性，只是为了兼容旧版本。

JDK1.8中彻底放弃了`Segment`转而采用的是`Node`,其设计思想也不再是JDK1.7中的分段锁思想；
JDK1.8版本的`ConcurrentHashMap`的数据结构已经接近`HashMap`，相对而言，`ConcurrentHashMap` 只是增加了同步操作来控制并发。

![jdk1.8ConcurrentHashMap](/myblog/posts/images/essays/jdk1.8ConcurrentHashMap.png)

相关概念：
- `sizeCtl` ：默认为0，用来控制`table`的初始化和扩容操作;用`volatile`修饰，保证了其可见性；


JDK1.8`ConcurrentHashMap.put`方法源码;
```
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```
首先调用`Node.initTable()`方法，初始化table;`sizeCtl` 默认为0，如果`ConcurrentHashMap`实例化时有传参数，`sizeCtl` 会是一个2的幂次方的值。
所以执行第一次put方法时操作的线程会执行`Unsafe.compareAndSwapInt`方法修改`sizeCtl=-1`，只有一个线程能够修改成功，其它线程通过`Thread.yield()`礼让线程让出CPU时间片，等待`table`初始化完成。

```
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```
调用put方法，通过hash算法计算，将要存放数组中的位置`(n - 1) & hash`，如果该节点为空就通过CAS判断，创建一个Node放到该位置上。
```
int hash = spread(key.hashCode());

// hash算法，计算存放在map中的位置；要保证尽可能的均匀分散，避免hash冲突
static final int HASH_BITS = 0x7fffffff;
static final int spread(int h) {
    // 等同于： key.hashCode() ^ (key.hashCode() >>> 16) & 0x7fffffff
    return (h ^ (h >>> 16)) & HASH_BITS;
}
```
如果该位置不为空就会继续判断当前线程的`ConcurrentHashMap`是否进行扩容
```
// MOVED = -1
if ((fh = f.hash) == MOVED)
tab = helpTransfer(tab, f);
```
插入之前，再次利用`tabAt(tab, i) == f`判断，防止被其它线程修改;
之后就会对这个将要添加到该位置的元素加锁，判断是链表还是树节点，做不同的操作;
- 如果`f.hash >= 0`，说明f是链表结构的头结点，遍历链表，如果找到对应的`node`节点，则修改`value`，否则在链表尾部加入节点。
- 如果f是`TreeBin`类型节点，说明f是红黑树根节点，则在树结构上遍历元素，更新或增加节点。
- 如果链表中节点数`binCount >= TREEIFY_THRESHOLD(默认是8)`，则把链表转化为红黑树结构。

```
V oldVal = null;
synchronized (f) {
    if (tabAt(tab, i) == f) {
        if (fh >= 0) {
            binCount = 1;
            for (Node<K,V> e = f;; ++binCount) {
                K ek;
                if (e.hash == hash &&
                    ((ek = e.key) == key ||
                     (ek != null && key.equals(ek)))) {
                    oldVal = e.val;
                    if (!onlyIfAbsent)
                        e.val = value;
                    break;
                }
                Node<K,V> pred = e;
                if ((e = e.next) == null) {
                    pred.next = new Node<K,V>(hash, key,
                                              value, null);
                    break;
                }
            }
        }
        else if (f instanceof TreeBin) {
            Node<K,V> p;
            binCount = 2;
            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                           value)) != null) {
                oldVal = p.val;
                if (!onlyIfAbsent)
                    p.val = value;
            }
        }
    }
}
if (binCount != 0) {
    if (binCount >= TREEIFY_THRESHOLD)
        treeifyBin(tab, i);
    if (oldVal != null)
        return oldVal;
    break;
}
```
最后则进行扩容操作
```
//相当于size++
addCount(1L, binCount);
```
```
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    if ((as = counterCells) != null ||
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        CounterCell a; long v; int m;
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            !(uncontended =
              U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        s = sumCount();
    }
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}
```
节点从`table`移动到`nextTable`，大体思想是遍历、复制的过程。
通过`Unsafe.compareAndSwapInt`修改`sizeCtl`值，保证只有一个线程能够初始化`nextTable`，扩容后的数组长度为原来的两倍，但是容量是原来的1.5。

- 首先根据运算得到需要遍历的次数i，然后利用`tabAt`方法获得i位置的元素f，初始化一个`forwardNode`实例fwd。
- 如果`f == null`，则在`table`中的i位置放入fwd，这个过程是采用`Unsafe.compareAndSwapObjectf`方法实现的，实现了节点的并发移动。
- 如果f是链表的头节点，就构造一个反序链表，把他们分别放在`nextTable`的i和i+n的位置上，移动完成，采用`Unsafe.putObjectVolatile`方法给`table`原位置赋值fwd。
- 如果f是`TreeBin`节点，也做一个反序处理，并判断是否需要`untreeify`，把处理的结果分别放在nextTable的i和i+n的位置上，移动完成，同样采用`Unsafe.putObjectVolatile`方法给`table`原位置赋值fwd。



