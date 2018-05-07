---
title: Spring Boot-04-启动过程解析
date: 2018-05-07 20:45:10
tags: Spring Boot
---

### Spring Boot应用程序一般启动代码如下：
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```
### 1. @SpringBootApplication
[zhisheng-Spring Boot 2.0中SpringBootApplication注解详解](http://www.54tianzhisheng.cn/2018/04/19/SpringBootApplication-annotation/)

### 2. SpringApplication.run(Application.class, args);
[zhisheng-SpringApplication 深入探索](http://www.54tianzhisheng.cn/2018/04/30/springboot_SpringApplication/)

[Format-SpringBoot源码分析之SpringBoot的启动过程](http://fangjian0423.github.io/2017/04/30/springboot-startup-analysis/)