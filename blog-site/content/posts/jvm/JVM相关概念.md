---
title: "JVM相关概念"
date: 2021-04-27
draft: false
tags: ["Java", "JVM"]
slug: "jvm-about"
---

## System.gc()
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

