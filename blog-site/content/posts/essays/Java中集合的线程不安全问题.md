---
title: "Java中集合的线程不安全问题"
date: 2020-04-05
draft: false
tags: ["线程", "Java", "集合"]
slug: "java-thread-collection"
---

### ArrayList

- **ArrayList线程不安全示例:**
```java
	public static void main(String[] args) {
		ArrayList<String> arrayList = new ArrayList<>();
		for(int i=0; i< 3; i++) {
			new Thread(() -> {
				arrayList.add(UUID.randomUUID().toString());
				System.out.println(arrayList);
			},String.valueOf(i)).start();
		}
	}
```
- **运行结果:**
```java
// ConcurrentModificationException 同步修改异常
Exception in thread "8" java.util.ConcurrentModificationException

[null, 2041b613-8068-4ddd-9d01-305f5680d377]
[null, 2041b613-8068-4ddd-9d01-305f5680d377, b3e0296d-e263-4632-a023-4267cdec5e25]
[null, 2041b613-8068-4ddd-9d01-305f5680d377]
```
- **原因分析:**
当某个线程正在执行 `add()`方法时,被某个线程打断,添加到一半被打断,没有被添加完

- **解决方案:**
1. 使用`Vector`来代替`ArrayList`,`Vector`是线程安全的`ArrayList`,但是由于,并发量太小,被淘汰
2. 使用`Collections.synchronizedArrayList(new ArrayList<>())`来创建`ArrayList`.使用`Collections`工具类来创建`ArrayList`的思路是,在`ArrayList`的外边套了一个外壳,来使`ArrayList`线程安全
3. **使用`new CopyOnWriteArrayList()`来保证ArrayList线程安全**

#### CopyOnWriteArrayList原理
`CopyWriteArrayList`字面意思就是在写的时候复制,思想就是读写分离的思想

以下是`CopyOnWriteArrayList`的`add()`方法源码
```java
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
因为在源码里面加了`ReentrantLock`所以保证了某个线程在写的时候不会被打断,可以看到源码开始先是复制了一份数组(因为同一时刻只有一个线程写,其余的线程会读),在复制的数组上边进行写操作,写好以后在返回`true`.这样写的就把读写进行了分离.写好以后因为`array`加了`volatile`关键字,所以该数组是对于其他的线程是可见的,就会读取到最新的值.

### HashSet
`HashSet`和`ArrayList`类似,也是线程不安全的集合类,具体证明`HashSet`线程不安全的代码,请参考`ArrayList`线程不安全的示例.
因为与ArrayList类似,都属于一类问题,也会报`ConcurrentModificationException `异常.
- **解决方案**
1. `Collections.synchronizedSet(new HashSet<>())`使用集合工具类解决
2. 使用`new CopyOnWriteArraySet<>()`来保证集合线程安全

#### CopyOnWriteArraySet原理
```java
   private final CopyOnWriteArrayList<E> al;

    /**
     * Creates an empty set.
     */
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
```
底层是`CopyOnWriteArrayList`

### HashMap
`HashMap`也是线程不安全的集合类
- 解决方案
1.`Collections.synchronizedMap(new HashMap<>())`使用集合工具类
2.`new ConcurrentHashMap<>()`来保证线程安全

#### ConcurrentHashMap原理
**参考:**[CurrentHashMap原理](https://baijiahao.baidu.com/s?id=1617089947709260129&wfr=spider&for=pc)
在**JDK1.7中ConcurrentHashMap**采用了数组+Segment+分段锁的方式实现。

- Segment(分段锁)
ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表,同时又是一个ReentrantLock（Segment继承了ReentrantLock）。

- 内部结构
ConcurrentHashMap使用分段锁技术，将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问，能够实现真正的并发访问。如下图是ConcurrentHashMap的内部结构图：
![CurrentHashMap](/myblog/posts/images/essays/concurrentHashMap的内部结构图.jpeg)

从上面的结构我们可以了解到，`ConcurrentHashMap`定位一个元素的过程需要进行两次Hash操作。

第一次Hash定位到Segment，第二次Hash定位到元素所在的链表的头部。

- 该结构的优劣势

    - 坏处: 这一种结构的带来的副作用是Hash的过程要比普通的HashMap要长
    - 好处: 写操作的时候可以只对元素所在的Segment进行加锁即可，不会影响到其他的Segment，这样，在最理想的情况下，ConcurrentHashMap可以最高同时支持Segment数量大小的写操作（刚好这些写操作都非常平均地分布在所有的Segment上).所以，通过这一种结构，ConcurrentHashMap的并发能力可以大大的提高。

**JDK1.8版本的CurrentHashMap的实现原理**

JDK8中`ConcurrentHashMap`参考了JDK8 HashMap的实现，采用了数组+链表+红黑树的实现方式来设计，内部大量采用CAS操作.如果不懂CAS请移步[CAS原理](http://www.ljzblog.xyz/archives/cas%E5%8E%9F%E7%90%86)
JDK8中彻底放弃了Segment转而采用的是Node，其设计思想也不再是JDK1.7中的分段锁思想。

Node：保存key，value及key的hash值的数据结构。其中value和next都用volatile修饰，保证并发的可见性。
Java8 ConcurrentHashMap结构基本上和Java8的HashMap一样，不过保证线程安全性。

在JDK8中ConcurrentHashMap的结构，由于引入了红黑树，使得ConcurrentHashMap的实现非常复杂，我们都知道，红黑树是一种性能非常好的二叉查找树，其查找性能为O（logN），但是其实现过程也非常复杂，而且可读性也非常差，DougLea的思维能力确实不是一般人能比的，早期完全采用链表结构时Map的查找时间复杂度为O（N），JDK8中ConcurrentHashMap在链表的长度大于某个阈值的时候会将链表转换成红黑树进一步提高其查找性能。
![JDK1.8后的currentHashMap](/myblog/posts/images/essays/红黑树结构.jpeg)
其实可以看出JDK1.8版本的`ConcurrentHashMap`的数据结构已经接近HashMap，相对而言，ConcurrentHashMap只是增加了同步的操作来控制并发，从JDK1.7版本的ReentrantLock+Segment+HashEntry，到JDK1.8版本中synchronized+CAS+HashEntry+红黑树。

**CurrentHashMapJDK1.7,JDK1.8前后对比**
1. 数据结构：取消了Segment分段锁的数据结构，取而代之的是数组+链表+红黑树的结构。
2. 保证线程安全机制：JDK1.7采用segment的分段锁机制实现线程安全，其中segment继承自ReentrantLock。JDK1.8采用CAS+Synchronized保证线程安全。
3. 锁的粒度：原来是对需要进行数据操作的Segment加锁，现调整为对每个数组元素加锁（Node）。
4. 链表转化为红黑树:定位结点的hash算法简化会带来弊端,Hash冲突加剧,因此在链表节点数量大于8时，会将链表转化为红黑树进行存储。
5. 查询时间复杂度：从原来的遍历链表O(n)，变成遍历红黑树O(logN)。

