---
title: Spring Boot-05-项目源码结构
date: 2018-05-07 20:45:26
tags: Spring Boot
---

### spring-boot

Spring boot 主要的库，提供了支持 Spring Boot 其他部分的功能，其中包括了：
1. 在SpringApplication类，提供静态便捷方法，可以很容易写一个独立的 Spring 应用程序。它唯一的工作就是创造并更新一个合适的 SpringApplicationContext
1. 带有可选容器的嵌入式 Web 应用程序（Tomcat，Jetty 或 Undertow）
1. 一流的外部配置支持
1. 便捷ApplicationContext初始化程序，包括对敏感日志记录默认值的支持

### spring-boot-autoconfigure

 Spring Boot 可以根据类路径的内容配置大部分常用应用程序。单个@EnableAutoConfiguration注释会触发 Spring上下文的自动配置。
 
 自动配置尝试推断用户可能需要哪些 bean。例如，如果 HSQLDB在类路径中，并且用户尚未配置任何数据库连接，则他们可能需要定义内存数据库。当用户开始定义他们自己的 bean 时，自动配置将永远远离。

### spring-boot-cli

Spring 命令行应用程序编译并运行 Groovy 源代码，使得可以编写少量代码就能运行应用程序。Spring CLI 也可以监视文件，当它们改变时自动重新编译并重新启动。

### spring-boot-dependencies

该模块里面没有源码，只有所有依赖和插件的版本号信息。

是spring-boot-parent的父项目

### spring-boot-devtools

使 Spring Boot 应用支持热部署，提高开发者的开发效率，无需手动重启 Spring Boot 应用。

### spring-boot-parent

该模块是其他项目的 parent，该模块的父模块是 spring-boot-dependencies。

### spring-boot-properties-migrator

在 Spring Boot 2.0 中，许多配置属性被重新命名/删除，开发人员需要更新application.properties/ application.yml相应的配置。为了帮助你解决这一问题，Spring Boot 发布了一个新spring-boot-properties-migrator模块。一旦作为该模块作为依赖被添加到你的项目中，它不仅会分析应用程序的环境，而且还会在启动时打印诊断信息，而且还会在运行时为您暂时迁移属性。在您的应用程序迁移期间，这个模块是必备的，完成迁移后，请确保从项目的依赖关系中删除此模块。

### spring-boot-starters

无代码，由很多方便的依赖pom集合组成，如果你需要使用某种技术，通过添加少量的jar就可以把相关的依赖加入到项目中去。


### spring-boot-actuator-autoconfigure
### spring-boot-actuator

### spring-boot-docs

### spring-boot-test-autoconfigure
### spring-boot-test

### spring-boot-tools包含以下模块
    - spring-boot-antlib
    - spring-boot-autoconfigure-processor
    - spring-boot-configuration-metadata
    - spring-boot-configuration-processor
    - spring-boot-gradle-plugin	Skip building
    - spring-boot-loader-tools
    - spring-boot-loader
    - spring-boot-maven-plugin
    - spring-boot-test-support
    

* spring-boot-loader 模块通过自定义 jar 包结构，自定义类加载器，优雅的实现了嵌套 jar 资源的加载，通过打包时候重新设置启动类和组织 jar 结构，通过运行时设置自定义加载器来实现嵌套 jar 资源加载。[原理参考传送门](http://fangjian0423.github.io/2017/05/31/springboot-executable-jar/)