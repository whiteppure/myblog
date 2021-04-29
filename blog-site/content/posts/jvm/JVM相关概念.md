---
title: "JVM相关概念"
date: 2021-04-27
draft: false
tags: ["Java", "JVM"]
slug: "jvm-about"
---

## System.gc
在默认情况下，通过`System.gc()`者`Runtime.getRuntime().gc()`的调用，会显式触发FullGC，同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存。

源码调用了`Runtime.getRuntime().gc();`方法
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


## 并行、并发、串行
- 如果CPU多核，每个CPU核心上执行一个任务，在某一个时间点，这些任务是同时执行的，这叫并行；
- 如果CPU单核，在一个核心上执行多个任务，只有一个线程，每个任务都在这个线程上都按照顺序来执行，这叫串行；
- 如果CPU单核，在这一个核心上执行多个任务，同时用多个线程执行这些任务，CPU不断的去切换这些线程（因为CPU处理非常快，我们感觉这些任务好像是同时执行的）这叫并发。

### 并发
在操作系统中，指一个时间段中有几个程序都处于运行状态，且这几个程序都是在同一个(进程)处理器上运行。
然后CPU通过时间片轮询这几个不同线程上的程序，由于CPU处理的速度非常快，只要时间间隔处理得当，用户感觉是多个应用程序同时在进行。

并发的多个任务之间是互相抢占资源的。
### 并行
当CPU有一个以上核心时，当一个CPU核心执行一个进程时，另一个CPU核心可以执行另一个进程，两个进程互不抢占CPU资源，可以同时进行。

并行的多个任务之间是不互相抢占资源的。只有在多CPU或者一个CPU多核的情况中，才会发生并行。
### 串行
相较于并行的概念，单线程程序，代码按照从上到下的顺序来执行。

### 垃圾回收的的并行与并发
- 并行：指多条垃圾收集线程同时进行工作，垃圾回收线程在执行时不会停顿用户程序的运行。
- 并发：指用户线程与垃圾收集线程同时交替执行，但此时用户线程仍处于等待状态，出现STW现象。





 



