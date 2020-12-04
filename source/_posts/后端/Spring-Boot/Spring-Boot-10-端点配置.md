---
title: Spring Boot-10-端点配置
date: 2018-07-09 20:12:00
tags: Spring Boot
---

## 基础依赖
Spring Boot 监控核心是 spring-boot-starter-actuator 依赖，增加依赖后， Spring Boot 会默认配置一些通用的监控，比如 jvm 监控、类加载、健康监控等。
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
因为使用 HTTP 调用的方式，还需要 spring-boot-starter-web 依赖。


```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
另外，访问相应端点，默认开启了权限控制，需引入spring-boot-starter-security模块，并配置用户名密码。

```
// pom.xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>

// application.yml
security:
  basic:
    enabled: true
  user:
    name: fuyi
    password: fuyi
```

## 端点访问路径
#### 应用配置类端点
HTTP方法|	路径|	描述
---|---|---
GET|	/autoconfig|	查看应用的自动配置的使用情况
GET|	/beans|	查看应用的所有Bean的信息
GET|	/configprops|	查看应用的所有配置属性
GET|	/env|	查看应用的所有环境信息
GET|	/mappings|	查看所有url映射
GET|	/info|	查看应用信息
GET|    /loggers| 显示当前应用程序所有包和类的日志级别
GET|	/logfile|	查看日志文件的内容，需要设置logging.file或logging.path属性

#### 度量指标类
HTTP方法|	路径|	描述
---|---|---
GET|	/dump|	查看应用的线程状态信息
GET|	/health|	查看应用健康指标
GET|	/metrics|	查看应用基本指标
GET|	/trace|	查看基本的HTTP跟踪信息
GET|	/heapdump|	返回一个gzip压缩 hprof 堆转储文件
GET|    /auditevents|  显示当前应用程序授权时间的统计信息


#### 其他
HTTP方法|	路径|	描述
---|---|---
POST|	/shutdown|	允许优雅关闭当前应用（默认情况下不启用）
GET|	/actuator|	查看所有EndPoint的列表，需要加入spring-hateoas支持
GET|	/docs|	查看文档，需要依赖 spring-boot-actuator-docs
GET|	/flyway|	查看已经有迁徙路线数据库迁移
GET|	/liquibase|	查看已经有liquibase数据库迁移应用
GET|	/jolokia|	暴露JMX bean（当jolokia路径）


```
// 开启/shutdown端点
endpoints:
  shutdown:
    enabled: true

// 使用/actuator端点要引入的pom
<dependency>
	<groupId>org.springframework.hateoas</groupId>
	<artifactId>spring-hateoas</artifactId>
</dependency>

// 使用/doc端点要引入的pom
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-actuator-docs</artifactId>
</dependency>
```

## 端点配置
每个端点都可以在application.yml中进行定制。每个端点都有特定配置。

端点的前缀是：endpoints + “.”+ 端点名。如下为health端点的可配置项：
```
endpoints.health.enabled= # Enable the endpoint.
endpoints.health.id= # Endpoint identifier.
endpoints.health.mapping.*= # Mapping of health statuses to HTTP status codes. By default, registered health statuses map to sensible defaults (i.e. UP maps to 200).
endpoints.health.path= # Endpoint path.
endpoints.health.sensitive= # Mark if the endpoint exposes sensitive information.
endpoints.health.time-to-live=1000 # Time to live for cached result, in milliseconds.
```

如下启用/shutdown端点，修改/health端点访问路径为/heat:
```
endpoints:
  shutdown:
    enabled: true
  health:
    path: /heat
```

## management.* 常用配置
1. 修改端点访问的全局上下文路径

默认的端点访问路径是根目录下，我们可以通过修改配置，进行定制。

```
management.context-path=/manage
```
再访问http://localhost:8080/health时，就应该变为http://localhost:8080/manage/health

2. 分别配置管理端点端口和业务端口
默认管理端点端口和业务端口是一样的，公用一个端口，出于安全性考虑，可以改变端点的访问的端口。
```
management.port=9090
```
我们甚至可以关闭 http 端点。
```
management.port=-1
```



