---
title: Spring Boot-07-自定义一个Starter
date: 2018-06-28 18:42:26
tags: Spring Boot
---

## 前言
众所周知，Spring Boot由众多Starter组成，通常需要某个功能时，直接引入相应Starter，Spring Boot会自动进行相关配置，并将相应对象注入Spring IOC容器中，使用时直接用@Autowired引入即可，对于必须要配置的属性，如连接数据库的用户名和密码等，Spring Boot提供了约定的属性名，我们直接在application.properties或application.yml中配置即可。

Spring Boot Starter有什么神奇的呢？

带着好奇心下面我们来写一个自定义的Starter来一探究竟。

#### Starter命名问题
这里说下artifactId的命名问题，Spring 官方 Starter通常命名为spring-boot-starter-{name}如 spring-boot-starter-web， Spring官方建议非官方Starter命名应遵循{name}-spring-boot-starter的格式。

#### Spring 官方建议Starter格式
一个库的完整Spring Boot starter可能包含以下组件：
1. autoconfigure模块：包含自动配置代码
2. starter模块：整合autoconfigure模块和其他依赖库的依赖，总之，添加这个starter依赖就能使用这个库。通常是一个空jar。另外我们自己的Starter必须要直接或间接的引用官方的spring-boot-starter依赖。

