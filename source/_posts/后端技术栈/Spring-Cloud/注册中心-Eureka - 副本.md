---
title: 注册中心-Eureka
date: 2018-01-11 22:18:08
tags: Spring Cloud
---

---
### Service Discovery: Eureka
```
 register  动词，注册
 
 registry 名词，注册中心，注册表
 
 ${spring.application.name} 就是 Service ID 或 virtual host （VIP）
 
 eureka server : 它包含一个服务注册表和一个REST API，可以用来注册一个服务，注销一个服务，并发现其他服务的位置。
 eureka client : service ID 标识一个client，可包含多个实例
 eureka instance : 一个微服务application的单个启动实例，随着负载增长可启动多个水平缩放。
 eureka service : 当你注册一个微服务application成一个client,你能通过注册的service ID获取一个eureka service。
```

### Eureka Server Standalone

* spring-cloud-starter-eureka-server
* @EnableEurekaServer

```
eureka server的application.yml配置

server:
  port: 8761

eureka:
  instance:
    hostname: localhost
    preferIpAddress: true # 让client注册时，使用ip替代默认的hostname
  client:
    registerWithEureka: false #单机禁用自己注册自己
    fetchRegistry: false #单机时不从对等点拉取registry
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```
* 默认服务注册中心也会将自己作为客户端来尝试注册它自己，并且需要（至少一个）服务URL来定位对等方，单机我们需要禁用它的客户端注册行为，不禁用日志中会出现异常提醒。
* eureka server和client会通过心跳机制保持通讯，client每30秒发送一次心跳去续约，如果90秒内server不能收到client的心跳，则会将该client踢出注册表，自我保护模式下不会踢出失效的client
* eureka server没有后端存储，但registry中eureka instance都必须发送检测信号去使其注册保持最新状态（这些在内存中完成），eureka client还拥有一个rigistry的内存缓存，所以不必每次都去请求rigistry。

---

### Eureka Server HA

```
eureka server的application.yml配置

spring:
  profiles:
    active: peer1
  application:
    name: eureka-server-ha
    
---

spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
    preferIpAddress: true # 让client注册时，使用ip替代默认的hostname
  client:
    serviceUrl:
      defaultZone: http://peer2/eureka/
server:
  port: 1001

---

spring:
  profiles: peer2
  application:
    name:
eureka:
  instance:
    hostname: peer2
    preferIpAddress: true # 让client注册时，使用ip替代默认的hostname
  client:
    serviceUrl:
      defaultZone: http://peer1/eureka/
server:
  port: 1002
```
* 你能多个peer到系统中，只要它们互相至少通过一条边连接，它们之间能同步注册表。如果这些peers中物理上隔离的，系统原则上会存在脑裂类型失败。

### Eureka Client
* spring-cloud-starter-eureka
* @EnableEurekaClient/@EnableDiscoveryClient

```
discovery client的application.properties

spring.application.name=eureka-client-provider
server.port=2001
eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/  #服务注册中心的位置
```
* defaultZone,为任何不表示偏好的客户提供服务URL（即，这是一个有用的默认值）。
* client通过RestTemplate来真正发起REST请求

### 授权访问
* spring-boot-starter-security

```
eureka server 新增

security.basic.enabled=true
security.user.name=user
security.user.password=pwd
defaultZone=http://user:pwd@localhost:8761/eureka

eureka client 配置curl style like

http://user:pwd@localhost:8761/eureka
```

### 为什么注册一个服务很慢？
* 成为一个instance涉及一个到注册中心的周期性心跳，通过client的serverUrl, 默认间隔30秒，
一个服务不可用于被其他clients发现，直到instance，server，client所有的在本地缓存中有相同的metadata,因此会花费3个心跳，你能改变这个周期，通过设置eureka.instance.leaseRenewalIntervalInSeconds=30,这将会加速client连接到其他服务的进度，
在生产上，使用默认的可能更好，因为在server上有一些计算间隔使关于续约周期的猜想。


