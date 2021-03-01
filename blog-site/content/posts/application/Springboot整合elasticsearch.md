---
title: "Springboot整合elasticsearch"
date: 2020-02-09
draft: false
tags: ["springboot", "elasticsearch"]
slug: "springboot-elasticsearch"
---
这里只做整合,不做安装elasticsearch下载安装请移步到 
[下载安装ES](https://www.elastic.co/cn/downloads/elasticsearch)

# 1.导入依赖
要注意导入依赖的版本和安装`elasticsearch`的版本与springboot的兼容问题
![springbootelasticsearch.jpg](/myblog/posts/images/application/elasticsearch对应springboot版本.jpg)


```
这里测试的是elasticsearch2.1.3 与springboot 2.1.3

//gradle
compile group: 'org.springframework.boot', name: 'spring-boot-starter-data-elasticsearch', version: '2.1.3.RELEASE'

//pom
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
      <version>2.1.3.RELEASE</version>
</dependency>

//springboot版本
plugins {
    id 'org.springframework.boot' version '2.1.3.RELEASE'
    id 'java'
    id 'war'
}

```



# 2.导入成功之后进行测试
创建测试Javabean 重点: `@Document`注解
```
/**
 * 借助该javabean 完成 crud
 *
 * 测试elasticsearch用
 */
@ToString
@Data
@NoArgsConstructor
@AllArgsConstructor

/** 注意(坑点): ES 6以后不允许一个索引下有多个类型,只允许一个索引下有一个类型
 *
 *
 * String indexName(); //索引库的名称，个人建议以项目的名称命名
 *
 * String type() default ""; //类型，个人建议以实体的名称命名
 *
 * short shards() default 5; //默认分区数
 *
 * short replicas() default 1; //每个分区默认的备份数
 *
 * String refreshInterval() default "1s"; //刷新间隔
 *
 * String indexStoreType() default "fs"; //索引文件存储类型
 */
@Document(indexName = "shopping-mall-test", type = "test")
public class EsDemo implements Serializable {

    private static final long serialVersionUID = 1L;


    private Integer id;

    private String name;

    private Integer  age;

    private String sex;

}

```


- 创建接口
```
/**
 * ElasticsearchRepository<?,?>
 *
 * 第一个?: 写 Javabean
 * 第二个?: 写 该Javabean对应的主键类型
 */
@Component
public interface TestRepository extends ElasticsearchRepository<EsDemo,Integer> {
    // 自定义接口
}

创建测试controller
@RestController
@Slf4j
@RequestMapping("/elasticsearch/")
public class ElasticSearchController {

    @Autowired
    private TestRepository testRepository;

    @GetMapping("test")
    public Response<ProductInfo> test(){
        EsDemo test = new EsDemo(1, "张三", 20, "男");
        testRepository.index(test);
        //testRepository 调用elastic很多方法 前提是要先写入到elasticsearch里面 用index,save ....
//        testRepository.delete(Entity entity);
//        testRepository.findAll()
//        testRepository.search(条件)
        return new Response<>();
    }
}

```


# 3.配置application.yml
 ```
spring:
    # 使用spring-data 操作ElasticSearch配置
    data:
      elasticsearch:
        # 结点名称 默认为elasticsearch
        cluster-name: elasticsearch
        #节点的地址 注意api模式下端口号是9300，千万不要写成9200
        cluster-nodes: @ip:@port
        #是否开启本地存储
        repositories:
          enabled: true
```

