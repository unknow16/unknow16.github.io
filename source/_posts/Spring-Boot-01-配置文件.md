---
title: Spring Boot-01-配置文件
date: 2018-01-24 18:31:00
tags: Spring Boot
---

### 自定义属性
* 在*.yml或*.properties中自定义需要的属性键和值

### 类中注入自定义属性
* 使用@Value注解

```
@Value("${home.city}")
private String city;
```

* application.yml中的属性键值都会注入进Environment

```
@Autowired
Environment environment;

代码中
environment.getProperty("home.city")
```

* 如果带公共前缀的可以在类上注解@ConfigurationProperties(prefix = "home")，会自动绑定属性到类中同名属性

```
1. application.yml：

home:
  province: henan
  city: nanyang
  name: 付一

2. 将一系类属性合并注入实体中：

@Component
@ConfigurationProperties(prefix = "home")
public class HomeProperties {

    private String province;
    private String city;
    private String name;
    
    相应的getter/setter方法不能省
｝  

3. 在需要的类中，自动注入配置实体载体，调用getter获取
```

* 引入其他properties中的属性进Environment

```
@Configuration
@PropertySource(value = {"classpath:fuyi.properties"})
public class PropertyConfig {
}
```



### random.*属性
* ${random.long}：随机long值，3882765395915877886
* ${random.int}：随机int值，1932153008
* ${random.int(1,200)}：随机值范围1-200，
* ${random.uuid}：随机uuid，4a423ccb-f31a-4ad1-b11a-2ca52b73999b
* ${random.value}：随机值，028929d408f09b2e862cc0b8c538a375

### 公共应用属性
* [spring boot 提供的所有可配置属性](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)
* [spring boot 各版本文档](https://docs.spring.io/spring-boot/docs/)

### 版本
* Release ： 表示 是正式的版本
* RC ：stands for Release Candidate 表示 侯选版本 
* M ： stands for milestone 表示里程碑版本
* 一般而言, 稳定性由上而下, 依次降低.