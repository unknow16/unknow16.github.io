---
title: Spring Cloud-09-Zuul-多纬度限流
date: 2018-03-27 17:26:35
tags: Spring Cloud
---

- 对请求的目标URL进行限流（例如：某个URL每分钟只允许调用多少次）
- 对客户端的访问IP进行限流（例如：某个IP每分钟只允许请求多少次）
- 对某些特定用户或者用户组进行限流（例如：非VIP用户限制每分钟只允许调用100次某个API等）
- 多维度混合的限流。此时，就需要实现一些限流规则的编排机制。与、或、非等关系。

[link](https://my.oschina.net/giegie/blog/1583705)
