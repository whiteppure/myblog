---
title: "Redis详解"
date: 2021-06-17
draft: false
tags: ["Java", "Redis"]
slug: "java-redis"
---
## Redis介绍

## Redis与缓存

## Redis与消息队列

## Redis分布式锁
在代码里面我们常用`ReetrantLock、synchronized`保证线程安全。通过上面的锁，在某个时刻只能保证一个线程执行锁作用域内的代码。

类似这样：
```
public class MainTest {

    private static final  ReentrantLock lock = new ReentrantLock();
    
    public static void main(String[] args) {
        lock.lock();
        try {
            System.out.println("hello world");
        }finally {
            lock.unlock();
        }
    }
}
```
但是，当项目采用分布式部署方式之后，再使用`ReetrantLock、synchronized`就不能保证数据的准确性，可能会出现严重bug。
![基本的分布式部署结构](/myblog/posts/images/essays/基本的分布式部署结构.png)

举个例子，当很多个请求过来的时候，会先经过`Nginx`,然后`Nginx`再根据算法分发请求，到哪些服务器的程序上。
此时商品的库存为一件，有两个请求，到达不同服务器上的不同程序的相同代码，先后执行了查询SQL，查出来的数据是相同的，然后依次执行库存减一操作，此时库存会变成-1件。这就造成了超卖问题。

针对超卖问题，我们可以使用Redis分布式锁来解决。当然也有其他分布式锁，这里不做介绍。

### Redis分布式锁演进

#### 方式一
使用Redis，`setnx`：
> SETNX 是SET IF NOT EXISTS的简写.日常命令格式是SETNX key value，如果 key不存在，则SETNX成功返回1，如果这个key已经存在了，则返回0。
```
    @Autowired
    private RedisTemplate redisTemplate;

    // 保证value值唯一,这里是伪代码
    final String value = "";
    final String REDIS_LOCK = "redis_lock_demo";
    public void context(){
        try {
            Boolean flag =  redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK,value);
            if (!flag) {
                System.out.println("抢锁失败！");
            }
            String redisKey =  redisTemplate.opsForvalue().get("redis_key");
            int num0 = redisKey == null ? 0 : Integer.parseInt(redisKey);
    
            if (num0 <= 0){
                System.out.println("商品已售完！");
                return;
            }
    
            // 卖出商品，存入Redis中
            int num1  = num0 - 1;
            redisTemplate.opsForvalue().set("redis_key",num1);
        }finally {
            redisTemplate.delete(REDIS_LOCK);
        }
    }
```
需要注意的是`redisTemplate.delete()`方法要加在`finally`中是为了程序出现异常不释放锁。

但是这种写法会有一个问题，如果Redis服务器宕机了，或Redis服务被其他人kill掉了，此时恰好没有执行`finally`中的代码，就会造成Redis中永远都会存在这把锁，不会释放。

#### 方式二
针对上面Redis宕机的问题，我们可以对这个key加一个过期时间，来解决：
```
    @Autowired
    private RedisTemplate redisTemplate;

    // 保证value值唯一,这里是伪代码
    final String value = "";
    final String REDIS_LOCK = "redis_lock_demo";
    public void context(){
        try {   
            // 加锁
            Boolean flag =  redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK,value);
            // 设置过期时间，假设为10s
            redisTemplate.expire(REDIS_LOCK,10, TimeUnit.SECONDS);
    
            if (!flag) {
                System.out.println("抢锁失败！");
            }
            String redisKey =  redisTemplate.opsForvalue().get("redis_key");
            int num0 = redisKey == null ? 0 : Integer.parseInt(redisKey);
    
            if (num0 <= 0){
                System.out.println("商品已售完！");
                return;
            }
    
            // 卖出商品，存入Redis中
            int num1  = num0 - 1;
            redisTemplate.opsForvalue().set("redis_key",num1);
        }finally {
            redisTemplate.delete(REDIS_LOCK);
        }
    }
```
上面的代码虽然解决了Redis宕机的问题，但是也带来了一个新的问题：设置过期时间和加锁并不再一行，即是非原子操作。

举个例子，如果执行完`setnx`加锁，正要执行`expire`设置过期时间时，进程要重启维护了，那么这个锁就“长生不老”了，别的线程永远获取不到锁了。

