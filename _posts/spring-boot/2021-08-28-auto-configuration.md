---
title: Spring Boot - Auto-Configuration
author: HuyPVA
date: 2021-08-28
category: spring-boot
layout: post
source: https://github.com/huypva/spring-boot-auto-configuration-example
---

<div align="center">
    <img src="../assets/images/spring_boot/spring_boot_icon.png"/>
</div>

> Tạo lớp configuration và tự động load từ file trong Spring Boot

- Thêm configuration trong file application.yml

```yml
my-config:
    id: 1
    value: my value
``` 

- Tạo class tương ứng với configuration

```java
public class MyConf {

  public int id;
  public String value;

}
``` 

- Sử dụng annotation `@Configuration`, `@ConfigurationProperties` và các hàm setter để *Spring Boot* biết và tự động tạo bean tương ứng 

```java
@Setter
@Configuration
@ConfigurationProperties(prefix = "my-config")
public class MyConf {
...
```

- Sử dụng bean *MyConf* thông qua annotation *Autowired* 

```java
  @Autowired
  MyConf myConf;
  ...
  System.out.println("Id: " + myConf.getId());
  System.out.println("Value: " + myConf.getValue());
```

Output:

```
Id: 1
Value: my value
```