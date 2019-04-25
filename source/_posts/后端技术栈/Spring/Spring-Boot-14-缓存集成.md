---
title: Spring Boot-14-缓存集成
date: 2018-07-11 17:57:39
tags: Spring Boot
---

为了提高性能，减少数据库的压力，使用缓存是非常好的手段之一。

## Spring 缓存抽象
Spring 定义 CacheManager 和 Cache 接口用来统一不同的缓存技术。例如 JCache、 EhCache、 Hazelcast、 Guava、 Redis 等。

关于Spring对缓存的抽象体系，可以参考Spring分类中【Spring 缓存抽象】。

在使用 Spring 集成 Cache 的时候，我们需要注册实现的 CacheManager 接口 的 Bean。

## Spring Boot 自动配置 CacheManager
Spring Boot 为各种缓存实现提供了自动配置机制。

在spring-boot-autoconfigure.jar的org.springframework.boot.autoconfigure.cache包下，提供了如下的自动配置类。
- JcacheCacheConfiguration
- EhCacheCacheConfiguration
- HazelcastCacheConfiguration
- GuavaCacheConfiguration
- RedisCacheConfiguration
- SimpleCacheConfiguration 

如果Spring 容器中存在RedisTemplate的Bean, 即classpath下引入了Redis相关jar，则RedisCacheConfiguration满足条件，会创建相应的RedisCacheManager实现。

Spring 默认使用 ConcurrentMapCacheManager 作为CacheManager实现，其中采用ConcurrentMap作为缓存。
