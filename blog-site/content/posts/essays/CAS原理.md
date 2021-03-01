---
title: "CAS原理"
date: 2020-04-04
draft: false
tags: ["线程"]
slug: "cas-principle"
---

### CAS
**CAS(Compare and Swap),翻译成比较并交换.**
CAS是java.util.concurrent.atomic包的基础，即AtomicInteger和AtomicLong等是用CAS实现的
```java
/* 
 *expect 期望值
 *update 更新值
 */
 public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset,expect, update);
 }
```
```java
AtomicInteger number = new AtomicInteger(5);
````
例如:现在主物理内存(堆)number是5(如上代码),线程工作的时候会复制主物理内存的5(期望值)到自己的工作内存中,(这里不懂的可以去看看JMM,Java内存模型),此时该线程要把number改为6(更新值),主物理内存的number值和线程中的number一致,返回true,如果在这之前有其他的线程修改过主物理内存number的值,则主物理内存中的number值和线程中的不一致,会一致循环,直到线程中的number和主物理内存中number的值一样.
**简而言之比就是比较当前工作内存中的值和主内存中的值，如果相同则执行规定操作，否则继续比较，直到主内存和工作内存中的值一直为止**

````java
public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
        return var5;
}
````
### CAS与Unsafe关系
CAS的作用是比较并交换,就是先拿这个期望值,与主内存的值比较,判断主内存中该位置是否存在期望值,如果存在,则改为新的值,这个修改的过程是具有原子性的.因为CAS是cpu并发源语,并发源语体现在Java sun.misc.Unsafa类上.调用Unsafe类中的CAS方法，JVM会帮我们实现CAS汇编指令。这是一种完全依赖于硬件的功能，通过他实现了原子操作。由于CAS是一种系统原语，原语属于操作系统用语范畴，是由若干条指令组成的，用于完成某个功能的一个过程，并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU的原子指令，不会造成数据不一致问题。

### Unsafe
能看到上边的源码CAS其实是调用了Unsafa类的方法.Unsafa类是CAS核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，Unsafe相当于一个后门，基于该类可以直接操作特定内存数据。Unsafe类存在于sun.misc包中，其内部方法操作可以像C的指针(内存地址)一样直接操作内存，因此Java中CAS操作的执行依赖于Unsafe类的方法.
 - Unsafe类中的所有方法都是**native**修饰的，也就是说Unsafe类中的方法都直接**调用操作系统底层资源执行相应任务**
- 变量**valueOffset**，表示该变量值在**内存中的偏移量(内存地址)**，因为Unsafe就是根据内存偏移量获取数据的
- 变量value用volatile修饰，保证多线程之间的可见性              

### CAS缺点
- **循环时间长开销大**
getAndAddInt方法执行，有个do while循环，如果CAS失败，一直会进行尝试，如果CAS长时间不成功，可能会给CPU带来很大的开销

- **只能保证一个共享变量的原子操作**
对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁来保证原子性

- **ABA问题(重点)**

### ABA问题
**ABA问题示例代码:**
````java
 static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
 public static void main(String[] args) {
        new Thread(() -> {
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
````
CAS算法实现一个重要前提需要去除内存中某个时刻的数据并在当下时刻比较并替换，那么在这个时间差类会导致数据的变化。简而言之就是只比较结果,不比较过程.

比如线程1从内存位置V取出A，线程2同时也从内存取出A，并且线程2进行一些操作将值改为B，然后线程2又将V位置数据改成A，这时候线程1进行CAS操作发现内存中的值依然时A，然后线程1操作成功。
尽管线程1的CAS操作成功，但是不代表这个过程没有问题

#### ABA问题解决
- 利用AtomicReference类进行原子引用
````java
package juc.cas;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.ToString;

import java.util.concurrent.atomic.AtomicReference;

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
````java
// 输出结果
true	User(userName=李四, age=23)
false	User(userName=李四, age=23)
````
- 时间戳的原子引用**AtomicStampedReference**(修改版本号)
**主要是在对象中额外再增加一个标记来标识对象是否有过变更**

````java
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
````java
Thread 3	第1次版本号1
Thread 4	第1次版本号1
Thread 3	第2次版本号2
Thread 3	第3次版本号3
Thread 4	修改是否成功false	当前最新实际版本号：3
Thread 4	当前最新实际值：100
````