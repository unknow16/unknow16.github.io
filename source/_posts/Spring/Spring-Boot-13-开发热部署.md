---
title: Spring Boot-13-开发热部署
date: 2018-07-09 20:12:31
tags: Spring Boot
---

Spring Boot 支持页面与类文件的热部署。

#### spring-boot-devtools 实现热部署
spring-boot-devtools 最重要的功能就是热部署。它会监听 classpath 下的文件变动，并且会立即重启应用。
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```
其中，optional=true 表示依赖不会传递，换句话说，其他依赖该项目的项目，如果想要使用 devtools，需要重新引入。

如果，希望指定文件夹下的文件改变的时候，重新启动 Spring Boot，我们只要在 src/main/resources/application.properties 中配置信息。


```
spring.devtools.restart.additional-paths= # Additional paths to watch for changes.
```
更多spring.devtools.*属性配置可参考spring boot文档附录

#### Spring Loaded 实现热部署
Spring Loaded 也可以实现修改类文件的热部署。


```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
            <version>1.2.6.RELEASE</version>
        </dependency>
    </dependencies>
</plugin>
```

使用 mvn spring-boot:run 启动项目。

#### 模板文件热部署
在 Spring Boot，模板引擎的页面默认是开启缓存，如果修改页面内容，刷新页面是无法获取修改后的页面内容，所以，如果我们不需要模板引擎的缓存，可以进行关闭。


```
spring.freemarker.cache=false
spring.thymeleaf.cache=false
spring.velocity.cache=false
```

