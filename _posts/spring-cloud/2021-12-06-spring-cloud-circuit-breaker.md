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

- Ý nghĩa các giá trị config
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

- https://resilience4j.readme.io/docs/circuitbreaker