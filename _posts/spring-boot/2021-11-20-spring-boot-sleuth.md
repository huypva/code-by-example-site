---
title: Spring Boot - Spring Boot Sleuth
author: HuyPVA
date: 2021-11-20
category: spring-boot
layout: post
source: https://github.com/huypva/spring-boot-sleyth-example
---

<div align="center">
    <img src="../assets/images/spring_boot/spring_boot_icon.png"/>
</div>

> Hướng dẫn tích hợp [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) trong ứng dụng Spring Boot

## Mô hình
- Trong ví dụ này bao gồm 3 service
    - ServiceA: một service public 2 api và gọi vào ServiceB, ServiceC
    - ServiceB: một http service
    - ServiceC: một grpc service
    
    
- Thêm dependency spring-cloud-starter-sleuth trong mỗi service

```html
	<properties>
		<spring-cloud.version>2020.0.4</spring-cloud.version>
	</properties>
	<dependencies>
        ...
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
        ...
	</dependencies>
    
    <dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```

## Service A  

- Thêm dependencies sử dụng trong grpc
    - org.springframework.cloud:spring-cloud-starter-openfeign: dành cho http client
    - net.devh:grpc-spring-boot-starter: dùng chp grpc client 
    
```xml
    <properties>
    		<net.devh.grpc.starter.version>2.12.0.RELEASE</net.devh.grpc.starter.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>net.devh</groupId>
            <artifactId>grpc-spring-boot-starter</artifactId>
            <version>${net.devh.grpc.starter.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
		<dependency>
			<groupId>io.zipkin.brave</groupId>
			<artifactId>brave-instrumentation-grpc</artifactId>
		</dependency>
    </dependencies>
```

Mặc định OpenFeign tích hợp sẵn sleuth, còn với grpc cần sử dụng thêm dependency io.zipkin.brave:brave-instrumentation-grpc

- Cấu hình sleuth trong application.yml

```yaml
spring:
  sleuth:
    propagation:
      type: B3 #,W3C
    sampler:
      probability: 1.0
```

- Sử dụng FeignClient để tạo http client gọi đến ServiceB 
    
```java
@FeignClient(value = "serviceb", url = "${serviceb.url}")
public interface ServiceBClient {

  @RequestMapping(method = RequestMethod.GET, value = "${serviceb.path.greet}")
  public MessageB greet(@PathVariable(name = "id") int userId);

}
```

- Viết grpc client gọi đến ServiceC

```java
import net.devh.boot.grpc.client.inject.GrpcClient;
...
  @GrpcClient("servicec")
  private ServiceCBlockingStub serviceCBlockingStub;

  @Override
  public MessageC greet(int id) {
    GreetRequest request = GreetRequest.newBuilder()
        .setId(id)
        .build();

    GreetResponse response = this.serviceCBlockingStub.greet(request);
    return new MessageC(response.getMessage());
  }
```

## Service B

- Cấu hình client trong file application.yml

```yaml
serviceb:
  url: http://localhost:8082
  path:
    greet: /greet/{id}
```

- Viết controller đơn giản

```java
@RestController
public class Controller {

  @GetMapping("/greet/{id}")
  public MessageB greet(@PathVariable(name = "id") int id) {
    log.info("ServiceB.greet");
    return new MessageB("MessageB greet " + id);
  }
}
```

## ServiceC

- Cấu hình grpc client trong file application.yml

```yaml
grpc:
  client:
    servicec:
      address: static://localhost:8183
      enableKeepAlive: true
      keepAliveWithoutCalls: true
      negotiationType: plaintext
```

- Viết gRPC controller

```java
import net.devh.boot.grpc.server.service.GrpcService;
...
@GrpcService
public class GrpcController extends ServiceCGrpc.ServiceCImplBase {

  @Override
  public void greet(GreetRequest request, StreamObserver<GreetResponse> responseObserver) {
    log.info("ServiceC.greet " + request.getId());
    GreetResponse response = GreetResponse.newBuilder()
        .setMessage("Message C")
        .build();

    responseObserver.onNext(response);
    responseObserver.onCompleted();
  }

}
```

## Test sleuth

- Start 3 service A, B, C

- Gửi request test đến ServiceA, trong ServiceA sẽ gọi đến ServiceB

```shell script
$ curl http://localhost:8081/greetb/1
```

Log service A:
```text
2021-11-20 21:01:03.834  INFO [service-A,48c90d4d61668476,48c90d4d61668476] 2433 --- [nio-8081-exec-1] i.c.servicea.entrypoint.Controller       : ServiceA.greetB
```
và service B
```text
2021-11-20 21:01:03.939  INFO [service-B,48c90d4d61668476,c73d7b8f5c34adc6] 2429 --- [nio-8082-exec-1] i.c.serviceb.entrypoint.Controller       : ServiceB.greet
```
chung span 48c90d4d61668476

- Gửi request test đến ServiceA, trong ServiceA sẽ gọi đến ServiceC

```shell script
$ curl http://localhost:8081/greetc/1
```

Log service A:
```text
2021-11-20 21:02:54.170  INFO [service-A,9b6f0e1e5e5ca2da,9b6f0e1e5e5ca2da] 2433 --- [nio-8081-exec-2] i.c.servicea.entrypoint.Controller       : ServiceA.greetC
```
và service C
```text
2021-11-20 21:02:54.340  INFO [service-C,9b6f0e1e5e5ca2da,9cd977fdc2849bb4] 2100 --- [ault-executor-2] i.c.servicec.entrypoint.GrpcController   : ServiceC.greet 1
```
chung span 9b6f0e1e5e5ca2da

## Report lên Zipkin, Jaeger

- Setup Zipkin bằng docker-compose

```yaml
version: "3.4"
services:
  zipkin:
    image: openzipkin/zipkin
    ports:
      - 9411:9411
```

Run docker-compose

```shell script
docker-compose up -d
```

- Trong mỗi service, thêm thông tin dependency 

```xml
<dependencies>
    ...
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-sleuth-zipkin</artifactId>
    </dependency>
</dependencies>
```

- Thêm cấu hình zipkin trong application.yml

```
spring:
  zipkin:
    base-url: http://localhost:9411
```

- Send request và mở Zipkin UI tại địa chỉ http://localhost:9411/ lên xem report
<div align="center">
    <img src="../assets/images/spring_boot/zipkin_service_b.png"/>
</div>
<div align="center">
    <img src="../assets/images/spring_boot/zipkin_service_c.png"/>
</div>

- Ngoài ra, có thể sử dụng Jeager server để hiện thị report
Setup Jeager server bằng docker
```yaml
version: "3.4"
services:
  jaeger:
    image: jaegertracing/all-in-one:1.17
    ports:
      - 5775:5775/udp
      - 6831:6831/udp
      - 6832:6832/udp
      - 5778:5778
      - 16686:16686
      - 14268:14268
      - 14250:14250
      - 9411:9411
    environment:
      - COLLECTOR_ZIPKIN_HTTP_PORT=9411
```

Test và xem kết quả trên Jeager UI tại địa chỉ http://localhost:16686/

<div align="center">
    <img src="../assets/images/spring_boot/jaeger_a_b.png"/>
</div>
<div align="center">
    <img src="../assets/images/spring_boot/jaeger_a_c.png"/>
</div>

## Reference
- https://spring.io/projects/spring-cloud-sleuth