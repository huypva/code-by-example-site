---
title: Spring Boot - RESTful API
author: HuyPVA
date: 2021-09-14
category: spring-boot
layout: post
source: https://github.com/huypva/spring-boot-rest-example
---

<div align="center">
    <img src="../assets/images/spring_boot_icon.png"/>
</div>

> Hướng dẫn xây dựng RESTful service bằng Spring Boot

### Configuration

- Thêm dependency trong pom.xml
```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
``` 

- Thêm cấu hình port public trong file application.yml
```yml
server:
  port : 8081
```

### Write controller

- Viết class controller bằng cách sử dụng *RestController*
```java
@RestController
@RequestMapping("/api")
public class Controller {

  @Autowired
  GreetUseCase greetUseCase;

  @GetMapping("/get-mapping")
  public Greeting getMapping() {
    return greetUseCase.greet(0, "GetMapping");
  }
}
```

*RequestMapping* và *GetMapping* tạo nên path api cung cấp (Ví dụ: /api/get-mapping) 

### Mapping request

- Tạo mapping lấy variable từ path của api
```java
  @GetMapping("/path-variable/{user_id}")
  public Greeting pathVariable(@PathVariable(name = "user_id") int userId) {
    return greetUseCase.greet(userId, "PathVariable");
  }
```

- Tạo api nhận request param
```java
  @GetMapping("/request-param")
  public Greeting requestParam(@RequestParam(name = "user_id") int userId) {
    return greetUseCase.greet(userId, "RequestParam");
  }
```

- Tạo post request
```java
  @PostMapping("/request-body")
  public Greeting requestBody(@RequestBody User user) {
    return greetUseCase.greet(user.getUserId(), "RequestBody");
  }
```