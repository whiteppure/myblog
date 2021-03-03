
---
title: "Java中集合"
date: 2020-03-30
draft: false
tags: ["Java", "集合"]
slug: "java-collection"
---

Java中的集合主要包括 `Collection` 和 `Map` 两种，
`Collection` 存储着对象的集合，而 `Map` 存储着键值对（两个对象）的映射表。

## Collection
![集合框架](/myblog/posts/images/essays/集合框架.jpg)

### Set
- TreeSet：基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。
- HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。HashSet的value作为hashmap的key，来保证不重复。
- LinkedHashSet：具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序。

### List
- ArrayList：基于动态数组实现，支持随机访问。
- Vector：和 ArrayList 类似，但它是线程安全的。
- LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。

### Queue
- LinkedList：可以用它来实现双向队列。
- PriorityQueue：基于堆结构实现，可以用它来实现优先队列。

## Map
- TreeMap：基于红黑树实现。
- HashMap：基于哈希表实现。
- HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程可以同时写入 HashTable 并且不会导致数据不一致。它是遗留类，不应该去使用它。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。
- LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。


### HashMap

- 存贮: 通过key的hashcode方法找到在hashMap 存贮的位置,如果该位置有元素,则通过equals方法进行比较,equals返回值为true,则覆盖value,equals返回值为false则,在该数组元素的头部追加该元素,形成一个链表结构

- 读取:通过key的hashcode方法获取元素存在该数组的位置,然后通过equals拿到该值

- 结构: hashMap是一个散列数据结构,HashMap底层就是一个数组结构，数组中的每一项又是一个链表

- 归纳: 总的来说hashMap底层将key-value(键值对)当成一个整体来处理,hashMap底层采用一个 Entry 数组保存所有的键值对,当存储一个entry对象时,会根据key的hash算法来决定存放在数组中的位置,在根据equals方法来确定在链表中的位置,读取一个entry对象,先根据hash算法确定在数组中的位置,再根据equals来获取该值,equals和equals在hashMap中就像一个坐标一样,来确定hashMap中的值


### 1.8的hashMap与1.7之前的区别
- 结构 :  数组+链表+红黑树(链表长度>8时使用)
- 算法:   扩容后数据存储位置的计算方式也不一样

**hash冲突** 
发生hash冲突时,1.8链表采用尾插法,当大于8时,则转换为红黑树;
因为JDK1.7是用单链表进行的纵向延伸，当采用头插法时会容易出现逆序且环形链表死循环问题。
但是在JDK1.8之后是因为加入了红黑树使用尾插法，能够避免出现逆序且链表死循环的问题。

总结:
1. 新增红黑树数据结构,增强效率;
2. 采用尾插法+红黑树结构,解决逆序形成环形链表形成死循环的问题
3. 数据存储位置的计算方式不一样

### ConcurrentHashMap
线程安全的HashMap

图示:

在多线程中，每一个Segment对象守护了一个HashEntry数组，
当对ConcurrentHashMap中的元素修改时，
在获取到对应的Segment数组角标后，都会对此Segment对象加锁，
之后再去操作后面的HashEntry元素，这样每一个Segment对象下，都形成了一个小小的HashMap，在保证数据安全性的同时，又提高了同步的效率。
只要不是操作同一个Segment对象的话，就不会出现线程等待的问题

- 原理: Segment数组+HashEntry数组+链表;
其中Segment起到了加锁同步的作用，而HashEntry则起到了存储K.V键值对的作用。
