---
title: "JVM相关概念"
date: 2021-04-27
draft: false
tags: ["Java", "JVM"]
slug: "jvm-about"
---

## 内存溢出
内存溢出(Out Of Memory，简称OOM)是指应用系统中存在无法回收的内存或使用的内存过多，最终使得程序运行要用到的内存大于能提供的最大内存。
官方文档中对内存溢出的解释是，没有空闲内存，并且垃圾收集器也无法提供更多内存。

由于GC一直在发展，所有一般情况下，除非应用程序占用的内存增长速度非常快，造成垃圾回收已经跟不上内存消耗的速度，否则不太容易出现OOM的情况。

引起内存溢出的原因：
- Java虚拟机的堆内存设置不够。
- 代码中创建了大量大对象，并且长时间不能被垃圾收集器收集。

OOM异常信息变化：
- JDK7及之前:`java.lang.OutOfMemoryError:PermGen space`
- JDK8:`java.lang.OutofMemoryError:Metaspace`

在抛出`OutOfMemoryError`之前，通常垃圾收集器会被触发，尽其所能去清理出空间。当然，不是在任何情况下垃圾收集器都会被触发的：
比如，我们去分配一个超大对象，类似一个超大数组超过堆的最大值，JVM可以判断出垃圾收集并不能解决这个问题，所以直接抛出`OutOfMemoryError`。

## 内存泄漏
严格来说，只有对象不会再被程序用到了，但是GC又不能回收他们的情况，才叫内存泄漏。
但实际情况很多时候一些不太好的实践（或疏忽）会导致对象的生命周期变得很长甚至导致00M，也可以叫做宽泛意义上的“内存泄漏”。

Java使用可达性分析算法来标记垃圾，最上面的数据不可达，就是需要被回收的。
后期有一些对象不用了，按道理应该断开引用，但是存在一些链没有断开，从而导致没有办法被回收。从而造成内存泄漏。

![内存泄漏](/myblog/posts/images/essays/内存泄漏.png)

**内存泄漏与内存溢出的关系**

尽管内存泄漏并不会立刻引起程序崩溃，但是一旦发生内存泄漏，程序中的可用内存就会被逐步蚕食，直至耗尽所有内存，最终出现`outOfMemory`异常，导致程序崩溃。

### 举例
> 买房子：80平的房子，但是有10平是公摊的面积，我们是无法使用这10平的空间，这就是所谓的内存泄漏

- 单例模式；单例的生命周期和应用程序是一样长的，所以单例程序中，如果单例对象持有对外部对象的引用的话，而外部对象引用却不再使用，那么这个外部对象是不能被回收的，则会导致内存泄漏的产生。

- 提供`close`的资源未关闭导致内存泄漏;数据库连接，网络连接和IO连接必须手动，否则是不能被回收的。

## STW
`Stop-the-world`直译为：停止一切，简称STW，指的是垃圾回收发生过程中，会产生应用程序的停顿。
停顿产生时整个应用程序线程都会被暂停，没有任何响应，有点像卡死的感觉。

STW事件和采用哪款GC无关，因为所有的GC都有这个事件。任何垃圾回收器都不能完全避免`Stop-the-world`情况发生，只能说垃圾回收器越来越优秀，回收效率越来越高，尽可能地缩短了暂停时间。

**为什么垃圾回收时要STW？**

在垃圾回收标记阶段，JVM使用可达性分析算法进行标记那些对象是垃圾，如果出现分析过程中对象引用关系还在不断变化，则分析结果的准确性无法保证。
所以在垃圾回收的时候要STW，分析工作必须在一个能确保一致性的快照中进行。

STW是JVM在后台自动发起和自动完成的。在用户不可见的情况下，把用户正常的工作线程全部停掉。
被STW中断的应用程序线程会在完成GC之后恢复，频繁中断会让用户感觉像是网速不快造成电影卡带一样，所以我们需要减少STW的发生。

## System.gc
在默认情况下，通过`System.gc()`者`Runtime.getRuntime().gc()`的调用，会显式触发FullGC，同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存。

源码调用了`Runtime.getRuntime().gc();`

