---
title: Java常见规范
toc: true
date: 2021-04-19 09:36:03
tags:
categories:
---

## Common Annotations
JSR-250: http://jcp.org/en/jsr/detail?id=250

```
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```

- @PostConstruct: 见名知意，被该注解标注的方法在构造方法执行后才执行，一般用来做一些初始化工作，如果该类中有需要依赖注入的成员，则会在依赖注入完成后才执行该方法。它标注的方法签名有一些限制：不能有方法参数、void返回值、不能是static的等，在该注解注释中有说明。
- @PreDestroy：见名知意，被修饰的方法是一个回调通知方法，当该方法所属类的实例从容器中移除时调用的，该方法通常用来释放一些持有的资源。方法签名要求同@PostConstruct。

## bean validation
- 官网：https://beanvalidation.org
- JSR-308: Bean Validation 2.0      https://jcp.org/en/jsr/detail?id=380
- JSR-349: Bean Validation 1.1      https://jcp.org/en/jsr/detail?id=349
- JSR-303: Bean Validation 1.0      https://jcp.org/en/jsr/detail?id=303
- 推荐实现 Hibernate Validator       http://www.hibernate.org/subprojects/validator.html

Hibernate Validator提供了规范中所有内置 constraint 的实现，除此之外还有一些附加的 constraint。

一个 constraint就是一个约束，一个 constraint 通常由 annotation 和相应的 constraint validator 组成，它们是一对多的关系。也就是说一个 annotation 可以有多个 constraint validator 对应。在运行时，Bean Validation 框架本身会根据被注释元素的类型来选择合适的 constraint validator 对数据进行验证。

有些时候，在用户的应用中需要一些更复杂的 constraint。Bean Validation 提供扩展 constraint 的机制。可以通过两种方法去实现，一种是组合现有的 constraint 来生成一个更复杂的 constraint，另外一种是开发一个全新的 constraint。

PS: 在Spring项目中，在Controller的方法入参中只需要给被校验实体加 @Valid注解方可生效

## JAS-RS
JAX-RS是JAVA EE6 引入的一个新技术。 JAX-RS即Java API for RESTful Web Service，使用了Java SE5引入的Java注解来简化Web服务的客户端和服务端的开发和部署。

JAX-RS提供了一些注解将一个资源类，一个POJO Java类，封装为Web资源。包括：
- @Path，标注资源类或者方法的相对路径
- @GET，@PUT，@POST，@DELETE，标注方法是HTTP请求的类型。
- @Produces，标注返回的MIME媒体类型
- @Consumes，标注可接受请求的MIME媒体类型
- @PathParam，@QueryParam，@HeaderParam，@CookieParam，@MatrixParam，@FormParam,分别标注方法的参数来自于HTTP请求的不同位置，例如@PathParam来自于URL的路径，@QueryParam来自于URL的查询参数，@HeaderParam来自于HTTP请求的头信息，@CookieParam来自于HTTP请求的Cookie。

基于JAX-RS实现的框架有Jersey，RESTEasy等。这两个框架创建的应用可以很方便地部署到Servlet 容器中，比如Tomcat，JBoss等。值得一提的是RESTEasy是由JBoss公司开发的，所以将用RESTEasy框架实现的应用部署到JBoss服务器上，可以实现很多额外的功能。

## JPA
JPA全称为Java Persistence API ，Java持久化API是Sun公司在Java EE 5规范中提出的Java持久化接口。JPA吸取了目前Java持久化技术的优点，旨在规范、简化Java对象的持久化工作。它只是一套规范API接口，使用JPA持久化对象，并不是依赖于某一个具体框架实现，这保证了基于JPA开发的企业应用能够经过少量的修改就能够在不同的实现下运行。常见JPA实现如下：
- Hibernate
- MyBatis
- Spring Data JPA
- TopLink: Oracle公司的产品，作为一个遵循OTN协议的商业产品，TopLink 在开发过程中可以自由地下载和使用，但是一旦作为商业产品被使用，则需要收取费用。由于这一点，TopLink 的市场占有率不高。

目前JPA最新版本为2017年发布的JPA 2.2： https://jcp.org/en/jsr/detail?id=338

核心组件如下：
- EntityManagerFactory接口: 创建和管理多个EntityManager实例
- EntityManager接口：管理对象Entity的CRUD操作(create, update, delete, Query)
- Entity: 对应需要持久化的对象实体
- EntityTransaction接口: 对事务的处理接口，与EntityManager一对一
- Persistence类: 包含获取EntityManagerFactory实例的静态方法
- Query接口: 获取满足creteria的关系对象

## 参考资料
> - []()
> - []()
