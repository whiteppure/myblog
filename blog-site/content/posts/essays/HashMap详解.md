---
title: "HashMap详解"
date: 2021-05-03
draft: false
tags: ["Java", "集合"]
slug: "java-hashmap"
---
## 相关概念
- `capacity`： 容量，默认16。
- `loadFactor`： 加载因子，默认是0.75
- `threshold`： 阈值；`阈值 = 容量 * 加载因子`。默认12。当元素数量超过阈值时便会触发扩容。
- hash碰撞(冲突)：当插入hashmap中元素的key出现重复时，就会覆盖之前key对应的值。

## 结构
![HashMap结构](/myblog/posts/images/essays/HashMap结构.png)

- JDK1.7：数组 + 单向链表
- JDK1.8: 数组 + 单向链表/红黑树

在JDK1.8时，如果存储Map中数组元素对应的索引的每个链表超过8，就将单向链表转化为红黑树；当红黑树的节点少于6个的时候又开始使用链表。

### 为什么要使用红黑树？
当有发生大量的hash冲突时，因为链表遍历效率很慢，为了提升查询的效率，所以使用了红黑树的数据结构。

### 为什么不一开始就用红黑树代替链表结构？
JDK文档注释：
> Because TreeNodes are about twice the size of regular nodes, we use them only when bins contain enough nodes to warrant use  (see TREEIFY_THRESHOLD). 
And when they become too small (due to removal or resizing) they are converted back to plain bins. 

单个 TreeNode 需要占用的空间大约是普通 Node 的两倍，所以只有当包含足够多的 Nodes 时才会转成 TreeNodes，而是否足够多就是由 TREEIFY_THRESHOLD 的值（默认值8）决定的。
而当桶中节点数由于移除或者 resize 变少后，又会变回普通的链表的形式，以便节省空间，这个阈值是 UNTREEIFY_THRESHOLD（默认值6）。

### 为什么树化阈值为8？
JDK1.8HashMap文档注释：
> 如果 hashCode 分布良好，也就是 hash 计算的结果离散好的话，那么红黑树这种形式是很少会被用到的，因为各个值都均匀分布，很少出现链表很长的情况。
在理想情况下，链表长度符合泊松分布，各个长度的命中概率依次递减，当长度为 8 的时候，概率仅为 0.00000006。这是一个小于千万分之一的概率，通常我们的 Map 里面是不会存储这么多的数据的，所以通常情况下，并不会发生从链表向红黑树的转换。

HashMap是通过hash算法，来判断对象应该放在哪个桶里面的；
JDK 并不能阻止我们用户实现自己的哈希算法，如果我们故意把哈希算法变得不均匀，那么每次存放对象很容易造成hash冲突。

链表长度超过 8 就转为红黑树的设计，更多的是为了防止用户自己实现了不好的哈希算法时导致链表过长，从而导致查询效率低，而此时转为红黑树更多的是一种保底策略，用来保证极端情况下查询的效率。
红黑树的引入保证了在大量hash冲突的情况下，HashMap还具有良好的查询性能。

### 为什么树化阈值和链表阈值不设置成一样？
为了防止出现节点个数频繁在一个相同的数值来回切换。
举个极端例子，现在单链表的节点个数是9，开始变成红黑树，然后红黑树节点个数又变成8，就又得变成单链表，然后节点个数又变成9，就又得变成红黑树，这样的情况消耗严重浪费。
因此干脆错开两个阈值的大小，使得变成红黑树后“不那么容易”就需要变回单链表，同样，使得变成单链表后，“不那么容易”就需要变回红黑树。

### 引入红黑树后，如果单链表节点个数超过8个是否一定会树化？
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

## 扩容
首次put时，先会触发扩容（算是初始化），然后存入数据，然后判断是否需要扩容；
不是首次put，则不再初始化，直接存入数据，然后判断是否需要扩容；

JDK1.7: 先扩容在添加元素。
JDK1.8: 先添加元素在扩容。

### 为什么要进行扩容？
随着HashMap中的元素增加，获取元素方法的效率就会越来越低，为了保证获取元素方法的效率，所以针对HashMap中的数组进行扩容。
扩容数组的方式只能再去开辟一个新的数组，并把之前的元素转移到新数组上。

### 什么时机进行扩容？
扩容条件：当元素数量超过阈值时便会触发扩容
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
- `新阈值 = 新容量 * 加载因子` 

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
元素A：hash值：0101 0101
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



## 存储

### HashMap初始化时，如果指定容量大小为10，那么实际大小是多少？
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

`HashMap`的`tableSizeFor`方法做了处理，能保证n永远都是2次幂
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


## 参考文章
- https://blog.csdn.net/u010841296/article/details/82832166
- https://blog.csdn.net/Elliot_Elliot/article/details/115610387
- https://blog.csdn.net/pange1991/article/details/82347284
- https://blog.csdn.net/xiewenfeng520/article/details/107119970
- https://zhuanlan.zhihu.com/p/114363420
- https://www.yuque.com/renyong-jmovm/kb/vso1h5
- https://juejin.cn/post/6844903554264596487
