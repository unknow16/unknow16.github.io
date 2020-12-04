---
title: Hystrix-10-SpringMVC整合Hystrix及监控
date: 2018-06-13 16:53:20
tags: Hystrix
---

### SpringMVC整合使用Hystrix隔离调用依赖，并实现监控
1. 在SpringMVC中需要隔离调用依赖的地方，用Hystrix的Command进行包装调用
2. Hystrix使用HystrixMetricsStreamServlet对当前JVM下所有Command调用情况进行统计并将数据流持续输出，并暴露一个URL。
> 统计数据输出的URL格式：http://hostname:port/application/hystrix.stream 
3. 将2中的URL放入HystrixDashboard监控平台中即可实现监控

## 非SpringBoot项目
##### hystrix的maven依赖配置:
```
<dependency>
	<groupId>com.netflix.hystrix</groupId>
	<artifactId>hystrix-core</artifactId>
	<version>1.3.16</version>
</dependency>
<dependency>
	<groupId>com.netflix.hystrix</groupId>
	<artifactId>hystrix-metrics-event-stream</artifactId>
	<version>1.1.2</version>
</dependency>
<dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-javanica</artifactId>
    <version>1.5.9</version>
</dependency>
```
##### web.xml配置servlet
```
<servlet>  
    <display-name>HystrixMetricsStreamServlet</display-name>  
    <servlet-name>HystrixMetricsStreamServlet</servlet-name>  
    <servlet-class>com.netflix.hystrix.contrib.metrics.eventstream.HystrixMetricsStreamServlet</servlet-class>  
</servlet>  
<servlet-mapping>  
    <servlet-name>HystrixMetricsStreamServlet</servlet-name>  
    <url-pattern>/hystrix.stream</url-pattern>  
</servlet-mapping>  
<!--  
    对应统计数据输出URL格式 :
    http://hostname:port/application/hystrix.stream 
--> 
```

## SpringBoot项目

#### pom.xml
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.fuyi</groupId>
	<artifactId>hystrix-demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.0.RELEASE</version>
		<relativePath/>
	</parent>

	<properties>
		<hystrix.version>1.3.16</hystrix.version>
		<hystrix-metrics-event-stream.version>1.1.2</hystrix-metrics-event-stream.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>com.netflix.hystrix</groupId>
			<artifactId>hystrix-core</artifactId>
			<version>${hystrix.version}</version>
		</dependency>
		<dependency>
			<groupId>com.netflix.hystrix</groupId>
			<artifactId>hystrix-metrics-event-stream</artifactId>
			<version>${hystrix-metrics-event-stream.version}</version>
		</dependency>
		<dependency>
		    <groupId>com.netflix.hystrix</groupId>
		    <artifactId>hystrix-javanica</artifactId>
		    <version>1.5.9</version>
		</dependency>

		
		
		<dependency>
	      <groupId>org.springframework.boot</groupId>
	      <artifactId>spring-boot-starter-web</artifactId>
	    </dependency>
	    <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-actuator</artifactId>
	    </dependency>
	</dependencies>

	<repositories>
		<!-- 仓库地址 -->
		<repository>
			<id>nexus</id>
			<name>local private nexus</name>
			<url>http://maven.oschina.net/content/groups/public/</url>
			<releases>
				<enabled>true</enabled>
			</releases>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>
	
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

#### 启动类

```
package com.fuyi.hystrix;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;

import com.netflix.hystrix.contrib.metrics.eventstream.HystrixMetricsStreamServlet;

@SpringBootApplication
public class Main  {

	public static void main(String[] args) {
		SpringApplication.run(Main.class, args);
	}
	
	// SpringBoot加载Servlet有很多种方法，不一定非像此处
	@Bean
    public ServletRegistrationBean servletRegistrationBean() {
		// ServletName默认值为首字母小写，即myServlet
        return new ServletRegistrationBean(new HystrixMetricsStreamServlet(), "/hystrix.stream");
    }
}

```

#### application.properties

```
server.port=1111
```

#### 定义控制器

```
package com.fuyi.hystrix.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import com.fuyi.hystrix.command.HelloCommand;

@Controller
public class HelloController {

	@RequestMapping("/hello")
	@ResponseBody
	public Object sayHello() {
	    // 此处HelloCommand为前文示例中的
		return new HelloCommand("fuyi").execute();
	}
}

```
## 测试
1. 访问统计数据输出Url

```
http://localhost:1111/hystrix.stream
```

2. 访问控制器

```
http://localhost:1111/hello
```

3. 可以看到1中有统计数据输出，可将1中Url放入HystrixDashBoard解析统计数据，以图形界面展示。