可参考[dubbo-spring-boot-starter](https://github.com/apache/incubator-dubbo-spring-boot-project)

## 实现
这里讲一下我们的Starter要实现的功能，很简单，提供一个Service，包含一个能够将字符串加上前后缀的方法String wrap(String word)。

我这里并没有按照Spring官方的建议格式。

## 项目结构

```
springboot-demo  // 聚合工程
    - autoconfig-test  // 引入自定义的Starter的进行测试
    - autoconfig-test-starter  // 自定义的Starter
    - pom.xml 
```
#### springboot-demo的pom.xml

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.fuyi</groupId>
	<artifactId>springboot-demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>

	<modules>
		<module>autoconfig-test-starter</module>
		<module>autoconfig-test</module>
	</modules>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.14.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>
</project>
```
## autoconfig-test-starter项目
#### pom.xml
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<artifactId>autoconfig-test-starter</artifactId>

	<parent>
		<groupId>com.fuyi</groupId>
		<artifactId>springboot-demo</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-autoconfigure</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
	</dependencies>
</project>
```
#### 1. 定义属性配置类

```
package com.fuyi.starter;

import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * 这个类定义了默认的属性值，如该类中，有两个属性prefix和suffix默认值分别为“www.”和“.com”.
 * <p>
 * {@ConfigurationProperties} 注解会定义一个前缀匹配，如果想修改属性值，<br>
 * 可以在application.properties中使用“前缀.属性=修改的值”进行修改。<br>
 * 
 * @author fuyi
 *
 */
@ConfigurationProperties(prefix = "fuyi.service")
public class FuyiProperties {

	private String prefix = "www.";
    private String suffix = ".com";
    
	public String getPrefix() {
		return prefix;
	}
	public void setPrefix(String prefix) {
		this.prefix = prefix;
	}
	public String getSuffix() {
		return suffix;
	}
	public void setSuffix(String suffix) {
		this.suffix = suffix;
	}

}

```

#### 2. 定义服务类

```
package com.fuyi.starter;

/**
 * 服务类是指主要的功能类，如果没有SpringBoot，这些服务类在Spring中都是需要自己去配置生成的。
 * <p>
 * 如Mybatis中的DataSource。
 * <p>
 * 这里讲一下我们的Starter要实现的功能，很简单，提供一个Service,<br>
 * 包含一个能够将字符串加上前后缀的方法String wrap(String word)。
 * 
 * @author fuyi
 *
 */
public class FuyiService {
	
	private String prefix;
    private String suffix;

    public FuyiService(String prefix, String suffix) {
        this.prefix = prefix;
        this.suffix = suffix;
    }

    public String wrap(String word) {
        return prefix + word + suffix;
    }
}

```

#### 3. 定义自动配置类

```
package com.fuyi.starter;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 自动配置类主要作用是SpringBoot的配置核心，它会写在MEAT-INF/spring.factories中<br>
 * 告知SpringBoot在启动时去读取该类并根据该类的规则进行配置。
 * 
 * @author fuyi
 *
 */
@Configuration
@EnableConfigurationProperties(FuyiProperties.class) <4>
@ConditionalOnClass(FuyiService.class) <1>
@ConditionalOnProperty(prefix = "fuyi.service" , value = "enabled" , matchIfMissing = true)<2>
public class FuyiServiceAutoConfiguration {

	@Autowired
	private FuyiProperties fuyiProperties;
	
	@Bean
	@ConditionalOnMissingBean<3>
	public FuyiService autoConfig() {
		FuyiService fuyiService = new FuyiService(fuyiProperties.getPrefix(), fuyiProperties.getSuffix());
		return fuyiService;
	}
}

```
简单说下涉及的条件注解，后开专篇详解：
1. @ConditionalOnClass(FuyiService.class)：当classpath下存在FuyiService类时，才执行自动配置类FuyiServiceAutoConfiguration。
2. @ConditionalOnProperty(prefix = "fuyi.service" , value = "enabled" , matchIfMissing = true)：检查指定的属性有指定的值，此处配置为，如果没配置该属性默认true.
3. @ConditionalOnMissingBean：当Spring IOC容器中不存在FuyiService实例时，才执行注解的方法。该方法则是生成一个FuyiService实例放入容器。
4. @EnableConfigurationProperties注解根据FuyiProperties类开启属性注入，允许在application.properties修改里面的属性值的默认值。

#### 4. 在META-INF/spring.factories中指定自动配置类
```
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.fuyi.starter.FuyiServiceAutoConfiguration
```

好啦，搞定！下面可以使用maven install命令把starter存到本地，其他SpringBoot项目需要使用这个starter，直接导入就可以啦，然后可直接用@Autowired注入功能类FuyiService使用。

## autoconfig-test项目
#### pom.xml
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<artifactId>autoconfig-test</artifactId>

	<parent>
		<groupId>com.fuyi</groupId>
		<artifactId>springboot-demo</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<dependencies>
	    <!-- 引入自定义的Starter -->
		<dependency>
			<groupId>com.fuyi</groupId>
			<artifactId>autoconfig-test-starter</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

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
Note: 引入spring-boot-starter-web的原因是：因为spring-boot-starter-test虽然会引入Spring Boot相关核心类， 但scope是test，只会在测试生命周期里存在，在运行时找不到Spring Boot相关核心类，引入spring-boot-starter-web用来引入运行时能存在的Spring Boot相关核心类。

#### 1. 启动类

```
package com.fuyi.boot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BootApplication {

	public static void main(String[] args) {
		SpringApplication.run(BootApplication.class, args);
	}

}
```

#### 2. 测试类

```
package com.fuyi.test;

import static org.assertj.core.api.Assertions.*;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.fuyi.boot.BootApplication;
import com.fuyi.starter.FuyiService;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = {BootApplication.class})
public class AppTest {

	@Autowired
	private FuyiService fuyiService;
	
	@Test
	public void test1() {
		assertThat(fuyiService.wrap("fuyi")).isEqualTo("www.fuyi.com");
	}
}
```

测试通过！

#### 3. 测试修改默认配置
在src/test/resources下application.properties文件中新增下面内容：

```
fuyi.service.prefix=ftp.
fuyi.service.suffix=.org
```

再执行原来的测试报错如下：
```
org.junit.ComparisonFailure: expected:<"[www.fuyi.com]"> but was:<"[ftp.fuyi.org]">
```
很明显默认值已经被修改了。

## 总结下Starter的工作原理

1. Spring Boot在启动时扫描项目所依赖的JAR包，寻找包含spring.factories文件的JAR包
1. 根据spring.factories配置加载AutoConfigure类
1. 根据 @Conditional注解的条件，进行自动配置并将Bean注入Spring Context