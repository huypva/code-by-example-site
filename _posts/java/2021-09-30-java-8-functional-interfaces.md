---
title: Java 8 - Functional Interfaces
author: HuyPVA
date: 2021-09-30
category: java
layout: post
source: https://github.com/huypva/java-8-functional-interfaces-example
---

<div align="center">
    <img src="../assets/images/java/functional_interfaces.png"/>
</div>

> Function Interfaces trong Java 8

## Functional Interfaces

Functional Interface là các Interface có duy nhất một method trừu trượng, được đánh dấu bởi annotation *@FunctionalInterface*

Sử dụng Functional Interface thông qua Lambda Expressions

- Dạng 1

```
(args) -> {block code}
```

- Dạng 2: dành cho `block code` chỉ có 1 dòng lệnh

```
(args) -> code
```

Tiếp theo là một số Functional Interfaces có sẵn trong Java 8 thường được sử dụng

## Supplier

- *Suplier* chứa method trừu tượng không tham số, và return về một đối tượng

```java
@FunctionalInterface
public interface Supplier<T> {

    T get();
}
```

- Ví dụ: sử dụng functional interface Supplier

```java
  public void println(Supplier<String> supp) {
    System.out.println(supp.get());
  }

  public void beforeJava8() {
    println(new Supplier<String>(){

      @Override
      public String get() {
        return "Example";
      }
    });
  }

  public void java8() {
    println(() -> "Example");
  }
```

## Consumer

- *Consumer* chứa method trừu tượng có một tham số đầu vào và không return (void method). 

```java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);
}
```

- Ví dụ: print một List trong java

```java
  List<String> list = Arrays.asList("a", "b", "c", "d", "e");

  public void beforeJava8() {
    list.forEach(new Consumer<String>() {
      @Override
      public void accept(String s) {
        System.out.print(s + "\t");
      }
    });
    System.out.println();
  }

  public void java8() {
    list.forEach(s -> System.out.print(s + "\t"));
    System.out.println();
  }
```

## Predicate

- *Predicate* chứa method trừu tượng có một tham số đầu vào, và return boolean (true/false)

```java
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);
}
```

- Ví du: lọc các số lẻ trong một list Integer

```java
  List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);

  public void beforeJava8() {
    Stream<Integer> stream2 = list.stream().filter(new Predicate<Integer>() {
      @Override
      public boolean test(Integer t) {
        return t % 2 == 1;
      }
    });
  }

  public void java8() {
    Stream<Integer> stream2 = list.stream().filter(t -> {
      return t % 2 == 1;
    });
  }
```

## Function

- *Function* chứa method trừu tượng có một tham số đầu vào và return một tham số khác

```java
@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);
}
```

- Ví dụ: upper case danh sách các String

```java
  List<String> list = Arrays.asList("a", "c", "B", "D", "e");

  public void beforeJava8() {
    Stream<String> stream2 = list.stream().map(new Function<String, String>() {
      @Override
      public String apply(String s) {
        return s == null ? null : s.toUpperCase();
      }
    });
  }

  public void java8() {
    Stream<String> stream2 = list.stream().map(s -> s == null ? null : s.toUpperCase());
  }
```

## Comparator

- Trước Java 8, *Comparator* là một Interface được sử dụng nhiều để so sánh 2 tham số. 
Từ Java 8, Functional interface *Comparator* chứa method trừu tượng có 2 tham số (a, b) và return kiểu int với ý nghĩa
    - `1`: a>b
    - `0`: a=b
    - `-1`: a < b

```java
@FunctionalInterface
public interface Comparator<T> {

    int compare(T o1, T o2);
}
```

- Ví dụ: sắp xếp danh sách số nguyên

```java
  List<Integer> list = Arrays.asList(1, 5, 2, 6, 3, 4, 7);

  public void beforeJava8() {
    Collections.sort(list, new Comparator<Integer>() {
      @Override
      public int compare(Integer o1, Integer o2) {
        return o1.compareTo(o2);
      }
    });
  }

  public void java8() {
    Collections.sort(list, (o1, o2) -> o1.compareTo(o2));
  }
```