---
title: Spring Boot + Cassandra
author: HuyPVA
date: 2021-10-14
category: spring-boot
layout: post
source: https://github.com/huypva/spring-boot-cassandra-example
---

<div align="center">
    <img src="../assets/images/spring_boot_icon.png"/>
</div>

> Hướng dẫn tích hợp Cassandra trong Spring Boot application  

- Thêm dependency trong pom.xml
```xml
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-boot-starter-data-cassandra</artifactId>
    </dependency>
```

- Thêm cấu hình Cassandra trong file application.yml

```yml
spring:
  data:
    cassandra:
      contact-points: 127.0.0.1
      port: 9042
      keyspace-name: spring_boot_cassandra_example
```

- Tạo class entity mapping table trong cassandra

```java
@Data
@Table("User")
public class UserEntity {

  @PrimaryKey("user_id")
  private long userId;

  @Column("user_name")
  private String userName;

}
```

- Tạo Repository thao tác với table

```java
@Repository
public interface UserRepository extends CassandraRepository<UserEntity, Long> {

}
```

- Sử dụng Repository để thực hiện query đến database

```java
  @Autowired
  UserRepository userRepository;
    
  @Override
  public Greeting greet(long userId) {
    UserEntity userEntity = userRepository.findById(userId).get();
    return new Greeting(userEntity.getUserId(), String.format("Hello %s!", userEntity.getUserName()));
  }
```