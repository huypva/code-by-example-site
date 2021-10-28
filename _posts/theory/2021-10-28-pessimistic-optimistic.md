---
title: Theory - Pessimistic vs Optimistic
author: HuyPVA
date: 2021-10-18
category: theory
layout: post
---

<div align="center">
    <img src="../assets/images/theory/sonar_qube.png"/>
</div>


<div align="center">
    <img src="../assets/images/theory/pessimistic_optimistic.png"/>
</div>

> Phân biệt Pessimistic và Optimistic Lock 

## Pessimistic Lock

- Là cơ chế sử dụng lock record khi write record

- Cách thực hiện
<div align="center">
    <img src="../assets/images/theory/pessimistic.png"/>
</div>    
  

## Optimistic Lock

- Là cơ chế sử dụng version (hoặc một giá trị tương đương) để kiểm tra record có bị thay đổi không trước khi write nó.

- Cách thực hiện như sau
<div align="center">
    <img src="../assets/images/theory/optimistic.png"/>
</div>  

## Usage

- Optimistic Locking
    - Sử dụng trong trường hợp ít xảy ra conflict giữa 2 transaction
    - Chỉ kiểm tra data khi write (UPDATE, DELECT) nên gây ra inconsistent khi read dữ liệu
    
- Pessimistic Locking
    - Sử dụng trong trường hợp có khả năng xảy ra conflict cao
    - Performaince thấp do chi phí acquire lock
    - Cẩn thận khi áp dụng, có khả năng xảy ra deadlock khi acquire trên nhiều data

## Reference

- https://stackoverflow.com/questions/129329/optimistic-vs-pessimistic-locking
- https://martinfowler.com/eaaCatalog/optimisticOfflineLock.html
- https://martinfowler.com/eaaCatalog/pessimisticOfflineLock.html