#### 方式三
针对上面加锁和设置过期时间的问题，我们可以使用Redis提供的一个方法，使其具备原子性：
```
    @Autowired
    private RedisTemplate redisTemplate;

    // 保证value值唯一,这里是伪代码
    final String value = "";
    final String REDIS_LOCK = "redis_lock_demo";
    public void context(){
        try {
            Boolean flag =  redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK,value, 10, TimeUnit.SECONDS);

            if (!flag) {
                System.out.println("抢锁失败！");
            }
            String redisKey =  redisTemplate.opsForvalue().get("redis_key");
            int num0 = redisKey == null ? 0 : Integer.parseInt(redisKey);

            if (num0 <= 0){
                System.out.println("商品已售完！");
                return;
            }

            // 卖出商品，存入Redis中
            int num1  = num0 - 1;
            redisTemplate.opsForvalue().set("redis_key",num1);
        }finally {
            redisTemplate.delete(REDIS_LOCK);
        }
    }
```
加过期时间释放锁的这种方式会带来另一个问题，某个线程加锁，然后执行业务代码，业务代码执行的时间超过了限定时间，此时Redis会释放锁，然后第二个请求就进来了，此时第一个线程业务代码执行完毕，执行释放锁步骤。这就造成误删除其他线程的锁。

简单说就是，张冠李戴，当前线程删除了其他线程的锁。

#### 方式四
针对方式三带来的问题，需要加一个判断，来避免误删除其他线程的锁：
```
    @Autowired
    private RedisTemplate redisTemplate;

    final String REDIS_LOCK = "redis_lock_demo";
    // 保证value值唯一,这里是伪代码
    final String value = "";
    public void context(){
        try {
            Boolean flag =  redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK,value, 10, TimeUnit.SECONDS);

            if (!flag) {
                System.out.println("抢锁失败！");
            }
            String redisKey =  redisTemplate.opsForvalue().get("redis_key");
            int num0 = redisKey == null ? 0 : Integer.parseInt(redisKey);

            if (num0 <= 0){
                System.out.println("商品已售完！");
                return;
            }

            // 卖出商品，存入Redis中
            int num1  = num0 - 1;
            redisTemplate.opsForvalue().set("redis_key",num1);
        }finally {
            // 判断是否是当前线程，如果是当前线程则允许释放锁
            if (redisTemplate.opsForvalue().get(REDIS_LOCK).equalsIgnoreCase(value)){
                redisTemplate.delete(REDIS_LOCK);
            }
        }
    }
```
实际上这种方式判断和删除的操作不是原子的，不是原子性的就会出现问题。即该锁没有保存持有者的唯一标识，可能被别的客户端解锁。

