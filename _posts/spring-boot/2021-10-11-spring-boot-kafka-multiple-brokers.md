---
title: Spring Boot + Kafka with multiple brokers
author: HuyPVA
date: 2021-10-11
category: spring-boot
layout: post
source: https://github.com/huypva/spring-boot-kafka-multiple-brokers-example
---

<div align="center">
    <img src="../assets/images/spring_boot/kafka_multi_brokers.png"/>
</div>

> Hướng dẫn Spring Boot application tích hợp với nhiều Kafka brokers khác nhau 

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

- Thêm cấu hình Kafka trong file application.yml, ở đây có 2 Kafka khác nhau

```yml
user-kafka:
  bootstrap-servers: "localhost:19093"
  listener:
    missing-topics-fatal: false
bank-kafka:
  bootstrap-servers: "localhost:29093"
  listener:
    missing-topics-fatal: false
```

- Tạo class load properties cho broker thứ nhất

```java
@Configuration
@ConfigurationProperties(prefix = "user-kafka")
public class UserKafkaProperties extends KafkaProperties {

}
```

và với broker thứ 2

```java
@Configuration
@ConfigurationProperties(prefix = "bank-kafka")
public class BankKafkaProperties extends KafkaProperties {

}
```

- Để sử dụng được nhiều brokers, chúng ta sẽ tự tạo các bean cần thiết cho Producer và Consumer. Trước hết, disable cơ chế auto configuration cho Kafka

```java
@SpringBootApplication(exclude = KafkaAutoConfiguration.class)
public class KafkaMultipleBrokersApplication {
  
    public static void main(String[] args) {
		SpringApplication.run(KafkaMultipleBrokersApplication.class, args);
	}
}
```

### Tạo Producer

- Tạo class Configuration để tạo bean *KafkaTemplate* cho broker thứ nhất

```java
@Component
public class UserKafkaConfiguration {

  @Autowired
  UserKafkaProperties properties;

  //Producer
  @Bean("userKafkaTemplate")
  public KafkaTemplate<?, ?> kafkaTemplate() {
    KafkaTemplate<Object, Object> kafkaTemplate = new KafkaTemplate(kafkaProducerFactory());
    kafkaTemplate.setDefaultTopic(this.properties.getTemplate().getDefaultTopic());
    return kafkaTemplate;
  }

  public ProducerFactory<?, ?> kafkaProducerFactory() {
    DefaultKafkaProducerFactory<?, ?> factory = new DefaultKafkaProducerFactory(this.properties.buildProducerProperties());
    String transactionIdPrefix = this.properties.getProducer().getTransactionIdPrefix();
    if (transactionIdPrefix != null) {
      factory.setTransactionIdPrefix(transactionIdPrefix);
    }

    return factory;
  }
```

- Tạo class *Producer* autowired bean *KafkaTemplate* để gửi message lên broker

```java
@Service
public class UserKafkaProducerImpl implements UserKafkaProducer {

  @Autowired
  @Qualifier("userKafkaTemplate")
  KafkaTemplate<String, String> userKafkaTemplate;

  @Override
  public void sendUserMesage(UserMessage userMessage) {
    userKafkaTemplate.send("USER_TOPIC", GsonUtils.toJson(userMessage))
        .addCallback(new ListenableFutureCallback<SendResult<String, String>>() {

          @Override
          public void onSuccess(SendResult<String, String> result) {
            log.info("Kafka sent message='{}' with offset={}", GsonUtils.toJson(userMessage),
                result.getRecordMetadata().offset());
          }

          @Override
          public void onFailure(Throwable ex) {
            log.error("Kafka unable to send message='{}'", GsonUtils.toJson(userMessage), ex);
          }
        });
  }

}
```

- Tương tự tạo các class `BankKafkaConfiguration`, `BankKafkaProducerImpl` cho broker thứ 2

### Tạo Consumer

- Tạo các bean *ConcurrentKafkaListenerContainerFactory*, *ConsumerFactory* cần thiết cho Consumer trong class `UserKafkaConfiguration` 

```java
@Component
public class UserKafkaConfiguration {

  @Autowired
  UserKafkaProperties properties;

  //... code create bean for Producer

  @Bean("userKafkaListenerContainerFactory")
  ConcurrentKafkaListenerContainerFactory<?, ?> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<Object, Object> factory = new ConcurrentKafkaListenerContainerFactory();
    factory.setConsumerFactory(kafkaConsumerFactory());
    return factory;
  }

  public ConsumerFactory<Object, Object> kafkaConsumerFactory() {
    return new DefaultKafkaConsumerFactory(this.properties.buildConsumerProperties());
  }
}
```

- Viết class Consumer

```java
@Service
public class UserKafkaConsumer {

  @Autowired
  UserUseCase userUseCase;

  @KafkaListener(topics = "USER_TOPIC", groupId = "example",
      containerFactory = "userKafkaListenerContainerFactory")
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
}
```

- Tương tự, thêm bean trong `BankKafkaConfiguration`, và tạo class `BankKafkaConsumer` cho broker thứ 2

- Enable Kafka cho project

```java
@EnableKafka
public class KafkaMultipleBrokersApplication {
  //...
}
```


