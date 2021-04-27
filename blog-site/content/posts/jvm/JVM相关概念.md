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

