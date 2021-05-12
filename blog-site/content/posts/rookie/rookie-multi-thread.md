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

自旋锁的实现原理是[CAS](),例如`AtomicInteger`中`getAndUpdate()`方法
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


### 共享锁与排它锁
### 轻量级锁与重量级锁

### 偏向锁
### 读写锁
### 互斥锁
### 独占锁


## 线程安全
### 三大特性
原子性、可见性、有序性

### volatile
### CAS
CAS全称为`Compare and Swap`被译为比较并交换。在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。
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
上面的三个方法可以对应去查看 `openjdk` 的 `hotspot`源码：`src/share/vm/prims/unsafe.cpp`
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


CAS作用也一目了然，在多线程环境中，就是比较当前线程工作内存中的值和主内存中的值，如果相同则执行规定操作，否则继续比较，直到主内存和工作内存中的值一直为止。例如代码：
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

从源码可以看出，是通过CPU指令进行调用,当CPU中某个处理器对缓存中的共享变量进行了操作，其他处理器会有个嗅探机制，将其他处理器的该共享变量的缓存失效，待其他线程读取时会重新从主内存中读取最新的数据，基于 `MESI` 缓存一致性协议来实现的。

简述，就是通过CPU的缓存一致性协议来保证线程之间的数据一致性的。

> CPU 处理器速度远远大于在主内存中的，为了解决速度差异，在他们之间架设了多级缓存，如 L1、L2、L3 级别的缓存，这些缓存离CPU越近就越快，将频繁操作的数据缓存到这里，加快访问速度。

![JMM](/myblog/posts/images/essays/JMM.jpg)


#### CAS与Unsafe关系
CAS的作用是比较并交换,就是先拿这个期望值,与主内存的值比较,判断主内存中该位置是否存在期望值,
如果存在,则改为新的值,这个修改的过程是具有原子性的.
因为CAS是cpu并发源语,并发源语体现在Java sun.misc.Unsafa类上.
调用Unsafe类中的CAS方法，JVM会帮我们实现CAS汇编指令。这是一种完全依赖于硬件的功能，通过他实现了原子操作。
由于CAS是一种系统原语，原语属于操作系统用语范畴，是由若干条指令组成的，用于完成某个功能的一个过程，并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU的原子指令，不会造成数据不一致问题。

> PS Unsafe类
CAS其实是调用了 `Unsafe` 类的方法 `Unsafa` 类是CAS核心类，由于Java方法无法直接访问底层系统，需要通过本地（`native`）方法来访问，`Unsafe` 相当于一个后门，基于该类可以直接操作特定内存数据。
Unsafe类存在于sun.misc包中，其内部方法操作可以像C的指针(内存地址)一样直接操作内存，因此Java中CAS操作的执行依赖于Unsafe类的方法。
Unsafe类中的所有方法都是**native**修饰的，也就是说Unsafe类中的方法都直接**调用操作系统底层资源执行相应任务**。

#### 缺点
1. 因为是采用自旋锁的方式来实现所以，自然有自旋锁的缺点，循环时间长开销大,例如：`getAndAddInt` 方法执行，有个`do while`循环，如果CAS失败，一直会进行尝试，如果CAS长时间不成功，可能会给CPU带来很大的开销。
    ```
    public final int getAndAddInt(Object var1, long var2, int var4) {
            int var5;
            do {
                var5 = this.getIntVolatile(var1, var2);
            } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
            return var5;
    }
    ```
2. 只能保证一个共享变量的原子操作，对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁来保证原子性。
3. ABA问题。

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
    
    3.1 **ABA问题解决**
    
    3.1.1 利用 `AtomicReference` 类进行原子引用
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
    
    3.1.2 使用时间戳的原子引用`AtomicStampedReference`修改版本号。主要是在对象中额外再增加一个标记来标识对象是否有过变更
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


### reentrantLock
### AQS

### synchronized

## 线程安全的集合
### List
### Set
### Map




