---
title: "Springboot整合kafka"
date: 2020-08-20
draft: false
tags: ["springboot", "MQ"]
slug: "springboot-kafka"
---


## Windows平台kafka环境搭建
- https://kafka.apachecn.org
- https://blog.csdn.net/u010054969/article/details/70241478

## MacOS平台kafka环境搭建

### 安装kafka

kafka依赖Java和zookeeper，安装前请先安装Java和zookeeper。

安装zookeeper
```
brew install zookeeper
```

安装kafka
```
brew install kafka
```

### 启动kafka

假定你是按照上边的方法安装的kafka,接下来启动kafka

因为kafka依赖zk，所以首先要启动zookeeper
- cd到该路径：`cd /usr/local/Cellar/zookeeper/xxx/bin` xxx是zookeeper的版本
- 执行启动命令 `zkServer start`，停止命令为 `zkServer stop`


**启动之前必须修改kafka的配置文件，否则后面的启动会失败**
- 执行该命令 ``vim /usr/local/etc/kafka/server.properties``
- 增加``listeners=PLAINTEXT://localhost:9092`` 其中有一行，默认被注释掉了，打开修改即可

最后启动kafka
- cd到该路径：`cd /usr/local/Cellar/kafka/xxx/bin` xxx是kafka的版本
- 执行该命令启动kafka ``kafka-server-start /usr/local/etc/kafka/server.properties &``

到此为止kafka启动完成

### kafka消息测试

- 创建test主题 `kafka-topics --create --zookeeper localhost:2181`<br>` --replication-factor 1 --partitions 1 --topic test`
- 查看主题是否创建成功 ``kafka-topics --list --zookeeper localhost:2181`` 该命令会列出所有的主题
- 在kafka/bin目录，topic为test的主题 ``kafka-console-producer --topic test --broker-list localhost:9092``
- 打开新的终端，在kafka/bin目录，topic为test的主题 ``kafka-console-consumer --bootstrap-server localhost:9092 -topic test ``

在生产者终端输入信息，在消费者终端就能看的见
<div style="width: 50%;display: inline-block">
    <img src="/myblog/posts/images/application/kafka生产者.jpg" alt="生产者">
</div>
<div style="width: 45%;display: inline-block">
    <img src="/myblog/posts/images/application/kafka消费者.jpg" alt="消费者">
</div>

## springboot整合kafka

### 引入pom依赖
```
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
```
### 配置application.yml
```
server:
  port: 8080
  servlet:
    context-path: /demo
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      retries: 0
      batch-size: 16384
      buffer-memory: 33554432
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      group-id: spring-boot-demo
      # 手动提交
      enable-auto-commit: false
      auto-offset-reset: latest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      properties:
        session.timeout.ms: 60000
    listener:
      log-container-config: false
      concurrency: 5
      # 手动提交
      ack-mode: manual_immediate
```

### 创建kafka配置类
```
public interface KafkaConsts {
    /**
     * 默认分区大小
     */
    Integer DEFAULT_PARTITION_NUM = 3;

    /**
     * Topic 名称
     */
    String TOPIC_TEST = "test";
}
```

```
@Configuration
@EnableConfigurationProperties({KafkaProperties.class})
@EnableKafka
@AllArgsConstructor
public class KafkaConfig {
    private final KafkaProperties kafkaProperties;

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    @Bean
    public ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(kafkaProperties.buildProducerProperties());
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(KafkaConsts.DEFAULT_PARTITION_NUM);
        factory.setBatchListener(true);
        factory.getContainerProperties().setPollTimeout(3000);
        return factory;
    }

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(kafkaProperties.buildConsumerProperties());
    }

    @Bean("ackContainerFactory")
    public ConcurrentKafkaListenerContainerFactory<String, String> ackContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);
        factory.setConcurrency(KafkaConsts.DEFAULT_PARTITION_NUM);
        return factory;
    }

}
```
### 创建消息处理器
```
@Component
@Slf4j
public class MessageHandler {

    @KafkaListener(topics = KafkaConsts.TOPIC_TEST, containerFactory = "ackContainerFactory")
    public void handleMessage(ConsumerRecord record, Acknowledgment acknowledgment) {
        try {
            String message = (String) record.value();
            log.info("收到消息: {}", message);
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        } finally {
            // 手动提交 offset
            acknowledgment.acknowledge();
        }
    }
}
```

### 创建测试类
测试之前请确保kafka已启动
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootDemoMqKafkaApplicationTests {
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    /**
     * 测试发送消息
     */
    @Test
    public void testSend() {
        kafkaTemplate.send(KafkaConsts.TOPIC_TEST, "hello,kafka...");
    }

}
```