大意为：
> 运行垃圾收集器。
调用GC方法意味着Java虚拟机要努力回收未使用的对象，以便使它们当前占用的内存能够快速重用。当控制从方法调用中返回时，Java虚拟机已经尽了最大努力从所有丢弃的对象中回收空间。
调用`System.gc()`有效地等同于调用:`Runtime.getRuntime().gc()`
```
    /**
     * Runs the garbage collector.
     * <p>
     * Calling the <code>gc</code> method suggests that the Java Virtual
     * Machine expend effort toward recycling unused objects in order to
     * make the memory they currently occupy available for quick reuse.
     * When control returns from the method call, the Java Virtual
     * Machine has made a best effort to reclaim space from all discarded
     * objects.
     * <p>
     * The call <code>System.gc()</code> is effectively equivalent to the
     * call:
     * <blockquote><pre>
     * Runtime.getRuntime().gc()
     * </pre></blockquote>
     *
     * @see     java.lang.Runtime#gc()
     */
    public static void gc() {
        Runtime.getRuntime().gc();
    }
```
调用`System.gc();`无法保证对垃圾收集器的调用;一般情况下，垃圾回收应该是自动进行的，无须手动触发。

代码演示是否触发GC
```
// 在线程不忙的情况下，GC几乎都会执行都会调用finalize()方法 多试几次(15~30)
public class MainTest {
    public static void main(String[] args) {
        new MainTest();

        // 建议垃圾回收器执行垃圾收集行为 不会立即执行
        System.gc();

        // 调用System.gc();后再调用System.runFinalization(); 会强制调用失去引用对象的 finalize() 方法
//        System.runFinalization();
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize 被调用 ...");
    }
}
```


## 垃圾回收的的串行、并行、并发

> PS: 并行、并发、串行
>- 并行：前提是在多核CPU或多个CPU条件下，多个线程同时被多个CPU执行，同时执行的线程并不会抢占CPU资源。
>- 串行：前提是在单核CPU条件下，单线程程序执行，不能同时执行，也不能去切换执行。也就是在同一时间段只能做一件事。
>- 并发：前提是多线程条件下，多个线程抢占一个CPU资源，多个线程被交替执行。因为CPU运算速度很快，所以用户感觉不到线程切换的卡顿。

无论并行、并发，都可以有多个线程执行，如果是多个线程抢占一个CPU就成了并发，多个CPU同时执行多个线程就是并行。

对于单CPU的计算机来说，同一时间是只能干一件事儿的，如果是单线程线程就是串行；如果是多个线程就是并发。
而对于多CPU的计算机说，同一时间能干多个事，如果多个CPU同时执行多个线程就是并行；如果一个CPU同时执行多个线程就是并行。

- 并行垃圾回收：在停止用户线程之后，多条GC线程并行进行垃圾回收，此时用户线程仍处于等待状态，出现STW现象。

- 并发垃圾回收：指多条垃圾收集线程同时进行工作，GC线程和用户线程同时运行，不会出现STW现象。

- 串行垃圾回收：在同一时间段内只允许有一个CPU用于执行垃圾回收操作，该收集器会在工作时冻结所有应用程序线程，这使它在所有目的和用途上都无法在服务器环境中使用。


## 安全点与安全区域

### 安全点
程序执行时并非在所有地方都能停顿下来开始GC，只有在特定的位置才能停顿下来开始GC，这些位置称为安全点。

安全点的选择很重要，如果太少可能导致GC等待的时间太长，如果太频繁可能导致运行时的性能问题。
大部分指令的执行时间都非常短暂，通常会根据“是否具有让程序长时间执行的特征”为标准。比如：选择一些执行时间较长的指令作为安全点。

**如何在GC发生时，检查所有线程都跑到最近的安全点停顿下来呢？**
- 抢先式中断：首先中断所有线程。如果还有线程不在安全点，就恢复线程，让线程跑到安全点。目前没有虚拟机采用了；
- 主动式中断：设置一个中断标志，各个线程运行到安全点的时候主动轮询这个标志，如果中断标志为真，则将自己进行中断挂起。

