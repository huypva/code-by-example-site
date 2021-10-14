---
title: Spring Boot - AOP
author: HuyPVA
date: 2021-09-23
category: spring-boot
layout: post
source: https://github.com/huypva/spring-boot-aop-example
---

<div align="center">
    <img src="../assets/images/spring_boot_icon.png"/>
</div>

> Hướng dẫn sử dụng Spring AOP để xử lý cho annotation trong Spring Boot application

- Thư viện sử dụng
  - [spring-boot-starter-aop](https://docs.spring.io/spring-framework/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)

- Thêm dependency trong file pom.xml

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
		</dependency>
    
```

- Tạo annotation

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TimeLogger {

  String value() default "";
}
```

- Tạo class xử lý annotation

```java
@Aspect
@Component
public class TimeLoggerAspectJ {

  @Pointcut("@annotation(timeLogger)")
  public void annotatedTimeLogger(TimeLogger timeLogger) {
  }

  @Around("annotatedTimeLogger(timeLogger)")
  public Object around(ProceedingJoinPoint joinPoint, TimeLogger timeLogger) throws Throwable {
    String value = timeLogger.value();
    long startTime = System.currentTimeMillis();

    try {
      return joinPoint.proceed();
    } finally {

      long processTime = System.currentTimeMillis() - startTime;
      log.info(String.format("Process [%s] in [%s]", value, processTime));
    }

  }
}
```

- Thêm annotaion `EnableAspectJAutoProxy` để kích hoạt Aop

```java
@EnableAspectJAutoProxy
@SpringBootApplication
public class AopApplication {

	public static void main(String[] args) {
		SpringApplication.run(AopApplication.class, args);
	}

}
```