#### 方式五
针对方式四的问题，Redis官网有推荐的解决方法，即，[解锁Lua脚本](http://www.redis.cn/commands/set)：
```
if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then
   redis.call('expire',KEYS[1],ARGV[2])
else
   return 0
end;
```

```
    @Autowired
    private RedisTemplate redisTemplate;

    final String REDIS_LOCK = "redis_lock_demo";
    // 保证value值唯一,这里是伪代码
    final String value = "";
    public void context(){
        try {
            Boolean flag =  redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK,value, 10, TimeUnit.SECONDS);

            if (!flag) {
                System.out.println("抢锁失败！");
            }
            String redisKey =  redisTemplate.opsForvalue().get("redis_key");
            int num0 = redisKey == null ? 0 : Integer.parseInt(redisKey);

            if (num0 <= 0){
                System.out.println("商品已售完！");
                return;
            }

            // 卖出商品，存入Redis中
            int num1  = num0 - 1;
            redisTemplate.opsForvalue().set("redis_key",num1);
        }finally {
            // 伪代码
            JRedis = jedis = JRedisUtils.getJRedis();

            String lua_scripts =
                    "if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then\n" +
                    "   redis.call('expire',KEYS[1],ARGV[2])\n" +
                    "else\n" +
                    "   return 0\n" +
                    "end;";
            Object result = jedis.eval(lua_scripts, Collections.singletonList(REDIS_LOCK), Collections.singletonList(value))
            if (result.equals("1")) {
                System.out.println("删除key成功");
            }
                    
            if (jedis != null) {
                jedis.close();
            }

            if (redisTemplate.opsForvalue().get(REDIS_LOCK).equalsIgnoreCase(value)){
                redisTemplate.delete(REDIS_LOCK);
            }
        }
    }
```
除了用这中方式，也可以用Redis事务来处理方式四带来的问题。

对于上面的解决方法，其实并没有真正的解决缓存续期的问题，还是会带来能存在锁过期释放，业务没执行完的问题。

#### 方式六
参考文章：
- https://www.cnblogs.com/yunlongn/p/14609443.html

针对缓存续期的问题，我们可以开一个守护线程，每隔一段时间检查锁是否还存在，存在则对锁的过期时间延长，防止锁过期提前释放。

`Redisson`框架解决了这个问题：
```
    @Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    private Redisson redisson;

    final String REDIS_LOCK = "redis_lock_demo";
    // 保证value值唯一,这里是伪代码
    final String value = "";
    public void context(){
        try {
            RLock lock = redisson.getLock(REDIS_LOCK);
            lock.lock(REDIS_LOCK);

            String redisKey =  redisTemplate.opsForvalue().get("redis_key");
            int num0 = redisKey == null ? 0 : Integer.parseInt(redisKey);

            if (num0 <= 0){
                System.out.println("商品已售完！");
                return;
            }

            // 卖出商品，存入Redis中
            int num1  = num0 - 1;
            redisTemplate.opsForvalue().set("redis_key",num1);
        }finally {
            lock.unlock();
        }
    }
```

`Redisson`大致工作原理：
只要线程一加锁成功，就会启动一个`watch dog`看门狗，它是一个后台线程，会每隔10秒检查一下，如果线程一还持有锁，那么就会不断的延长锁key的生存时间。
因此，`Redisson`解决了锁过期释放，业务没执行完问题。
![Redisson原理图](/myblog/posts/images/essays/Redisson原理图.png)

看似完美的解决方案，但是在高并发下可能也会出现下面的异常：
```
Caused by: java.lang.IllegalMonitorStateException: attempt to unlock lock, not locked by current thread by node id: 32caba49-5799-491b-aa7b-47d789dbca93 thread-id: 1
```
异常出现的原因，加锁和解锁的线程不是同一个。

#### 方式七
针对上面的异常，需要判断当前线程是否持有锁，如果还持有，则释放，如果未持有，则说明已被释放:
```
    @Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    private Redisson redisson;

    final String REDIS_LOCK = "redis_lock_demo";
    // 保证value值唯一,这里是伪代码
    final String value = "";
    public void context(){
        try {
            RLock lock = redisson.getLock(REDIS_LOCK);
            lock.lock(REDIS_LOCK);

            String redisKey =  redisTemplate.opsForvalue().get("redis_key");
            int num0 = redisKey == null ? 0 : Integer.parseInt(redisKey);

            if (num0 <= 0){
                System.out.println("商品已售完！");
                return;
            }

            // 卖出商品，存入Redis中
            int num1  = num0 - 1;
            redisTemplate.opsForvalue().set("redis_key",num1);
        }finally {
            // 查询当前线程是否持有此锁
            if (lock.isLocked() && lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
```
这样写，程序的健壮性会更好，代码会更加严谨。


## Redis内存淘汰策略

### Redis内存满了
长期把Redis做缓存用，总有一天Redis内存总会满的。有没有相关这个问题，Redis内存满了会怎么样？

在`redis.conf`中把Redis内存设置为1个字节，做一个测试：
```
// 默认单位就是字节
maxmemory 1 
```
设置完之后重启为了确保测试的准确性，重启一下Redis，之后在用下面的命令，向Redis中存入键值对,模拟Redis打满的情况：
```
set k1 v1
```
执行完后会看到下面的信息：
```
(error) OOM command not allowed when used memory > 'maxmemory'.
```
大意：OOM，当当前内存大于最大内存时，这个命令不允许被执行。

是的，Redis也会出现OOM，正因如此，我们才要避免这种情况发生，正常情况下，不考虑极端业务，Redis不是MySql数据库，不能什么都往里边写，一般情况下Redis只存放热点数据。

Redis默认最大内存是全部的内存，我们在实际配置的时候，一般配实际服务器内存的3/4也就足够了。

### 删除策略
正因为Redis内存打满后报OOM，为了避免出现该情况所以要设置Redis的删除策略。思考一个问题，一个键到了过期时间之后是不是马上就从内存中被删除的？

当然不是的，那过期之后到底什么时候被删除？是个什么操作？

Redis提供了三种删除策略：
1. 定时删除：创建一个定时器，定时随机的对key执行删除操作；
2. 惰性删除：类似与懒加载，每次只有用到key的时候才会检查，该key是否已经过期，如果过期进行删除操作；
3. 定期删除：每隔一段时间，就会检查删除掉过期的key；

定时删除，即用时间换空间；它对于内存来说是友好的，定时清理出干净的空间，但是对于CPU来说并不是友好的，程序需要维护一个定时器，这就会占用CPU资源。

惰性的删除，即用空间换时间；它对于CPU来说是友好的，CPU不需要维护其它额外的操作，但是对于内存来说是不友好的，因为要是有些key一直没有被访问到，就会一直占用着内存。

定期删除，是上面两种方案的折中方案，它每隔一段时间删除过期的key，也就是根据具体的业务，合理的取一个时间定期的删除key。

若果在数据量很大的情况下，定时删除时，key从来没有被检查到过；惰性删除时，key从来没有被使用过，这样就会造成内存泄漏，大量的key堆积在内存中，导致Redis内存空间紧张。

所以我们必须有一个兜底方案，即Redis的内存淘汰策略。
### 淘汰策略
在 `Redis 4.0` 版本之前有 6 种策略，4.0 增加了 2种，主要新增了 `LFU` 算法。下图为 `Redis 6.2.0` 版本的配置文件：

![Redis内存淘汰策略](/myblog/posts/images/essays/Redis内存淘汰策略.png)

淘汰策略默认,使用`noeviction`，意思是不再驱逐的，即等着内存被打满。
```
The default is:
maxmemory-policy noeviction
```
| 策略名称               | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `noeviction`(默认策略) | 不会驱逐任何key，即内存满了就报错。   |
| `allkeys-lru`          | 所有key都是使用**LRU算法**进行淘汰。                         |
| `volatile-lru`         | 所有**设置了过期时间的key使用LRU算法**进行淘汰。             |
| `allkeys-random`       | 所有的key使用**随机淘汰**的方式进行淘汰。                    |
| `volatile-random`      | 所有**设置了过期时间的key使用随机淘汰**的方式进行淘汰。      |
| `volatile-ttl`         | 所有设置了过期时间的key**根据过期时间进行淘汰，越早过期就越快被淘汰**。 |

假如在Redis中的数据有一部分是热点数据，而剩下的数据是冷门数据，或者我们不太清楚我们应用的缓存访问分布状况，这时可以使用`allkeys-lru`。

可以在`redis.conf`配置文件中配置:
```
maxmemory-policy allkeys-lru   // 淘汰策略名字
```
当然也可以动态的配置，在Redis运行时修改：
```
// 设置内存淘汰策略
config set maxmemory-policy allkeys-lru

// 查看内存淘汰策略
config get maxmemory-policy
```

### LRU算法及实现
LRU是，`Least Recently Used`的缩写，即最近最少使用，是一种常用的 ***页面置换算法***。

>进程运行时，若其访问的页面不在内存而需将其调入，但内存已无空闲空间时，就需要从内存中调出一页程序或数据，送入磁盘的对换区，其中选择调出页面的算法就称为页面置换算法。

这个算法的思想就是： 如果一个数据在最近一段时间没有被访问到，那么在将来它被访问的可能性也很小。
所以，当指定的空间已存满数据时，应当把最久没有被访问到的数据淘汰。

明白了思想之后，要实现LRU算法，首先要确定数据结构，再确定实现思路。如果对算法有要求，查询和插入的时间复杂度都是`O(1)`，可以选用链表+哈希的结构来存储：

![LRU结构](/myblog/posts/images/essays/LRU结构.png)

#### 方式一
链表+哈希，我们不难想到JDK中的`LinkedHashMap`,在`LinkedHashMap`文档注释中找到关于LRU算法的相关描述：
>A special {@link #LinkedHashMap(int,float,boolean) constructor} is provided to create a linked hash map whose order of iteration is the order  in which its entries were last accessed,from least-recently accessed to most-recently (<i>access-order</i>).  This kind of map is well-suited to building LRU caches.

大意：`{@link #LinkedHashMap(int,float,boolean)}`提供了一个特殊的构造器来创建一个链表散列映射，其迭代顺序为其条目最后访问的顺序，从最近最少访问到最近最近(`access-order`)。这种映射非常适合构建LRU缓存。

参照`LinkedHashMap`实现LRU算法：
```
public class MainTest {

    public static void main(String[] args) {
        LRUDemo<Integer,String> list0 = new LRUDemo<>(3,true);
        System.out.println("-------------accessOrder等于true-------------");
        context(list0);
        System.out.println("-------------accessOrder等于false-------------");
        LRUDemo<Integer,String> list1 = new LRUDemo<>(3,false);
        context(list1);
    }
    
    public static void context(LRUDemo<Integer,String> list){
        list.put(1,"a");
        list.put(2,"b");
        list.put(3,"c");
        System.out.println(list.keySet());

        list.put(4,"d");
        System.out.println(list.keySet());
        System.out.println();

        list.put(3,"123");
        System.out.println(list.keySet());

        list.put(3,"1234");
        System.out.println(list.keySet());

        list.put(3,"12345");
        System.out.println(list.keySet());
        System.out.println();

        list.put(5,"123456");
        System.out.println(list.keySet());
    }
}

class LRUDemo<K,V> extends LinkedHashMap<K,V>{

    private int capacity;

    public LRUDemo(int capacity, boolean accessOrder) {
        /**
         * accessOrder the ordering mode -
         * <tt>true</tt> for access-order 存取顺序:如果存贮集合中有相同的元素，再次插入时先删除在插入
         * <tt>false</tt> for insertion-order 插入顺序：不会因为集合中有相同元素，再次插入该元素就会打乱位置
         */
        super(capacity,0.75f,accessOrder);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return super.size() > capacity;
    }
}
```

#### 方式二
除了可以参照`LinkedHashMap`，也可以自己手动实现：
```
public class MainTest {

    public static void main(String[] args) {
        LRUDemo<Integer,String> list0 = new LRUDemo<>(3);
        context(list0);
    }

    public static void context(LRUDemo<Integer,String> list){
        list.put(1,"a");
        list.put(2,"b");
        list.put(3,"c");
        System.out.println(list.getMap().keySet());

        list.put(4,"d");
        System.out.println(list.getMap().keySet());
        System.out.println();

        list.put(3,"123");
        System.out.println(list.getMap().keySet());

        list.put(3,"1234");
        System.out.println(list.getMap().keySet());

        list.put(3,"12345");
        System.out.println(list.getMap().keySet());
        System.out.println();

        list.put(5,"123456");
        System.out.println(list.getMap().keySet());
    }
}

class LRUDemo<K, V> {

    static class Node<K,V>{
        K key;
        V value;
        Node<K,V> prev;
        Node<K, V> next;

        public Node(){
            prev = next = null;
        }

        public Node(K key,V value){
            this.key = key;
            this.value = value;
        }
    }

    static class DoubleLinkedList<K,V>{
        Node<K,V> head;
        Node<K, V> tail;

        public DoubleLinkedList(){
            head = new Node<>();
            tail = new Node<>();
            
            // 如果变成尾插法，需要调换头、尾指针的指向
            head.next = tail;
            tail.prev = head;
        }

        public void putHead(Node<K, V> node){
            node.next = head.next;
            node.prev = head; // 将新结点插到头部
            head.next.prev = node;
            head.next = node;
        }

        public void remove(Node<K, V> node){
            node.prev.next = node.next;
            node.next.prev = node.prev;
            node.prev = node.next = null;
        }

        public Node<K,V> getLastNode(){
            return tail.prev;
        }
    }

    private int capacity;
    private Map<K, Node<K,V>> map;
    private DoubleLinkedList<K,V> doubleLinkedList;

    public Map<K, Node<K, V>> getMap() {
        return map;
    }

    public LRUDemo(int capacity){
        this.capacity = capacity;
        this.map = new HashMap<>();
        doubleLinkedList = new DoubleLinkedList<>();
    }

    public void put(K key,V val){
        if (key == null || val == null){
            return;
        }
        // 如果集合中key已经存在，则先删除
        if (map.containsKey(key)){
            Node<K, V> node = map.get(key);
            node.value = val;
            map.put(key,node);

            // 刷新node
            doubleLinkedList.remove(node);
            doubleLinkedList.putHead(node);
        }else {
            // 删除最少使用的key
            if (map.size() == capacity){
                Node<K, V> lastNode = doubleLinkedList.getLastNode();
                doubleLinkedList.remove(lastNode);
                map.remove(lastNode.key);
            }
            Node<K, V> newNode = new Node<>(key,val);
            map.put(key,newNode);
            doubleLinkedList.putHead(newNode);
        }
    }

    public V get(K key){
        if (!map.containsKey(key)){
            return null;
        }
        Node<K, V> node = map.get(key);

        // 刷新结点位置，将该key移动到队列头部
        doubleLinkedList.remove(node);
        doubleLinkedList.putHead(node);
        return node.value;
    }
}
```
## Redis持久化
