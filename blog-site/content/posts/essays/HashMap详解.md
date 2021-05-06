---
title: "HashMap详解"
date: 2021-05-03
draft: false
tags: ["Java", "集合"]
slug: "java-hashmap"
---
## 相关概念
- `capacity`： 容量，默认16。
- `loadFactor`： 负载因子，表示HashMap满的程度，默认值为0.75f，也就是说默认情况下，当HashMap中元素个数达到了容量的3/4的时候就会进行自动扩容。
- `threshold`： 阈值；`阈值 = 容量 * 负载因子`。默认12。
- hash碰撞：两个不同的输入值，根据同一散列函数计算出的散列值相同的现象叫做碰撞。hash碰撞就是用同一hash散列函数计算出相同的散列值；当插入hashmap中元素的key出现重复时，这个时候就发生了hash碰撞。

## 结构
![HashMap结构](/myblog/posts/images/essays/HashMap结构.png)

- JDK1.7：数组 + 单向链表
- JDK1.8: 数组 + 单向链表/红黑树

在JDK1.8时，如果存储Map中数组元素对应的索引的每个链表超过8，就将单向链表转化为红黑树；当红黑树的节点少于6个的时候又开始使用链表。

### 为什么要使用红黑树
当有发生大量的hash冲突时，因为链表遍历效率很慢，为了提升查询的效率，所以使用了红黑树的数据结构。

### 为什么不一开始就用红黑树代替链表结构
JDK文档注释：
> Because TreeNodes are about twice the size of regular nodes, we use them only when bins contain enough nodes to warrant use  (see TREEIFY_THRESHOLD). 
And when they become too small (due to removal or resizing) they are converted back to plain bins. 

单个 TreeNode 需要占用的空间大约是普通 Node 的两倍，所以只有当包含足够多的 Nodes 时才会转成 TreeNodes，而是否足够多就是由 TREEIFY_THRESHOLD 的值（默认值8）决定的。
而当桶中节点数由于移除或者 resize 变少后，又会变回普通的链表的形式，以便节省空间，这个阈值是 UNTREEIFY_THRESHOLD（默认值6）。

### 为什么树化阈值为8
JDK1.8HashMap文档注释：
> 如果 hashCode 分布良好，也就是 hash 计算的结果离散好的话，那么红黑树这种形式是很少会被用到的，因为各个值都均匀分布，很少出现链表很长的情况。
在理想情况下，链表长度符合泊松分布，各个长度的命中概率依次递减，当长度为 8 的时候，概率仅为 0.00000006。这是一个小于千万分之一的概率，通常我们的 Map 里面是不会存储这么多的数据的，所以通常情况下，并不会发生从链表向红黑树的转换。

HashMap是通过hash算法，来判断对象应该放在哪个桶里面的；
JDK 并不能阻止我们用户实现自己的哈希算法，如果我们故意把哈希算法变得不均匀，那么每次存放对象很容易造成hash冲突。

链表长度超过 8 就转为红黑树的设计，更多的是为了防止用户自己实现了不好的哈希算法时导致链表过长，从而导致查询效率低，而此时转为红黑树更多的是一种保底策略，用来保证极端情况下查询的效率。
红黑树的引入保证了在大量hash冲突的情况下，HashMap还具有良好的查询性能。

### 为什么树化阈值和链表阈值不设置成一样
为了防止出现节点个数频繁在一个相同的数值来回切换。
举个极端例子，现在单链表的节点个数是9，开始变成红黑树，然后红黑树节点个数又变成8，就又得变成单链表，然后节点个数又变成9，就又得变成红黑树，这样的情况消耗严重浪费。
因此干脆错开两个阈值的大小，使得变成红黑树后“不那么容易”就需要变回单链表，同样，使得变成单链表后，“不那么容易”就需要变回红黑树。

### 引入红黑树后，如果单链表节点个数超过8个是否一定会树化
不一定，在进行树化之前会进行判断`(n = tab.length) < MIN_TREEIFY_CAPACITY)`是否需要扩容，如果表中数组元素小于这个阈值（默认是64），就会进行扩容。 
因为扩容不仅能增加表中的容量，还能缩短单链表的节点数，从而更长远的解决链表遍历慢问题。
```
    /**
     * Replaces all linked nodes in bin at index for given hash unless
     * table is too small, in which case resizes instead.
     */
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```

