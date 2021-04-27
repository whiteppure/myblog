---
title: "Java对象"
date: 2021-04-12
draft: false
tags: ["Java", "JVM"]
slug: "java-object"
---

## 对象实例化

### 对象创建方式
- 使用new关键字创建：最常见的方式、单例类中调用`getInstance`的静态类方法，`XXXFactory`的静态方法；
- 使用反射方式创建：
    - 使用`Class`的`newInstance`方法：在JDK9里面被标记为过时的方法，因为只能调用空参构造器；
    - 使用`Constructor`的`newInstance(XXX)`：反射的方式，可以调用空参的，或者带参的构造器；
    - 使用`clone()`：不调用任何的构造器，要求当前的类需要实现`Cloneable`接口中的`clone`接口；
    - 使用序列化：序列化一般用于`Socket`的网络传输；
- 使用第三方库 `Objenesis`；

### 创建对象的过程及步骤
创建对象代码演示及字节码指令
```
public class MainTest {
    public static void main(String[] args) {
        Object obj = new Object();
    }
}
```
![创建对象字节码指令](/myblog/posts/images/essays/创建对象字节码指令.png)

创建对象步骤：
1. 加载类元信息
2. 为对象分配内存
3. 处理并发问题
4. 属性的默认初始化（零值初始化）
5. 设置对象头信息
6. 属性的显示初始化、代码块中初始化、构造器中初始化

**判断对象对应的类是否加载、链接、初始化**

虚拟机遇到一条new指令，首先去检查这个指令的参数能否在 `Metaspace` 的常量池中定位到一个类的符号引用，
并且检查这个符号引用代表的类是否已经被加载，解析和初始化。（即判断类元信息是否存在）。
如果没有，那么在双亲委派模式下，使用当前类加载器以 `ClassLoader + 包名 + 类名` 为key进行查找对应的 `.class `文件，
如果没有找到文件，则抛出 `ClassNotFoundException` 异常，如果找到，则进行类加载，并生成对应的Class对象。

**为对象分配内存**

首先计算对象占用空间的大小，接着在堆中划分一块内存给新对象。
如果实例成员变量是引用类型，仅分配引用变量空间即可，即4个字节大小.

如果内存规整：指针碰撞；如果内存不规整：虚拟表需要维护一个列表，即空闲列表分配

> 指针碰撞:
所有用过的内存在一边，空闲的内存放另外一边，中间放着一个指针作为分界点的指示器，分配内存就仅仅是把指针指向空闲那边挪动一段与对象大小相等的距离罢了。
如果垃圾收集器选择的是Serial ，ParNew这种基于压缩算法的，虚拟机采用这种分配方式。一般使用带Compact（整理）过程的收集器时，使用指针碰撞。

> 空闲列表分配:
虚拟机维护了一个列表，记录上那些内存块是可用的，再分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的内容。
这种分配方式成为了 “空闲列表（Free List）”。

选择哪种分配方式由Java堆是否规整所决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。

**处理并发问题**

- 采用CAS配上失败重试保证更新的原子性
- 每个线程预先分配TLAB - 通过设置 `-XX:+UseTLAB`参数来设置,在Java8是默认开启的

**初始化分配到的内存**

就是给对象属性赋值的操作
- 属性的默认初始化
- 显示初始化
- 代码块中的初始化
- 构造器初始化

给所有属性设置默认值，保证对象实例字段在不赋值时可以直接使用。

**设置对象的对象头**

将对象的所属类（即类的元数据信息）、对象的 `HashCode` 和对象的GC信息、锁信息等数据存储在对象的对象头中。这个过程的具体设置方式取决于JVM实现。

**执行init方法进行初始化**

初始化成员变量，执行实例化代码块，调用类的构造方法，并把堆内对象的首地址赋值给引用变量。
因此一般来说（由字节码中跟随 `invokespecial` 指令所决定），new指令之后会接着执行方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完成创建出来。

## [对象组成](http://openjdk.java.net/groups/hotspot/docs/HotSpotGlossary.html)
**引入依赖**
```
<!-- 引入查看对象布局的依赖 -->
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.9</version>
</dependency>

```

**测试代码**
```
@Test
public void demo() {
    T t = new T();//这个对象里面是空的什么都没有
    System.out.println(ClassLayout.parseInstance(t).toPrintable());
}

```

