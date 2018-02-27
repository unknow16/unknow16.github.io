---
title: Zuul-路由配置
date: 2018-02-27 17:23:58
tags: spring cloud
---

### Route  Endpoint
需包含Spring Boot Actuator
* Get请求 /routes将会返回映射路由表
* Post请求将会强制刷新现有route,在服务路由中有更改的情况时

默认该endpoint会有安全限制，两种解决方案：
1. 设置 management.security.enabled: false
2. add security-starter，设置user/pwd

* endpoints.routes.enabled=false.会禁用此端点
* 服务路由改变时会自动刷新，但post是一种强制改变立即生效的方法

### 忽略部分service

```
 zuul:
  ignoredServices: '*'
  routes:
    users: /myusers/**
```
- 所有服务被忽略，除了users
- users为注册中心的服务名
- url为 /myusers/** 会映射给serviceId 为users的微服务

### 单独指定path和serviceId

```
 zuul:
  routes:
    users:
      path: /myusers/**
      serviceId: users_service
```
- users只是一个标记，users_service为注册中心的服务名
- url为 /myusers/** 会映射给serviceId 为users的微服务

### 请求path映射到非服务发现中的url上

```
 zuul:
  routes:
    users:
      path: /myusers/**
      url: http://example.com/users_service
```
此时请求不能在 HystrixCommand中执行，也不能用ribbon负载均衡请求

- 若要实现则需手动维护

```
zuul:
  routes:
    users:
      path: /myusers/**
      serviceId: users

ribbon:
  eureka:
    enabled: false

users:
  ribbon:
    listOfServers: http://localhost:2001, http://localhost:2002
```

### 代码路由配置

```
@Bean
public PatternServiceRouteMapper serviceRouteMapper() {
    return new PatternServiceRouteMapper(
        "(?<name>^.+)-(?<version>v.+$)",
        "${version}/${name}");
}
```
如，请求path为“/v1/myusers/**”被映射到serviceId为myusers-v1服务上

### 设置全局访问前缀
- zuul.prefix= /api  一定要以/开头
- 默认这个前缀会被zuul剥离掉，再转发给服务
- 也可设置 zuul.stripPrefix=false， 不剥离
- 也能单独指定为某个服务不剥离

```
 zuul:
  routes:
    users:
      path: /myusers/**
      stripPrefix: false
```
- zuul.stripPrefix使用的前提是设置了zuul.prefix	
- X-Forwarded-Host请求头会默认添加在转发的请求中，可设置zuul.addProxyHeaders = false，不让其添加
- zuul.route.home: /        将会所有的请求（/**）都会to "Home"Service

### 精确忽略
- 如果需要更细粒度的忽略，则可以指定要忽略的特定模式。 
- 这些模式在路由定位过程开始时被评估，这意味着模式中应包含前缀以保证匹配。 
- 忽略的模式跨越所有服务并取代任何其他路由规范。

```
 zuul:
  ignoredPatterns: /**/admin/**
  routes:
    users: /myusers/**
```
意味着包含admin的path将被忽略

### 匹配顺序
如果需要配置的路由保留其声明的顺序，必须使用yaml file, propertes file不能保留

```
 zuul:
  routes:
    users:
      path: /myusers/**
    legacy:
      path: /**
```
上面的配置如果用properties file，会导致访问/myusers/的路径不可用
