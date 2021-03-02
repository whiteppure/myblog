---
title: "Java中常用到的锁"
date: 2020-04-07
draft: false
tags: ["线程", "Java"]
slug: "java-lock"
---

## 公平锁
指多个线程按照申请锁的顺序来获取锁类似排队打饭 先来后到
- **优点:** 所有的线程都能得到资源，不会饿死在队列中。
- **缺点:** 吞吐量会下降很多，队列里面除了第一个线程，其他的线程都会阻塞，cpu唤醒阻塞线程的开销会很大。

## 非公平锁
指在多线程获取锁的顺序并不是按照申请锁的顺序,有可能后申请的线程比先申请的线程优先获取到锁,在高并发的情况下,有可能造成**优先级反转**或者**饥饿现象**
- **优点:** 可以减少CPU唤醒线程的开销，整体的吞吐效率会高点，CPU也不必取唤醒所有线程，会减少唤起线程的数量。
- **缺点:** 你们可能也发现了，这样可能导致队列中间的线程一直获取不到锁或者长时间获取不到锁，导致饿死.

并发包```ReentrantLock```的创建可以指定构造函数的boolean类型来得到公平锁或者非公平锁 默认是非公平锁,```synchronized```也是非公平锁.
源码如下:
```
   /**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

## 可重入锁(递归锁)
指同一个线程外层函数获得锁之后，内层仍然能获取到该锁，在同一个线程在外层方法获取锁的时候，在进入内层方法或会自动获取该锁.
**可重入锁最大的作用就是避免死锁**

## 不可重入锁
所谓不可重入锁，即若当前线程执行某个方法已经获取了该锁，那么在方法中尝试再次获取锁时，就会获取不到被阻塞

- **举个栗子:** 当你进入你家时门外会有锁,进入房间后厨房卫生间都可以随便进出,这个叫可重入锁.当你进入房间时,发现厨房,卫生间都有上锁.这个叫不可重入锁

- **以下的代码证明** ```synchronized```**和** ```ReentrantLock```**都是可重入锁**

### synchronized
 ```
public class SynchronziedDemo {

    private synchronized void print() {
        doAdd();
    }
    private synchronized void doAdd() {
        System.out.println("doAdd...");
    }

    public static void main(String[] args) {
        SynchronziedDemo synchronziedDemo = new SynchronziedDemo();
        synchronziedDemo.print(); // doAdd...
    }
}
```
### ReentrantLock
```
public class ReentrantLockDemo {
    private Lock lock = new ReentrantLock();

    private void print() {
        lock.lock();
        doAdd();
        lock.unlock();
    }

    private void doAdd() {
	// doAdd 方法中加两次锁和解两次锁也可以,不会报错
	// 如果少了一个unlock()也不会报错,会形成死锁
        lock.lock();
        lock.lock();
        System.out.println("doAdd...");
        lock.unlock();
        lock.unlock();
    }

    public static void main(String[] args) {
        ReentrantLockDemo reentrantLockDemo = new ReentrantLockDemo();
        reentrantLockDemo.print();
    }
}

```
## 自旋锁
自旋锁（spin lock）是一种非阻塞锁，也就是说，如果某线程需要获取锁，但该锁已经被其他线程占用时，该线程不会被挂起，而是在不断的消耗CPU的时间，不停的试图获取锁.这样的好处是减少线程上线文切换的消耗，缺点就是循环会消耗 CPU.原理是[CAS原理](./../cas-principle)
```
public class SpinLock {
    private AtomicReference<Thread> atomicReference = new AtomicReference<>();
    private void lock () {
        System.out.println(Thread.currentThread() + " coming...");
        while (!atomicReference.compareAndSet(null, Thread.currentThread())) {
            // loop
        }
    }

    private void unlock() {
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread, null);
        System.out.println(thread + " unlock...");
    }

    public static void main(String[] args) throws InterruptedException {
        SpinLock spinLock = new SpinLock();
        new Thread(() -> {
            spinLock.lock();
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("hahaha");
            spinLock.unlock();

        }).start();

        Thread.sleep(1);

        new Thread(() -> {
            spinLock.lock();
            System.out.println("hehehe");
            spinLock.unlock();
        }).start();
    }
}

```
```
Thread[Thread-0,5,main] coming...
Thread[Thread-1,5,main] coming...
hahaha
Thread[Thread-0,5,main] unlock...
hehehe
Thread[Thread-1,5,main] unlock...
```
## 独占锁(写锁)
指该锁一次只能被一个线程独占,所持有.对于````synchronized````和````ReentrantLock````而言都是独占锁

## 共享锁(读锁)
该锁可以被多个线程持有.对于 ```ReentrantLock```和 ```synchronized```都是独占锁；对与 ```ReentrantReadWriteLock ```其读锁是共享锁而写锁是独占锁。
读锁的共享可保证并发读是非常高效的，**读写、写读和写写的过程是互斥的.**

**读写锁案例**

`ReentrantReadWriteLock 能保证读写、写读和写写的过程是互斥的时候是独享的，读读的时候是共享的
```
class MyCache {

    private volatile Map<String, Object> map = new HashMap<>();

    private ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();

    public void put(String key, Object value) {
        rwLock.writeLock().lock();
        try {
            System.out.println("开始 写入 ..." + key);
            map.put(key, value);
            System.out.println("写入完成 ...");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            rwLock.writeLock().unlock();
        }

    }

    public Object get(String key) {
        Object obj = null;
        rwLock.writeLock().lock();
        try {
            System.out.println("开始读取 ..." + key);
            obj = map.get(key);
            System.out.println("读取完成 ..." + obj);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            rwLock.writeLock().unlock();
        }
        return obj;
    }

}

public class Test {

    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(() -> {
                myCache.put(finalI + "", finalI + "");
            }, String.valueOf(i)).start();
        }

        System.out.println("---------------");

        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(() -> {
                myCache.get(finalI + "");
            }, String.valueOf(i)).start();
        }
    }
}

```
```
开始 写入 ...0
写入完成 ...
开始 写入 ...1
写入完成 ...
开始 写入 ...2
写入完成 ...
开始 写入 ...3
写入完成 ...
开始 写入 ...4
写入完成 ...
开始 写入 ...6
写入完成 ...
开始 写入 ...5
写入完成 ...
开始 写入 ...7
写入完成 ...
开始读取 ...0
读取完成 ...0
开始读取 ...3
读取完成 ...3
开始读取 ...4
读取完成 ...4
开始读取 ...5
读取完成 ...5
开始读取 ...7
读取完成 ...7
开始读取 ...9
读取完成 ...null
开始 写入 ...8
写入完成 ...
开始读取 ...1
读取完成 ...1
开始读取 ...2
读取完成 ...2
开始读取 ...6
读取完成 ...6
开始读取 ...8
读取完成 ...8
开始 写入 ...9
写入完成 ...
```




