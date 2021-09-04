---
title: Spring Data JPA trong Spring Boot
author: HuyPVA
date: 2021-08-30
category: spring-boot
layout: post
source: https://github.com/huypva/spring-boot-jpa-example
---

<div align="center">
    <img src="../assets/images/spring_boot_icon.png"/>
</div>

Hướng dẫn giao tiếp database trong Spring Boot

- Thư viện sử dụng:
  - [spring-data-jpa](https://spring.io/projects/spring-data-jpa)

- Định nghĩa dependency trong pom.xml

```xml
    <!--spring mvc, rest-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--spring data jpa-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <!--mysql connector java-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
```

- Cấu hình datasource & jpa trong application.yml

```yml
spring:
    datasource:
        url: "jdbc:mysql://localhost:3306/spring_boot_jpa_example"
        username: user
        password: password
        driver-class-name: com.mysql.cj.jdbc.Driver
    jpa:
        database-platform: org.hibernate.dialect.MySQL5Dialect
        show-sql: true
        hibernate:
          ddl-auto: none
``` 

- Tạo class entity tương ứng với table trong database

```java
@Entity
@Table(name = "user")
public class UserDto {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column(name = "user_id")
  private int userId;

  @Column(name = "user_name")
  private String userName;
}
``` 

- Tạo Repository thao tác với table

```java
public interface UserRepository extends JpaRepository<UserDto, Integer> {

  Optional<UserDto> findUserByUserName(String username);
}
```

- Query database thông qua repository

```java
  @Autowired
  private final UserRepository userRepository;
  
  @Override
  public UserDto getUserByName(String username) {
      return userRepository.findUserByUserName(username)
          .orElseThrow(() -> new UserNotFoundException(String.format("Not found user by name %s", username)));
  }
```