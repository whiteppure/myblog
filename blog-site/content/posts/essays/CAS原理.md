---
title: "CAS原理"
date: 2020-04-04
draft: false
tags: ["线程"]
slug: "cas-principle"
---

## CAS
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

### 如何保证数据一致性

从源码可以看出，是通过CPU指令进行调用,当CPU中某个处理器对缓存中的共享变量进行了操作，其他处理器会有个嗅探机制，将其他处理器的该共享变量的缓存失效，待其他线程读取时会重新从主内存中读取最新的数据，基于 `MESI` 缓存一致性协议来实现的。

简述，就是通过CPU的缓存一致性协议来保证线程之间的数据一致性的。

> CPU 处理器速度远远大于在主内存中的，为了解决速度差异，在他们之间架设了多级缓存，如 L1、L2、L3 级别的缓存，这些缓存离CPU越近就越快，将频繁操作的数据缓存到这里，加快访问速度。

![JMM](/myblog/posts/images/essays/JMM.jpg)


### CAS与Unsafe关系
CAS的作用是比较并交换,就是先拿这个期望值,与主内存的值比较,判断主内存中该位置是否存在期望值,
如果存在,则改为新的值,这个修改的过程是具有原子性的.
因为CAS是cpu并发源语,并发源语体现在Java sun.misc.Unsafa类上.
调用Unsafe类中的CAS方法，JVM会帮我们实现CAS汇编指令。这是一种完全依赖于硬件的功能，通过他实现了原子操作。
由于CAS是一种系统原语，原语属于操作系统用语范畴，是由若干条指令组成的，用于完成某个功能的一个过程，并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU的原子指令，不会造成数据不一致问题。

> PS Unsafe类
CAS其实是调用了 `Unsafe` 类的方法 `Unsafa` 类是CAS核心类，由于Java方法无法直接访问底层系统，需要通过本地（`native`）方法来访问，`Unsafe` 相当于一个后门，基于该类可以直接操作特定内存数据。
Unsafe类存在于sun.misc包中，其内部方法操作可以像C的指针(内存地址)一样直接操作内存，因此Java中CAS操作的执行依赖于Unsafe类的方法。
Unsafe类中的所有方法都是**native**修饰的，也就是说Unsafe类中的方法都直接**调用操作系统底层资源执行相应任务**。

### 缺点
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