**测试结果**
```
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           a1 2c 01 20 (10100001 00101100 00000001 00100000) (536947873)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

在64位Jvm下,通过测试的结果我们可以看到Java对象的布局分为对象头,对齐数据(因为Jvm规定内存分配的字节必须是8的倍数,否则无法分配内存),
当该类里边有变量时,还包含实例变量。

![对象内存布局](/myblog/posts/images/essays/对象内存布局.png)

### 对象头
对象头在32位下是4个字节,64位下是12个字节(96bite)。
对象头包含了两部分，分别是 运行时元数据和类型指针；如果是数组，还需要记录数组的长度.

运行时元数据:
- 哈希值
- GC分代年龄
- 锁状态标志
- 线程持有的锁
- 偏向线程ID
- 偏向时间戳

类型指针：

指向类元数据`InstanceKlass`，确定该对象所属的类型。指向的其实是方法区中存放的类元信息;并不是所有的对象都有类型指针。


### 实例数据
独立于方法之外的变量,在类里定义的变量无 `static` 修饰；包括从父类继承下来的变量和本身拥有的字段。

一些规则：
- 相同宽度的字段总是被分配到一起（比如都是8个字节的，会放到一起）；
- 父类中定义的变量会出现在子类之前；
- 如果JVM参数`CompactFields`设置为true，子类的窄变量可能插入到父类的间隙；

### 对齐填充数据
不是必须的，也没有特别的含义，仅仅起到占位符的作用。
因为Jvm规定内存分配的字节必须是8的倍数,否则无法分配内存.如果对象头占得字节 + 实例变量占得字节刚好为8的倍数,对齐填充数据则不存在。


## 对象的访问定位
JVM是如何通过栈帧中的对象引用访问到其内部的对象实例呢？
![对象指针访问](/myblog/posts/images/essays/对象指针访问.png)

hotspot使用的是直接访问。因为句柄访问开辟了句柄池，所以直接访问相较于句柄访问效率稍高一点。

### 句柄访问
句柄访问是在栈的局部变量表中，记录的对象的引用，然后在堆空间中开辟了一块空间，也就是句柄池。指向的是方法区中的对象类型数据。

优点：`reference` 中存储稳定句柄地址，对象被移动（垃圾收集时移动对象很普遍）时只会改变句柄中实例数据指针即可，`reference` 本身不需要被修改

![句柄访问](/myblog/posts/images/essays/句柄访问.png)

### 直接访问
直接指针是局部变量表中的引用，直接指向堆中的实例，在对象实例中有类型指针，指向的是方法区中的对象类型数据。

![直接访问](/myblog/posts/images/essays/直接访问.png)

## 对象的终止机制
Java语言提供了对象终止（`finalization`）机制来允许开发人员提供对象被销毁之前的自定义处理逻辑。
当垃圾回收器发现没有引用指向一个对象，即：垃圾回收此对象之前，总会先调用这个对象的`finalize()`方法。

`finalize()` 方法允许在子类中被重写，用于在对象被回收时进行资源释放。
通常在这个方法中进行一些资源释放和清理的工作，比如关闭文件、套接字和数据库连接等。

文档注释大意：当GC确定不再有对对象的引用时，由垃圾收集器在对象上调用。子类重写`finalize`方法来释放系统资源或执行其他清理。
```
   /**
     * Called by the garbage collector on an object when garbage collection
     * determines that there are no more references to the object.
     * A subclass overrides the {@code finalize} method to dispose of
     * system resources or to perform other cleanup.
     */
    protected void finalize() throws Throwable { }
```

**永远不要主动调用某个对象的`finalize`方法应该交给垃圾回收机制调用原因**

- 在调用`finalize`方法时时可能会导致对象复活；
- `finalize`方法的执行时间是没有保障的，它完全由GC线程决定，极端情况下，若不发生GC，则`finalize`方法将没有执行机会;
因为优先级比较低，即使主动调用该方法，也不会因此就直接进行回收
- 一个糟糕的`finalize`方法会严重影响GC的性能;

**由于`finalize`方法的存在,虚拟机中的对象一般处于三种可能的状态**

如果从所有的根节点都无法访问到某个对象，说明对象己经不再使用了。一般来说，此对象需要被回收。
但事实上，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段。一个无法触及的对象有可能在某一个条件下“复活”自己，如果这样，那么对它的回收就是不合理的，为此，虚拟机中定义了的对象可能的三种状态：
- 可触及的：从根节点开始，可以到达这个对象；对象存活被使用；
- 可复活的：对象的所有引用都被释放，但是对象有可能在`finalize`中复活；对象被复活，对象在`finalize`方法中被重新使用；
- 不可触及的：对象的`finalize`方法被调用，并且没有复活，那么就会进入不可触及状态；对象死亡，对象没有被使用；

只有在对象不可触及时才可以被回收。不可触及的对象不可能被复活，因为`finalize()`只会被调用一次。

**`finalize`机制判定一个对象能否被回收过程**

判定一个对象是否可回收，至少要经历两次标记过程：
- 如果对象没有没有引用链，则进行第一次标记
- 进行筛选，判断此对象是否有必要执行`finalize`方法
    1. 如果对象没有重写`finalize`方法，或者`finalize`方法已经被虚拟机调用过，则虚拟机视为“没有必要执行”，对象被判定为不可触及的。
    2. 如果对象重写了`finalize`方法，且还未执行过，那么会被插入到`F-Queue`队列中，由一个虚拟机自动创建的、低优先级的`Finalizer`线程触发其`finalize`方法执行。
    3. `finalize`方法是对象逃脱死亡的最后机会，稍后GC会对`F-Queue`队列中的对象进行第二次标记。如果对象在`finalize`方法中与引用链上的任何一个对象建立了联系，那么在第二次标记时，该对象会被移出“即将回收”集合。
    之后，对象会再次出现没有引用存在的情况。在这个情况下，`finalize`方法不会被再次调用，对象会直接变成不可触及的状态，也就是说，一个对象的`finalize`方法只会被调用一次。
    
代码演示对象能否被回收过程
```
public class MainTest {

    public static MainTest var;

    /**
     * 此方法只能被调用一次
     * 可对该方法进行注释，来测试finalize方法是否能复活对象
     */
    @Override
    protected void finalize() throws Throwable {
        System.out.println("调用当前类重写的finalize()方法");
        // 复活对象 让当前带回收对象重新与引用链中的对象建立联系
        var = this;
    }

    public static void main(String[] args) throws InterruptedException {
        var = new MainTest();
        var = null;
        System.gc();
        System.out.println("-----------------第一次gc操作------------");
        // 因为Finalizer线程的优先级比较低，暂停2秒，以等待它
        Thread.sleep(2000);
        if (var == null) {
            System.out.println("对象已经死了");
            // 如果第一次对象就死亡了 就终止
            return;
        } else {
            System.out.println("对象还活着");
        }

        System.out.println("-----------------第二次gc操作------------");
        var = null;
        System.gc();
        // 下面代码和上面代码是一样的，但是 对象却自救失败了
        Thread.sleep(2000);
        if (var == null) {
            System.out.println("对象已经死了");
        } else {
            System.out.println("对象还活着");
        }
    }

}
```