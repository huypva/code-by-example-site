---
title: Spring Boot + Jeager
author: HuyPVA
date: 2021-08-30
category: spring-boot
layout: post
source: https://github.com/huypva/grpc-tracing-example
---

<div align="center">
    <img src="../assets/images/spring_boot_icon.png"/>
</div>

> Hướng dẫn tích hợp OpenTracing tracing Grpc API trong Spring Boot application 

Thư viện sử dụng:
- [grpc-spring-boot-starter](https://github.com/yidongnan/grpc-spring-boot-starter) - Spring Boot starter module for gRPC framework
- [opentracing-contrib/java-spring-cloud](https://github.com/opentracing-contrib/java-spring-cloud) - OpenTracing instrumentation for Spring Boot
- [opentracing-contrib/java-grpc](https://github.com/opentracing-contrib/java-grpc) - OpenTracing instrumentation for gRPC

Định nghĩa dependency trong pom.xml
```xml
    #server service only
    <dependency>
        <groupId>net.devh</groupId>
        <artifactId>grpc-spring-boot-starter</artifactId>
        <version>VERSION</version>
    </dependency>
    #client service only
    <dependency>
        <groupId>net.devh</groupId>
        <artifactId>grpc-client-spring-boot-starter</artifactId>
        <version>VERSION</version>
    </dependency>
    #both client and server
    <dependency>
        <groupId>io.opentracing.contrib</groupId>
        <artifactId>opentracing-grpc</artifactId>
        <version>VERSION</version>
    </dependency>
    <dependency>
        <groupId>io.opentracing.contrib</groupId>
        <artifactId>opentracing-spring-jaeger-cloud-starter</artifactId>
        <version>VERSION</version>
    </dependency>
```

## Server

- Cấu hình OpenTracing trong application.properties
```properties
opentracing.jaeger.service-name=...
opentracing.jaeger.enabled=true
opentracing.jaeger.udp-sender.host=localhost
opentracing.jaeger.udp-sender.port=6831
opentracing.jaeger.log-spans=true
opentracing.jaeger.enable-b3-propagation=false
opentracing.jaeger.probabilistic-sampler.sampling-rate=1.0
``` 

- Tạo grpc Controler

```java
@GrpcService
public class GrpcController extends SimpleGrpc.SimpleImplBase {
 
  @Override
  public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
    HelloReply reply = HelloReply.newBuilder().setMessage("Hello ==> " + req.getName()).build();
    responseObserver.onNext(reply);
    responseObserver.onCompleted();
  }
 
}
``` 

- Tạo và intercept một `TracingServerInterceptor` vào GrpcControler thông qua annotation `GrpcGlobalServerInterceptor`

```java
  @Autowired
  private Tracer tracer;
 
  @GrpcGlobalServerInterceptor
  TracingServerInterceptor tracingServerInterceptor() {
    TracingServerInterceptor tracingInterceptor = TracingServerInterceptor
      .newBuilder()
      .withTracer(tracer)
      .withStreaming()
      .withVerbosity()
      .withTracedAttributes(ServerRequestAttribute.HEADERS,
          ServerRequestAttribute.METHOD_TYPE)
      .build();
 
    return tracingInterceptor;
  }
```

## Client

- Tạo grpc-client

```java
  @GrpcClient("service-name")
  private SimpleBlockingStub simpleStub;
   
  @Override
  public String hello(String name) {
    try {
      final HelloReply response = this.simpleStub.sayHello(HelloRequest.newBuilder().setName(name).build());
      return response.getMessage();
    } catch (final StatusRuntimeException e) {
      return "FAILED with " + e.getStatus().getCode().name();
    }
  }
```

- Tạo `TracingClientInterceptor` và intercept vào grpc-client thông qua annotation `GrpcGlobalClientInterceptor`

```java
  @Autowired
  private Tracer tracer;
 
  @GrpcGlobalClientInterceptor
  TracingClientInterceptor tracingServerInterceptor() {
    TracingClientInterceptor tracingInterceptor = TracingClientInterceptor
      .newBuilder()
      .withTracer(tracer)
      .withStreaming()
      .withTracedAttributes(ClientRequestAttribute.ALL_CALL_OPTIONS, ClientRequestAttribute.HEADERS)
      .build();
 
    return tracingInterceptor;
  }
```