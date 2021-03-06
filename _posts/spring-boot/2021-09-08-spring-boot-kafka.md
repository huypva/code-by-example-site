---
title: Spring Boot + Kafka
author: HuyPVA
date: 2021-09-08
category: spring-boot
layout: post
source: https://github.com/huypva/spring-boot-kafka-example
---

<div align="center">
    <img src="../assets/images/spring_boot/spring_boot_icon.png"/>
</div>

> Hướng dẫn tích hợp Kafka trong Spring Boot application

### Configuration

- Thêm dependency trong pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
</dependencies>
``` 

- Thêm cấu hình kafa trong file application.yml

```yml
spring:
  kafka:
    bootstrap-servers: "localhost:9093"
    listener:
      missing-topics-fatal: false
```

### Write a producer

- Sử dụng bean `KafkaTemplate` để gửi một message lên topic `UserMessage` của Kafka

```java
    @Autowired
    KafkaTemplate<String, String> userKafka;
  
    @Override
    public void sendUserMesage(UserMessage message) {
      userKafka.send("UserMessage", GsonUtils.toJson(message))
          .addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
  
            @Override
            public void onSuccess(SendResult<String, String> result) {
              log.info("Kafka sent message='{}' with offset={}", message,
                  result.getRecordMetadata().offset());
            }
  
            @Override
            public void onFailure(Throwable ex) {
              log.error("Kafka unable to send message='{}'", message, ex);
            }
          });
    }
```

### Write a consumer

```java
  @Autowired
  UserUseCase userUseCase;
  
  @KafkaListener(topics = "UserMessage", groupId = "example")
  public void consume(ConsumerRecord<String, String> record) {
    try {
      log.info(
          "Consumed - Partition: {} - Offset: {} - Value: {}",
          record.partition(),
          record.offset(),
          record.value());
  
      UserMessage userMessage = GsonUtils.fromJson(record.value(), UserMessage.class);
      userUseCase.goodbye(userMessage);
  
    } catch (Exception ex) {
      log.error("Exception - Reason:", ex);
    }
  }    
```
