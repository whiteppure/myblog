---
title: "SpringBoot整合redis"
date: 2020-03-01
draft: false
tags: ["springboot", "redis"]
slug: "springboot-redis"
---


## Redis介绍
redis是开源的一个高性能的 key-value 数据库。

### 主要特点
- Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用
- Redis支持数据的备份，即master-slave模式的数据备份
- Redis 可以存储键与5种不同数据结构类型之间的映射
    - **String** 可以是字符串、整数或者浮点数 对整个字符串或者字符串的其中一部分执行操作；对象和浮点数执行自增(increment)或者自减(decrement)
    - **List** 一个链表，链表上的每个节点都包含了一个字符串 从链表的两端推入或者弹出元素；根据偏移量对链表进行修剪(trim)；读取单个或者多个元素；根据值来查找或者移除元素
    - **Set** 包含字符串的无序收集器(unorderedcollection)，并且被包含的每个字符串都是独一无二的、各不相同 添加、获取、移除单个元素；检查一个元素是否存在于某个集合中；计算交集、并集、差集；从集合里卖弄随机获取元素
    - **Hash** 包含键值对的无序散列表 添加、获取、移除单个键值对；获取所有键值对
    - **Zset** 字符串成员(member)与浮点数分值(score)之间的有序映射，元素的排列顺序由分值的大小决定 添加、获取、删除单个元素；根据分值范围(range)或者成员来获取元素

**注意: Java bean 要序列化**

### Redis优点
- 性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。
- 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
- 原子 – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
- 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。

参考[菜鸟教程](https://www.runoob.com/redis/redis-intro.html)
 
## 导入依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.1.3.RELEASE</version>
</dependency>
```



## 配置redis

### RedisConfig.class

```
@Configuration
public class RedisConfig {

    //用于解决注解操作redis 序列话的问题
    @Bean(name = "myCacheManager")
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {

        RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory);
        RedisSerializer<Object> jsonSerializer = new GenericJackson2JsonRedisSerializer();
        RedisSerializationContext.SerializationPair<Object> pair = RedisSerializationContext.SerializationPair
                .fromSerializer(jsonSerializer);
        RedisCacheConfiguration defaultCacheConfig= RedisCacheConfiguration.defaultCacheConfig()
                .serializeValuesWith(pair);
        defaultCacheConfig.entryTtl(Duration.ofMinutes(30));
        return new RedisCacheManager(redisCacheWriter, defaultCacheConfig);
    }
```
    
    
```
    /**
 * 解决用redisTemplate操作的序列化的问题
 *
 * @param factory RedisConnectionFactory
 * @return redisTemplate
 */
    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory factory){
        // 配置redisTemplate
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(factory);
        // key序列化
        redisTemplate.setKeySerializer(STRING_SERIALIZER);
        // value序列化
        redisTemplate.setValueSerializer(JACKSON__SERIALIZER);
        // Hash key序列化
        redisTemplate.setHashKeySerializer(STRING_SERIALIZER);
        // Hash value序列化
        redisTemplate.setHashValueSerializer(JACKSON__SERIALIZER);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }

}
```


### 启动类配置

```
@EnableCaching  //允许注解操作缓存
@SpringBootApplication
public class RedisExampleApplication {
   public static void main(String[] args) {
      SpringApplication.run(RedisExampleApplication.class, args);
   }
}
```

### application.yml 配置

```
#redis 缓存配置
redis:
  database: 0
  host: @ip
  port: 6379
  timeout: 8000
  #如果没有可不写
  password:
  jedis:
    pool:
      #连接池最大连接数量
      max-active: 10
      #连接池最大堵塞时间
      max-wait: -1
      #连接池最小空闲连接
      min-idle: 0
      #连接池最大空闲连接
      max-idle: 8
```


## 缓存注解介绍

### @Cacheable
`@Cacheable`作用将方法的运行结果进行缓存,以后要相同的数据,直接从缓存中获取.
 cacheNames 和 key 都必须填，如果不填 key ，默认的 key 是当前的方法名，更新缓存时会因为方法名不同而更新失败.

属性
- value:   指定缓存名称 可指定多个(数组)
- key:  缓存数据使用的key默认为方法参数支持 spEL
- keyGenerator:  key的生成器
- CacheManager:  指定缓存管理器,或获取解析器,两者二选一
- condition:  指定条件缓存
- unless:  否定缓存. 当unless条件为true时,返回值就不会被缓存,可获取到结果判断
- sync:  是否异步, 不支持unless

key 也可以动态设置为方法的参数(支持EL)
```
@Cacheable(cacheNames = "xxx", key = "#openid")
public Response detail(@RequestParam("openid") String openid){
      //to do sthing
}
```

```
@Cacheable(cacheNames = "xxx", key = "xxx",keyGenerator = "xxx", CacheManager = "xxx", condition = "xxx", unless = "xxx", sync = "xxx")
```


### @CachePut
既调用方法又更新缓存修改了数据库某个数据.
更新缓存,先调用目标方法将目标方法缓存起来,更新时要注意,要和查询时的key相同,否则缓存不会更新,属性和 @Cacheable 相同
```
@CachePut(cacheNames = "xxx", key = "xxx",keyGenerator = "xxx", CacheManager = "xxx", condition = "xxx", unless = "xxx", sync = "xxx")
```
### @CacheEvict
该注解作用是删除缓存，需要指定 key
- allEntries:是否删除所有字段的缓存
- beforeInvocation:是否在方法之前执行
```
@CacheEvict(cacheNames = "xxx", key = "xxx", allEntries = "xxx", beforeInvocation="xxx")
```

### @CacheConfig
`@CacheConfig `可指定公共key的生成策略
```
// 公共的cacheNames (value) 可以统一写在类上面 这样就不用每缓存一个就起名字了
// 公共的CacheManager
@CacheConfig(cacheNames = "xxx")
public class RedisExampleController {
  
    @Caching(cacheable = {@Cacheable}, put = {@CachePut})
    public Response<Map<String, Object>> test(){
          //to do
    }
}
```

### 复合注解 
```
// 查询数据时 先去更新数据 在放到缓存中
@Caching(cacheable = {@Cacheable}, put = {@CachePut})
@Caching(cacheable = {@Cacheable}, put = {@CachePut}, evict = {@CacheEvict})
```

## 测试

```
@Autowired
private RedisTemplate redisTemplate;

 @Test 
public void demo(){

    //常见redis类型数据操作 set zset 未列出
        
    //string 类型数据
    redisTemplate.opsForValue().set("test","123");
    redisTemplate.opsForValue().get("test") // 输出结果为123
    
   //list 数据类型
   redisTemplate.opsForList().rightPushAll("list",new String[]{"1","2","3"});//从右边插入元素
   redisTemplate.opsForList().range("list",0,-1);//获取所有元素
   redisTemplate.opsForList().index("listRight",1);//获取下标为2的元素
   redisTemplate.opsForList().rightPush("listRiht","1");//从右边插入 也可从左边插
   redisTemplate.opsForList().leftPop("list");//从左边弹出元素 元素弹出将不存在

   //hash
   redisTemplate.opsForHash().hasKey("redisHash","111");//判断该hash key 是否存在
   redisTemplate.opsForHash().put("redisHash","name","111");//存放 hash 数据
   redisTemplate.opsForHash().keys("redisHash");//获取该key对应的hash值
   redisTemplate.opsForHash().get("redisHash","age");//给定key 获取 hash 值
 
 
}
```