### 安全区域
安全点机制保证了程序执行时，在不太长的时间内就会遇到可进入GC的安全点。
但是，程序“不执行”的时候呢？例如线程处于`sleep`状态或`Blocked`状态，这时候线程无法响应JVM的中断请求，“走”到安全点去中断挂起，JVM也不太可能等待线程被唤醒。
对于这种情况，就需要安全区域来解决。

安全区域是指在一段代码片段中，对象的引用关系不会发生变化，在这个区域中的任何位置开始Gc都是安全的。
可以把安全点看做是被扩展了的安全区域。

- 当线程运行到安全区域的代码时，首先标识已经进入了安全区域，如果这段时间内发生GC，JVM会忽略标识为安全区域状态的线程
- 当线程即将离开安全区域时，会检查JVM是否已经完成GC，如果完成了，则继续运行，否则线程必须等待直到收到可以安全离开安全区域的信号为止；

## 引用
在JDK1.2版之后，Java对引用的概念进行了扩充，将引用分为：
- 强引用（StrongReference）：最传统的“引用”的定义；无论任何情况下，只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象。
- 软引用（SoftReference）：在系统将要发生内存溢出之前，将会把这些对象列入回收范围之中进行第二次回收。如果这次回收后还没有足够的内存，才会抛出内存溢出异常。
- 弱引用（WeakReference）：被弱引用关联的对象只能生存到下一次垃圾收集之前。当垃圾收集器工作时，无论内存空间是否足够，都会回收掉被弱引用关联的对象。
- 虚引用（PhantomReference）：一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获得一个对象的实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。

这4种引用强度依次逐渐减弱。除强引用外，其他3种引用均可以在`java.lang.ref`包中找到它们的身影。
强引用为JVM内部实现。其他三类引用类型全部继承自`Reference`父类。

![强软弱虚](/myblog/posts/images/essays/强软弱虚.jpg)

上述引用垃圾回收的前提条件是：对象都是可触及的(可达性分析结果为可达)，如果对象不可触及就直接被垃圾回收器回收了。
### 强引用
在Java程序中，最常见的引用类型是强引用，普通系统99%以上都是强引用，也就是我们最常见的普通对象引用，也是默认的引用类型。
当在Java语言中使用new操作符创建一个新的对象，并将其赋值给一个变量的时候，这个变量就成为指向该对象的一个强引用。

强引用测试
```
// 强引用测试
public class MainTest {
    public static void main(String[] args) {
        StringBuffer var0 = new StringBuffer("hello world");
        StringBuffer var1 = var0;

        var0 = null;
        System.gc();
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(var1.toString());
    }
}
```
强引用所指向的对象在任何时候都不会被系统回收，虚拟机会抛出OOM异常，也不会回收强引用所指向对象;
所以强引用是导致内存泄漏的主要原因。

### 软引用
软引用是一种比强引用生命周期稍弱的一种引用类型。在JVM内存充足的情况下，软引用并不会被垃圾回收器回收，只有在JVM内存不足的情况下，才会被垃圾回收器回收。

软引用是用来描述一些还有用，但非必需的对象。
只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行**第二次回收**，如果这次回收还没有足够的内存，才会抛出内存溢出异常。
> 这里的第一次回收是指不可达的对象

所以软引用一般用来实现一些内存敏感的缓存，只要内存空间足够，对象就会保持不被回收掉。
比如：高速缓存就有用到软引用。如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。

软引用测试
```
/**
 * 软引用测试
 * 
 * 虚拟机参数：
 * -Xms10m
 * -Xmx10m
 * -XX:+PrintGCDetails
 */
public class MainTest {
    public static void main(String[] args) {
        //SoftReference<User> softReference = new SoftReference<>(new User("hello"));
        // 上面的一行代码等价于下面的三行代码
        User user = new User("hello");
        SoftReference<User> softReference = new SoftReference<User>(user);
        // 一定要销毁强引用对象 否则创建软引用对象将毫无意义
        user = null;
        System.out.println("创建大对象之前：" + softReference.get());
        try{
            // 模拟堆内存资源紧张 看软引用对象是否会被回收
            byte[] bytes = new byte[1024 * 1024 *7];
        }catch (Throwable e) {
            e.printStackTrace();
        }finally {
            System.out.println("创建大对象之后：" + softReference.get());
        }
    }
}

class User {
    private String name;

    public User(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}

```
### 弱引用
弱引用也是用来描述那些非必需对象，被弱引用关联的对象只能生存到下一次垃圾收集发生为止。

