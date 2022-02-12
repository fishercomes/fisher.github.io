---
title: 消息中间件之Kafka
date: 2022-02-12
---
### Kafka的下载与安装

#### 官网下载

 http://kafka.apache.org/downloads

#### Window环境下的运行(单机)

1. 由于Kafka依赖于Zookeeper，需要先运行Zookeeper,安装kafka的根目录

   ```cmd
   .\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
   ```

2. 启动Kafka

   ```cmd
   .\bin\windows\kafka-server-start.bat .\config\server.properties
   ```

### SpringBoot集成Kafka

1. 引入依赖

   ```xml
     <dependency>
         <groupId>org.springframework.kafka</groupId>
         <artifactId>spring-kafka</artifactId>
       </dependency>
   ```

2. 配置application.yml

   ```yml
   spring: 
     kafka:
      producer:
        key-serializer: org.apache.kafka.common.serialization.StringSerializer
        value-serializer: org.apache.kafka.common.serialization.StringSerializer
        retries: 3
      bootstrap-servers: 127.0.0.1:9092
      consumer:
        group-id: NOTIFICATION-CONSUME-GROUP
        key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
        value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
   ```

3. 生产者demo

   ```java
   @RestController
   @RequestMapping("/kafka")
   @Slf4j
   public class KafkaController {
   
       @Resource
       private KafkaTemplate<String, String> kafkaTemplate;
   
       /**
        * 模拟Kafka 生产者发送消息到broker
        * @param msg
        */
       @PostMapping("send/{msg}")
       public void sendMsg(@PathVariable(name = "msg") String msg) {
           log.info("Kafka发送消息:{}", msg);
           MessageDTO messageDTO = new MessageDTO();
           messageDTO.setMsgContent(msg);
           kafkaTemplate.send(KafkaTopicConstant.FISHER_TOPIC, JSON.toJSONString(messageDTO)).addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
               @Override
               public void onFailure(Throwable throwable) {
                   log.info("Kafka发送消息回调,消息发送失败");
               }
   
               @Override
               public void onSuccess(SendResult<String, String> stringStringSendResult) {
                   log.info("Kafka发送消息回调,消息发送成功");
               }
           });
       }
   }
   ```

4. 消费者demo

   ```java
   @Slf4j
   @Component
   public class KafkaMsgListener {
   
       /**
        * 模拟Kafka 消费者消费消息
        * @param msg
        */
       @KafkaListener(topics = KafkaTopicConstant.FISHER_TOPIC)
       public void listen(String msg) {
           MessageDTO messageDTO = JSON.parseObject(msg, MessageDTO.class);
           log.info("Kafka消费者消费消息:{}", messageDTO.getMsgContent());
       }
   }
   ```

   