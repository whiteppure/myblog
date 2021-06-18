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

在代码里面我们常用`ReetrantLock、synchronized`保证线程安全。通过上面的锁，在某个时刻只有一个保证线程能够执行锁作用域内的代码。

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

举个例子，当很多个请求过来的时候，会先经过`Nginx`,然后`Nginx`在根据算法分发请求，到哪些服务器的程序上。
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

简单说就是，张冠李戴，删除了其他线程的锁。

#### 方式四
针对方式三带来的问题，需要加一个判断，来避免误删除其他线程的锁：
```
    @Autowired
    private RedisTemplate redisTemplate;;

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
    private RedisTemplate redisTemplate;;

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
只要线程一加锁成功，就会启动一个`watch dog`看门狗，它是一个后台线程，会每隔10秒检查一下，如果线程1还持有锁，那么就会不断的延长锁key的生存时间。
因此，`Redisson`就是使用`Redisson`解决了锁过期释放，业务没执行完问题。
![Redisson原理图](/myblog/posts/images/essays/Redisson原理图.png)

看似完美的解决方案，但是在高并发下偶尔也会出现下面的异常：
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


## Redis过期淘汰策略
因为Redis内存会打满，之后报OOM，为了避免出现类似的情况所以要设置Redis过期淘汰策略。


## Redis持久化
