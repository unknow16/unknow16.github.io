---
title: Spring Cloud-05-Spring Cloud Config（Dalston.SR5）
date: 2018-01-17 15:31:50
tags: Spring Cloud
---

## Spring Cloud Config（Dalston.SR5）
## 特点
* 集中管理
* 不同环境不同配置
* 运行期间动态调整配置
* 自动刷新

## 同类项目
* apollo (携程)
* diamond(阿里)
* disconf(百度)
* zookeeper
* spring cloud consul

## URL访问
* /{application}/{profile}[/{label}]
* /{application}-{profile}.yml

* /{label}/{application}-{profile}.yml
* /{application}-{profile}.properties
* /{label}/{application}-{profile}.properties

## 结合Git基本使用
* client 通过用 uri / profile / label/ application.name等信息去 config server 请求配置属性
* config server根据client提供的信息按约定的匹配从git repo加载相应配置，返回client

```
config server配置application.yml

server:
  port: 7777
  
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/unknow16/spring-cloud-demo
          # username: git repo的用户名
          # password: git repo的密码

# 配置client访问需要的用户名密码，name可不配
security:
  basic:
    enabled: true
  user:
    password: pwd
```


```
config client配置bootstrap.yml

spring:
  cloud:
    config:
      uri: http://localhost:7777 # 或curl style: http://fuyi:pwd@localhost:7777
      # spring.cloud.config.password/username将会覆盖curl style中的
      username: fuyi
      password: pwd
      profile: prod
      label: master
  application:
    name: simple
```

## 结合DiscoveryClient(Eureka)
* config server 注册到eureka server
* config client 启用discovery,并配置config server在注册中心的service-id
* config client 配置profile/label/application.name找到config server获取配置

```
config server配置application.yml

server:
  port: 7777
  
spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/fuyiyi/{application} # {application}-{profile}
  application:
    name: config-server

# 连接eureka server          
eureka:
  client:
    service-url:
      defaultZone: http://user:password@localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```



```
config client配置bootstrap.yml

spring:
  cloud:
    config:
      profile: prod
      label: master
      fail-fast: true
      discovery:
        enabled: true # 默认false，设为true表示使用注册中心中的configserver配置而不自己配置configserver的uri
        service-id: config-server # 指定config server在服务发现中的serviceId，默认为：configserver
      # uri: http://localhost:7777 服务发现时不使用
      username: fuyi
      password: pwd
  application:
    name: simple

# 连接eureka server    
eureka:
  client:
    service-url:
      defaultZone: http://user:password@localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```


## 加解密
* 相对config server和 git repo之间，跟config client无关
* 非对称加密需生成一个keystore, 如server.jks,放到classpath

```
# 都是在confg server 的application.yml中配置

# 对称(symmetric)加密key配置,盐
encrypt:
  key: fuyi
  
# 非对称(asymmetric )加密配置
encrypt:
  keyStore:
    location: classpath:/server.jks
    password: letmein
    alias: mytestkey
    secret: changeme
```


```
# 创建一个key store
$ keytool -genkeypair -alias mytestkey -keyalg RSA \
  -dname "CN=Web Server,OU=Unit,O=Organization,L=City,S=State,C=US" \
  -keypass changeme -keystore server.jks -storepass letmein
```
* 使用步骤：
1. 通过 $ curl localhost:8888/encrypt -d mysecret加密mysecret名文获得密文
2. 加{cipher}前缀，放入git repo相应文件中
3. config server获取git repo解密后发送给config client，对client透明

* git中在*.properties中的需要加密的values不能被引号包裹，否则不能被解密
* git中在*.yml中要加单引号

```
*.yml中
spring:
  datasource:
    username: dbuser
    password: '{cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ'

*.properties中
spring.datasource.password={cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ
```


## 本地文件系统（classpath）


```
config server 的 application.yml

server:
  port: 7777
  
security:
  basic:
    enabled: true
  user:
    password: pwd
  
spring:
  application:
    name: config-server
  profiles:
    active: native # 激活config server使用本地文件系统作为后端存储启动
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/config # 设置local 配置文件存放 path

# 注册到eureka server     
eureka:
  client:
    service-url:
      defaultZone: http://user:password@localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```


```
config client 的bootstrap.yml

server:
  port: 7788
  
spring:
  cloud:
    config:
      username: fuyi
      password: pwd
      profile: test
      label: master
      discovery:
        enabled: true
        service-id: config-server
  application:
    name: foo
 
# 注册到eureka server    
eureka:
  client:
    service-url:
      defaultZone: http://user:password@localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

## 手动刷新
* add spring-boot-starter-actuator
* 在需要支持刷新重新加载的bean上，如包含 @Value("${profile}") 的Controller上加 @RefreshScope注解
* 在更改git repo配置提交后，访问config client
* $ curl -X POST Http://localhost:7788/refresh端点手动刷新，原理失效配置缓存
* 参考[Refresh Scope](http://cloud.spring.io/spring-cloud-static/Dalston.SR5/single/spring-cloud.html#_refresh_scope#_refresh_scope)

## 自动刷新（spring cloud bus）
* 在手动刷新配置的基础上
* add spring-cloud-starter-bus-amqp

```
config client的application.yml

spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: user
    password: secret
```
* $ curl -X POST Http://localhost:7788/bus/refresh
* 向一个config client发送刷新事件，同时传播给基于rabbitmq的spring cloud bus,bus再广播给all clients
* 实现自动刷新步骤为：将该bus刷新url配置给git repo的webhook回调
* 即当每次向git repo进行提交后使用webhook回调进行刷新

