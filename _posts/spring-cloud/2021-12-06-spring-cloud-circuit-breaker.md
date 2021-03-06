---
title: Spring Boot - Circuit Breaker with Resilience4j
author: HuyPVA
date: 2021-12-06
category: spring-cloud
layout: post
source: https://github.com/huypva/spring-cloud-circuit-breaker-example
---

> Hướng dẫn áp dụng Circuit Breaker bằng Resilience4j


## Thư viện sử dụng

- Spring Boot: version 2.4.11
- Spring Cloud: version 2020.0.4
    - org.springframework.cloud:spring-cloud-starter-openfeign
    - org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j

## Tạo client service-A

- Thêm config cho OpenFeign client và Resilience4j

```yaml
serviceb:
  url: http://localhost:8082
  path:
    greet: /greet/{id}
feign:
  client:
    config:
      serviceb:
        connectTimeout: 3000
        readTimeout: 3000
resilience4j.circuitbreaker:
  instances:
    serviceB:
      registerHealthIndicator: true
      slidingWindowSize: 5
      permittedNumberOfCallsInHalfOpenState: 3
      slidingWindowType: COUNT_BASED
      minimumNumberOfCalls: 5
      waitDurationInOpenState: 10s
      failureRateThreshold: 50
```

- Enable OpenFeign client

```java
@EnableFeignClients
@SpringBootApplication
public class ServiceAApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServiceAApplication.class, args);
	}

}
```

- Tạo OpenFeign client và thêm `@CircuitBreaker`

```java
@FeignClient(value = "serviceb", url = "${serviceb.url}")
public interface ServiceBClient {

  @CircuitBreaker(name = "serviceB", fallbackMethod = "fallback")
  @RequestMapping(method = RequestMethod.GET, value = "${serviceb.path.greet}")
  public MessageB greet(@PathVariable(name = "id") int userId);

  default MessageB fallback(Throwable throwable) {
    return new MessageB("Fallback method");
  }
}
```

## Tạo server service-B đơn giản

```java
@RestController
public class Controller {

  @GetMapping("/greet/{id}")
  public MessageB greet(@PathVariable(name = "id") int id) throws InterruptedException {
    log.info("ServiceB.greet");
    Thread.sleep(id);
    return new MessageB("MessageB greet " + id);
  }
}
```

## Send request test 

## Ý nghĩa các giá trị config

![](../assets/images/spring_cloud/circuit_breaker_state_machine.jpeg)

  
    - registerHealthIndicator: true
    - failureRateThreshold: ngưỡng tỉ lệ lỗi, lớn hơn hoặc bằng ngưỡng này CircuitBreaker sẽ bật OPEN
    - slowCallDurationThreshold: khoảng thời gian 1 call được gọi là slow
    - slowCallRateThreshold: ngưỡng tỉ lệ request slow để CircuitBreaker bật OPEN
    - permittedNumberOfCallsInHalfOpenState: số lượng call khi CircuitBreaker đang HALF-OPEN
    - maxWaitDurationInHalfOpenState: thời gian tối đa CircuitBreaker ở trạng thái HALF-OPEN
    - slidingWindowType: cấu hình kiểu sliding window (COUNT_BASED, TIME_BASED)
    - slidingWindowSize: size mỗi sliding window
    - minimumNumberOfCalls: số lượng call tối thiểu trong sliding window mới tính 
    - waitDurationInOpenState: thời gian CircuitBreaker đợi chuyển từ OPEN sang HALF-OPEN 
    - ignoreExceptions: danh sách Exception ko được tính lỗi
    - recordFailurePredicate: một custom Predicate để đánh giá lỗi hay khôn

## Reference
- <https://resilience4j.readme.io/docs/circuitbreaker>