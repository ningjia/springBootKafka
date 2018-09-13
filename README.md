# SpringBoot整合Kafka
## 一、搭建Kafka服务
详细参见[Kafka基础（安装与配置）.md](/docs/Kafka基础（安装与配置）.md)

## 二、与SpringBoot进行整合
### 配置文件（application.yml）
```text
    spring:
      kafka:
        bootstrap-servers: 172.16.115.185:9092
        producer:
          key-serializer: org.apache.kafka.common.serialization.StringSerializer
          value-serializer: org.apache.kafka.common.serialization.StringSerializer
        consumer:
          group-id: test
          enable-auto-commit: true
          auto-commit-interval: 1000
          key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
          value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```
### 生产者代码
```java
    @RestController
    @RequestMapping("kafka")
    public class TestKafkaProducerController {
    
        @Autowired
        private KafkaTemplate<String, String> kafkaTemplate;
    
        @RequestMapping("send")
        public String send(String msg){
            kafkaTemplate.send("test_topic", msg);
            return "success";
        }
    
    }
```
### 消费者代码
```java
    @Component
    public class TestConsumer {
    
        @KafkaListener(topics = "test_topic")
        public void listen (ConsumerRecord<?, ?> record) throws Exception {
            System.out.printf("topic = %s, offset = %d, value = %s \n", record.topic(), record.offset(), record.value());
        }
    
    }
```
### 测试
1. 运行项目，执行：http://localhost:8080/kafka/send?msg=hello
2. 控制台输出：
```text
    opic = test_topic, offset = 19, value = hello 
```

## 三、Refer
- [SpringBoot整合Kafka](https://blog.csdn.net/saytime/article/details/79950635)