## 容量

### 为什么负载因子默认是0.75
这个值现在在JDK的源码中是0.75
```
/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```
在[JDK的官方文档](https://docs.oracle.com/javase/6/docs/api/java/util/HashMap.html)中解释如下：
> As a general rule, the default load factor (.75) offers a good tradeoff between time and space costs.
 Higher values decrease the space overhead but increase the lookup cost (reflected in most of the operations of the HashMap class, including get and put). 
The expected number of entries in the map and its load factor should be taken into account when setting its initial capacity, so as to minimize the number of rehash operations. 
If the initial capacity is greater than the maximum number of entries divided by the load factor, no rehash operations will ever occur.

大意：一般来说，默认的负载因子(0.75)在时间和空间成本之间提供了很好的权衡。
更高的值减少了空间开销，但增加了查找成本(反映在HashMap类的大多数操作中，包括get和put)。
在设置映射的初始容量时，应该考虑映射中预期的条目数及其负载因子，以便最小化重哈希操作的数量。如果初始容量大于最大条目数除以负载因子，则不会发生重新散列操作。

负载因子和hashmap中的扩容有关，当hashmap中的元素大于临界值（`threshold = loadFactor * capacity`）就会扩容。
所以负载因子的大小决定了什么时机扩容，扩容又影响到了hash碰撞的频率。所以设置一个合理的负载因子可以有效的避免hash碰撞。

设置为0.75的其他解释
- 根据数学公式推算。这个值在`log(2)`的时候比较合理;
- 为了提升扩容效率，HashMap的容量有一个固定的要求，那就是一定是2的幂。如果负载因子是3/4的话，那么和容量的乘积结果就可以是一个整数。

### 如果指定容量大小为10，那么实际大小是多少
实际大小是16。其容量为不小于指定容量的2的幂数。

**为什么容量始终是2的N次方？**

因为2的幂-1都是1111结尾的，所以碰撞几率小。减少Hash碰撞，尽量使Hash算法的结果均匀分布。

当使用put方法时，到底存入HashMap中的那个数组中？这时是通过hash算法决定的，如果某一个数组中的链表过长旧会影响查询的效率；
那么为了避免出现hash碰撞，让hash尽可能的散列分布，就需要在hash算法上做文章。

JDK1.7通过逻辑与运算，来判断这个元素该进入哪个数组；在下面的代码中length的长度始终为不小于指定容量的2的幂数。
```
static int indexFor(int h, int length) {
    return h & (length - 1);
}
```
为了更好的理解举个例子：
假设h=2，或h=3，length=15，进行与运算，最终的结果是一致的，会造成分布不均匀，增加了碰撞的几率，减慢了查询的效率，造成空间的浪费。
````
  0000 0010
& 0000 1110 
———————————— 
  0000 1110

  0000 0011
& 0000 1110 
————————————
  0000 1110
````


在JDK1.8中，在`putVal()`方法中通过`i = (n - 1) & hash`来计算key的散列地址
```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        // 此处省略了代码
        // i = (n - 1) & hash]
        if ((p = tab[i = (n - 1) & hash]) == null)
            
            tab[i] = newNode(hash, key, value, null);
        
 
        else {
            // 省略了代码
        }
}
```
> 这里的 "&" 等同于 %"，但是"%"运算的速度并没有"&"的操作速度快；
"&"操作能代替"%"运算，必须满足一定的条件，也就是`a%b=a&(b-1)`仅当b是2的n次方的时候方能成立。
这也就是为什么HashMap的容量需要保持在2的n次方了。

**怎么保持始终为2的N次方？**

`HashMap`的`tableSizeFor`方法做了处理，能保证n永远都是2次幂。

如果用户制定了初始容量，那么HashMap会计算出比该数大的第一个2的幂作为初始容量；
另外就是在扩容的时候，也是进行成倍的扩容，即4变成8，8变成16。

`1->1、3->4、7->8、9->16`
```
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    // cap-1后，n的二进制最右一位肯定和cap的最右一位不同，即一个为0，一个为1
    // 例如cap=17（00010001），n=cap-1=16（00010000）
    int n = cap - 1;
    // n = (00010000 | 00001000) = 00011000
    n |= n >>> 1;
    // n = (00011000 | 00000110) = 00011110
    n |= n >>> 2;
    // n = (00011110 | 00000001) = 00011111
    n |= n >>> 4;
    // n = (00011111 | 00000000) = 00011111
    n |= n >>> 8;
    // n = (00011111 | 00000000) = 00011111
    n |= n >>> 16;
    // n = 00011111 = 31
    // n = 31 + 1 = 32, 即最终的cap = 32 = 2 的 (n=5)次方
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

```

### 默认初始化容量为什么是16
没有找到相关解释，推断这应该就是个经验值，既然一定要设置一个默认的2^n 作为初始值，那么就需要在效率和内存使用上做一个权衡。
这个值既不能太小，也不能太大。太小了就有可能频繁发生扩容，影响效率。太大了又浪费空间，不划算。所以，16就作为一个经验值被采用了。

关于默认容量的定义
```
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
故意把16写成`1 << 4`这种形式，就是提醒开发者，这个地方要是2的次幂。

### 初始化容量设置多少合适
当我们使用`HashMap(int initialCapacity)`来初始化容量的时候，`HashMap`并不会使用我们传进来的`initialCapacity`直接作为初始容量。
JDK会默认帮我们计算一个相对合理的值当做初始容量。所谓合理值，其实是找到第一个比用户传入的值大的2的幂。

如果创建hashMap初始化容量设置为7，那么JDK通过计算会创建一个初始化为8的hashMap。
当hashMap中的元素到`0.75 * 8 = 6`就会进行扩容，这明显是我们不希望看到的。

参考JDK8中`putAll`方法中的实现
```
(int) ((float) expectedSize / 0.75F + 1.0F);
```
通过`expectedSize / 0.75F + 1.0F`计算，`7/0.75 + 1 = 10` ,10经过JDK处理之后，会被设置成16，这就大大的减少了扩容的几率。

当我们明确知道HashMap中元素的个数的时候，把默认容量设置成`expectedSize / 0.75F + 1.0F` 是一个在性能上相对好的选择，但是，同时也会牺牲些内存。

这个算法在guava中有实现，开发的时候，可以直接通过Maps类创建一个HashMap：
```
Map<String, String> map = Maps.newHashMapWithExpectedSize(7);
```
```
public static <K, V> HashMap<K, V> newHashMapWithExpectedSize(int expectedSize) {
    return new HashMap(capacity(expectedSize));
}

static int capacity(int expectedSize) {
    if (expectedSize < 3) {
        CollectPreconditions.checkNonnegative(expectedSize, "expectedSize");
        return expectedSize + 1;
    } else {
        return expectedSize < 1073741824 ? (int)((float)expectedSize / 0.75F + 1.0F) : 2147483647;
    }
}
```

## 扩容
- JDK1.7: 先扩容在添加元素。
- JDK1.8: 先添加元素在扩容。

### 为什么要进行扩容
随着HashMap中的元素增加，Hash碰撞导致获取元素方法的效率就会越来越低，为了保证获取元素方法的效率，所以针对HashMap中的数组进行扩容。
扩容数组的方式只能再去开辟一个新的数组，并把之前的元素转移到新数组上。

> PS 如何能避免哈希碰撞?
>- 容量太小。容量小，碰撞的概率就高了。狼多肉少，就会发生争抢。
>- hash算法不够好。算法不合理，就可能都分到同一个或几个桶中。分配不均，也会发生争抢。

### 什么时机进行扩容
HashMap的扩容条件就是当HashMap中的元素个数（size）超过临界值（threshold）时就会自动扩容。
在HashMap中，`threshold = loadFactor * capacity`。默认情况下负载因子为0.75,理解为当容器中元素到容器的3/4时就会扩容。
```
 if (++size > threshold)
    resize();
```
HashMap的容量是有上限的，必须小于`1<<30`，即`1073741824`。如果容量超出了这个数，则不再增长，且阈值会被设置为`Integer.MAX_VALUE`.
```
// Java8
if (oldCap >= MAXIMUM_CAPACITY) {
    threshold = Integer.MAX_VALUE;
    return oldTab;
}
// Java7
if (oldCapacity == MAXIMUM_CAPACITY) { 
    threshold = Integer.MAX_VALUE;
    return;
}
```
### 1.7扩容
- `新容量 = 旧容量 * 2` 
- `新阈值 = 新容量 * 负载因子` 

```
void addEntry(int hash, K key, V value, int bucketIndex) {  
    //size：The number of key-value mappings contained in this map.  
    //threshold：The next size value at which to resize (capacity * load factor)  
    //数组扩容条件：1.已经存在的key-value mappings的个数大于等于阈值  
    //             2.底层数组的bucketIndex坐标处不等于null  
    if ((size >= threshold) && (null != table[bucketIndex])) {  
        resize(2 * table.length);//扩容之后，数组长度变了  
        hash = (null != key) ? hash(key) : 0;//为什么要再次计算一下hash值呢？  
        bucketIndex = indexFor(hash, table.length);//扩容之后，数组长度变了，在数组的下标跟数组长度有关，得重算。  
    }  
    createEntry(hash, key, value, bucketIndex);  
} 
```
```
void resize(int newCapacity) {   //传入新的容量
    Entry[] oldTable = table;    //引用扩容前的Entry数组
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {  //扩容前的数组大小如果已经达到最大(2^30)了
        threshold = Integer.MAX_VALUE; //修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
        return;
    }

    Entry[] newTable = new Entry[newCapacity];  //初始化一个新的Entry数组
    transfer(newTable);                         //！！将数据转移到新的Entry数组里
    table = newTable;                           //HashMap的table属性引用新的Entry数组
    threshold = (int) (newCapacity * loadFactor);//修改阈值
}
```
通过transfer方法将旧数组上的元素转移到扩容后的新数组上
```
void transfer(Entry[] newTable) {
    Entry[] src = table;                   //src引用了旧的Entry数组
    int newCapacity = newTable.length;
    for (int j = 0; j < src.length; j++) { //遍历旧的Entry数组
        Entry<K, V> e = src[j];             //取得旧Entry数组的每个元素
        if (e != null) {
            src[j] = null;//释放旧Entry数组的对象引用（for循环后，旧的Entry数组不再引用任何对象）
            do {
                Entry<K, V> next = e.next;
                int i = indexFor(e.hash, newCapacity); //！！重新计算每个元素在数组中的位置
                e.next = newTable[i]; //标记[1]
                newTable[i] = e;      //将元素放在数组上
                e = next;             //访问下一个Entry链上的元素
            } while (e != null);
        }
    }
}
```


### 1.8扩容
容量变为原来的2倍，阈值也变为原来的2倍。容量和阈值都变为原来的2倍时，负载因子还是不变。

在1.8时做了一些优化，文档注释写的很清楚："元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置"。
也就是对比1.7的迁移到新的数组上省去了重新计算hash值的时间。

这里的"2次幂的位置"是指长度为原来数组元素的两倍的位置;

举个例子,现在容量为16，要扩容到32，要将之前的元素迁移过去，要根据hash值来判断迁移过去的位置；
假设元素A：hash值：0101 0101；根据代码`h & (length - 1)`可得`元素A & 15`、`元素A & 31`
```
扩容之前的位置：
  0101 0101
& 0000 1111
————————————
  0000 0101

扩容之后的位置：
  0101 0101
& 0001 1111
————————————
  0001 0101
```
发现规律：扩容前的hash值和扩容后的hash值，如果元素A二进制形式第三位如果是0，扩容之后就还是原来的位置，如果是1扩容后就是原来的位置加16，而16就是扩容的大小。


```
 /**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```





## 参考文章
- https://blog.csdn.net/u010841296/article/details/82832166
- https://blog.csdn.net/Elliot_Elliot/article/details/115610387
- https://blog.csdn.net/pange1991/article/details/82347284
- https://blog.csdn.net/xiewenfeng520/article/details/107119970
- https://zhuanlan.zhihu.com/p/114363420
- https://www.yuque.com/renyong-jmovm/kb/vso1h5
- https://juejin.cn/post/6844903554264596487
- http://hollischuang.gitee.io/tobetopjavaer/#/basics/java-basic/hashmap-init-capacity
