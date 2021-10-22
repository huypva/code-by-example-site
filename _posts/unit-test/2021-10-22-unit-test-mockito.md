---
title: Unit Test + Mockito
author: HuyPVA
date: 2021-10-21
category: unit-test
layout: post
source: https://github.com/huypva/unit-test-mockito-example
---

<div align="center">
    <img src="../assets/images/junit_5.png"/>
</div>

> Hướng dẫn thực hiện UnitTest trong SpringBoot application

## Dependency

- Thêm dependency trong pom.xml để thực hiện UnitTest

```html
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
```

Trong Spring Boot version 2.2.6 đã có sẵn thư viện cho junit 5

<div align="center">
    <img src="../assets/images/junit_5_libs.png"/>
</div>

## Mock bean

- Sử dụng annotation `@Mockbean` để mock bean

```java
@ExtendWith(SpringExtension.class)
class UserCaseTest {

  @MockBean
  DataProvider dataProvider;

  @Autowired
  UserCase userCase;

  @Test
  void returnMethod() {
    Mockito.when(dataProvider.getCount())
        .thenReturn(2);

    Assertions.assertEquals(2, userCase.returnMethod());
  }
}
```

## Mock static class

- Sử dụng `MockedStatic` như sau

```java
  try (MockedStatic<[Class cần mock]> mockedStatic = Mockito.mockStatic([Class cần mock].class); ) {

      //mock function bằng cách sử dụng mockedStatic 

    
  }
```

Ví dụ: Giả sử cần test class `MockStatic` sử dụng static class `StaticClass`

```java
public class MockStatic {

  public String doMockStatic() {
    return "Static: " + StaticClass.doStaticMethod(1);
  }

}
```

Viết unit test như sau:

```java
@ExtendWith(SpringExtension.class)
class MockStaticTest {

  @Test
  void doMockStatic() {
    try (MockedStatic<StaticClass> mockedStatic = Mockito.mockStatic(StaticClass.class); ) {

      mockedStatic
          .when(() -> StaticClass.doStaticMethod(1))
          .thenReturn("1");

      MockStatic mockStatic = new MockStatic();
      Assertions.assertEquals("Static: 1", mockStatic.doMockStatic());
    }
  }
}
```

## Mock return function 

- Giả lập function theo format

```java
Mockito.when(...).thenReturn(...);
```

Ví dụ:

```java
  @Test
  void returnMethod_Count_Return2() {
    Mockito.when(dataProvider.getCount())
        .thenReturn(2);

    Assertions.assertEquals(2, userCase.returnMethod());
  }
```

## Mock void function

- Giả lập void function theo format

```java
Mockito.doNothing().when(...);
```

Ví dụ:

```java
  @Test
  void voidMethod_() {
    Mockito.doNothing().when(dataProvider).increase(1);
    
    //...
  }
```
