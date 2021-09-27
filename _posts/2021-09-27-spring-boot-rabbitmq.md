---
title: Spring Boot - RabbitMQ
author: HuyPVA
date: 2021-09-27
category: spring-boot
layout: post
source: https://github.com/huypva/spring-boot-rabbitmq-example
---

<div align="center">
    <img src="../assets/images/spring_boot_icon.png"/>
</div>

> Hướng dẫn tích hợp RabbitMQ trong Spring Boot application

- Thư viện sử dụng
  - [spring-boot-starter-amqp](https://spring.io/guides/gs/messaging-rabbitmq/)

- Thêm dependency trong file pom.xml

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-amqp</artifactId>
		</dependency>
    
```

- Thêm configuration sử dụng rabbitmq trong file application.yml

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
```

- Tạo các bean *Queue*, *Exchange* và *Binding*

```java
@Configuration
public class RabbitConfiguration {

  public String queueName = "GREETING_QUEUE";
  public String exchangeName = "GREETING_EXCHANGE";
  public String routingKey = "greet.*";

  @Bean
  Queue queue() {
    return new Queue(queueName, false);
  }

  @Bean
  TopicExchange exchange() {
    return new TopicExchange(exchangeName);
  }

  @Bean
  Binding binding(Queue queue, TopicExchange exchange) {
    return BindingBuilder.bind(queue).to(exchange).with(routingKey);
  }

}
```

- Tạo provider sử dụng *RabbitTemplate* để send message đến RabbitMQ

```java
@Component
public class GreetingRabbitProviderImpl implements GreetingRabbitProvider {

  @Autowired
  RabbitConfiguration rabbitConfiguration;

  @Autowired
  RabbitTemplate rabbitTemplate;

  @Override
  public void sendGreeting(Greeting greeting) {
    rabbitTemplate.convertAndSend(
        rabbitConfiguration.exchangeName,
        rabbitConfiguration.routingKey,
        GsonUtils.toJson(greeting));
  }
}
```

- Sử dụng annotation *RabbitListener* để tạo consumer

```java
@Slf4j
@Component
public class GreetingRabbitListener {

  @RabbitListener(queues = "#{rabbitConfiguration.queueName}")
  public void greet(String greetingMessage) {
    log.info("Listen message from Rabbit: {}", greetingMessage);

    Greeting greeting = GsonUtils.fromJson(greetingMessage, Greeting.class);
    log.info("Message id={}, message={}", greeting.getId(), greeting.getMessage());
  }

}
```