在系统GC时，只要发现弱引用，不管系统堆空间使用是否充足，都会回收掉只被弱引用关联的对象。
由于垃圾回收器的线程通常优先级很低，因此，并不一定能很快地发现持有弱引用的对象。在这种情况下，弱引用对象可以存在较长的时间。

弱引用和软引用一样，在构造弱引用时，也可以指定一个引用队列，当弱引用对象被回收时，就会加入指定的引用队列，通过这个队列可以跟踪对象的回收情况。

软引用、弱引用都非常适合来保存那些可有可无的缓存数据。如果这么做，当系统内存不足时，这些缓存数据会被回收，不会导致内存溢出。
而当内存资源充足时，这些缓存数据又可以存在相当长的时间，从而起到加速系统的作用。

弱引用测试
```
/**
 * 弱引用测试
 */
public class MainTest {
    public static void main(String[] args) {
        WeakReference<User> weakReference = new WeakReference<>(new User("hello"));
        System.out.println("建议GC之前：" + weakReference.get());
        System.gc();
        System.out.println("建议GC之后：" + weakReference.get());
    }
}

class User {
    private String name;

    public User(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}
```
### 虚引用
虚引用也称为“幽灵引用”或者“幻影引用”，是所有引用类型中最弱的一个。

一个对象是否有虚引用的存在，完全不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它和没有引用几乎是一样的，随时都可能被垃圾回收器回收。
它不能单独使用，也无法通过虚引用来获取被引用的对象。当试图通过虚引用的`get（）`方法取得对象时，总是`null`;

为一个对象设置虚引用关联的唯一目的在于跟踪垃圾回收过程。比如：能在这个对象被收集器回收时收到一个系统通知。
**虚引用必须和引用队列一起使用。** 虚引用在创建时必须提供一个引用队列作为参数。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象后，将这个虚引用加入引用队列，以通知应用程序对象的回收情况。
由于虚引用可以跟踪对象的回收时间，因此，也可以将一些资源释放操作放置在虚引用中执行和记录。

虚引用测试
```
/**
 * 虚引用测试
 */
public class MainTest {
    // 当前类对象的声明
    public static MainTest obj;
    // 引用队列
    static ReferenceQueue<MainTest> phantomQueue = null;

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("调用当前类的finalize方法");
        obj = this;
    }

    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            while(true) {
                if (phantomQueue != null) {
                    PhantomReference<MainTest> objt = null;
                    try {
                        objt = (PhantomReference<MainTest>) phantomQueue.remove();
                    } catch (Exception e) {
                        e.getStackTrace();
                    }
                    if (objt != null) {
                        System.out.println("追踪垃圾回收过程：PhantomReferenceTest实例被GC了");
                    }
                }
            }
        }, "t1");
        thread.setDaemon(true);
        thread.start();

        phantomQueue = new ReferenceQueue<>();
        obj = new MainTest();
        // 构造了PhantomReferenceTest对象的虚引用，并指定了引用队列
        PhantomReference<MainTest> phantomReference = new PhantomReference<>(obj, phantomQueue);
        try {
            System.out.println(phantomReference.get());
            // 去除强引用
            obj = null;
            // 第一次进行GC，由于对象可复活，GC无法回收该对象
            System.out.println("第一次GC操作");
            System.gc();
            Thread.sleep(1000);
            if (obj == null) {
                System.out.println("obj 是 null");
            } else {
                System.out.println("obj 不是 null");
            }
            System.out.println("第二次GC操作");
            obj = null;
            System.gc();
            Thread.sleep(1000);
            if (obj == null) {
                System.out.println("obj 是 null");
            } else {
                System.out.println("obj 不是 null");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {

        }
    }
}
```



