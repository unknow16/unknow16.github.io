---
title: Spring Boot-01-配置文件
date: 2018-01-24 18:31:00
tags: Spring Boot
---

### 自定义属性
* 在*.yml或*.properties中自定义需要的属性键和值


### 参数间的引用
在application.properties中的各个参数之间也可以直接引用来使用，就像下面的设置：
```
home:
  province: henan
  city: nanyang
  name: 付一
  address: ${home.province}省${home.city}市
```


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


### 多环境配置
我们在开发Spring Boot应用时，通常同一套程序会被应用和安装到几个不同的环境，比如：开发、测试、生产等。其中每个环境的数据库地址、服务器端口等等配置都会不同，如果在为不同环境打包时都要频繁修改配置文件的话，那必将是个非常繁琐且容易发生错误的事。

对于多环境的配置，各种项目构建工具或是框架的基本思路是一致的，通过配置多份不同环境的配置文件，再通过打包命令指定需要打包的内容之后进行区分打包，Spring Boot也不例外，或者说更加简单。

在Spring Boot中多环境配置文件名需要满足application-{profile}.properties的格式，其中{profile}对应你的环境标识，比如：

- application-dev.properties：开发环境
- application-test.properties：测试环境
- application-prod.properties：生产环境

至于哪个具体的配置文件会被加载，需要在application.properties文件中通过spring.profiles.active属性来设置，其值对应{profile}值。

如：spring.profiles.active=test就会加载application-test.properties配置文件内容，如果在application-test.properties中加载不到某个属性，会再到application.properties中寻找加载


### 配置文件的生效顺序，会对值进行覆盖：
1. @TestPropertySource 注解
2. 命令行参数
3. Java系统属性（System.getProperties()）
4. 操作系统环境变量
5. 只有在random.*里包含的属性会产生一个RandomValuePropertySource
6. 在打包的jar外的应用程序配置文件（application.properties，包含YAML和profile变量）
7. 在打包的jar内的应用程序配置文件（application.properties，包含YAML和profile变量）
8. 在@Configuration类上的@PropertySource注解
9. 默认属性（使用SpringApplication.setDefaultProperties指定）

### jar包内配置文件加载顺序
* SpringApplication默认会加载application.properties/application.yml,按照下列表顺序
1. 当前目录的 /config子目录下
2. 当前目录
3. classpath下/config包中
4. classpath下

* 更改配置文件名和路径
```
spring.config.name:   配置文件名
spring.config.location:   以逗号分割的列表

```
由于两者需早期指定，所以只能在OS env, system property, commond line property
```
$ java -jar myproject.jar --spring.config.name=myproject
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
```

### 禁用命令行参数
默认SpringApplication会将命令行参数（--server.port=9000）添加到Environment中
如果不想命令行参数被添加到Environment中
SpringApplication.setAddCommandLineProperties(false).

### 公共应用属性
* [spring boot 提供的所有可配置属性](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)
* [spring boot 各版本文档](https://docs.spring.io/spring-boot/docs/)

### 版本
* Release ： 表示 是正式的版本
* RC ：stands for Release Candidate 表示 侯选版本 
* M ： stands for milestone 表示里程碑版本
* 一般而言, 稳定性由上而下, 依次降低.