---
title: Hystrix-11-使用Turbine监控集群
date: 2018-06-13 16:53:30
tags: Hystrix
---

### 架构图
![image](https://note.youdao.com/yws/api/personal/file/4274CBB07D0C4A3CA7EA5F01642E517F?method=download&shareKey=c513f70601574c07e44b46fa6e2dc2eb)

上图每个App都要暴露一个Url，用于将自己的依赖调用统计信息的输出。

单个App时，可直接将其Url输入Dashboard进行图形化分析。

采用App集群时，会将App的一套代码，在不同机器不同IP上起多个实例，调用时可能根据某种负载均衡策略调用，统计依赖调用信息时，应该将一个App集群内不同服务实例中相同的Command聚合到一个断路器上，进行统一监控。

### 聚合步骤
1. App暴露统计信息到单个Url

2. 搭建Turbine聚合服务器，聚合为一个Turbine-Url

3. 采用Dashboard解析Turbine-Url为图形监控界面

1,3前面已经介绍过，下面主要介绍搭建Turbine聚合服务器

### 搭建Turbine聚合服务器
##### 非SpringBoot项目
* 配置Turbine Servlet收集器
```
<servlet>  
   <description></description>  
   <display-name>TurbineStreamServlet</display-name>  
   <servlet-name>TurbineStreamServlet</servlet-name>  
   <servlet-class>com.netflix.turbine.streaming.servlet.TurbineStreamServlet</servlet-class>  
 </servlet>  
 <servlet-mapping>  
   <servlet-name>TurbineStreamServlet</servlet-name>  
   <url-pattern>/turbine.stream</url-pattern>  
 </servlet-mapping> 
```
* classpath下编写config.properties配置集群实例

单集群
```
# 单集群配置
turbine.aggregator.clusterConfig=mobile-online
# 配置mobile-online集群实例,多个实例ip，用逗号分割
turbine.ConfigPropertyBasedDiscovery.mobile-online.instances=127.0.0.1,192.168.1.184
# 配置mobile-online数据流servlet
turbine.instanceUrlSuffix.mobile-online=:1111/hystrix.stream
```
多集群
```
# 配置两个集群:mobile-online,web-online  
turbine.aggregator.clusterConfig=mobile-online,web-online 

# 配置mobile-online集群实例,多个实例ip，用逗号分割
turbine.ConfigPropertyBasedDiscovery.mobile-online.instances=127.0.0.1,192.168.1.184
# 配置mobile-online数据流servlet
turbine.instanceUrlSuffix.mobile-online=:1111/hystrix.stream

# 配置web-online集群实例  
turbine.ConfigPropertyBasedDiscovery.web-online.instances=127.0.0.1,192.168.1.184
turbine.instanceUrlSuffix.web-online=:1112/hystrix.stream
```

#### SpringBoot项目
* pom.xml

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.fuyi</groupId>
	<artifactId>hystrix-turbine</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.0.RELEASE</version>
		<relativePath />
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
		<!-- turbine依赖 -->
		<dependency>
		    <groupId>com.netflix.turbine</groupId>
		    <artifactId>turbine-core</artifactId>
		    <version>1.0.0</version>
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
* application.properties
```
server.port=3333
```
* config.properties 同非上文
* 启动类
```
package com.fuyi.hystrix.turbine;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;

import com.netflix.turbine.init.TurbineInit;
import com.netflix.turbine.streaming.servlet.TurbineStreamServlet;

@SpringBootApplication
public class Main {

	public static void main(String[] args) {
		SpringApplication.run(Main.class, args);
		TurbineInit.init();
	}
	
	@Bean
    public ServletRegistrationBean servletRegistrationBean() {
		// ServletName默认值为首字母小写，即myServlet
        return new ServletRegistrationBean(new TurbineStreamServlet(), "/turbine.stream");
    }
}

```
### 测试
1. 启动dashboard，访问 http://localhost:2222/hystrix

2. 在127.0.0.1和192.168.1.184分别启动整合了Hystrix的SpringMVC项目

3. 启动上文搭建的Turbine聚合服务器，其聚合后的统计数据输出Url为： http://localhost:3333/turbine.stream?cluster=mobile-online
4. 将3中的Url输入1中dashboard进行图形化分析监控