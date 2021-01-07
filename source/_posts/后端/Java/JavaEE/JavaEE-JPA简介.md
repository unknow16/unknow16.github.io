---
title: JavaEE-JPA简介
date: 2018-07-02 17:06:31
doc: true
tags: JavaEE
---

## 简介
JPA是Java的一个规范。 

由于JPA只是一个规范，它本身不执行任何操作。 它需要一个实现。 

常见JPA实现如下：
- Hibernate
- MyBatis
- Spring Data JPA
- TopLink 

TopLink：Oracle公司的产品，作为一个遵循OTN协议的商业产品，TopLink 在开发过程中可以自由地下载和使用，但是一旦作为商业产品被使用，则需要收取费用。由于这一点，TopLink 的市场占有率不高。

JPA全称为Java Persistence API ，Java持久化API是Sun公司在Java EE 5规范中提出的Java持久化接口。JPA吸取了目前Java持久化技术的优点，旨在规范、简化Java对象的持久化工作。使用JPA持久化对象，并不是依赖于某一个ORM框架。

JPA 是 JCP 组织发布的 Java EE 标准之一，因此任何声称符合 JPA 标准的框架都遵循同样的架构，提供相同的访问 API，这保证了基于JPA开发的企业应用能够经过少量的修改就能够在不同的JPA框架下运行。

## JPA版本

#### JPA 1.0
作为EJB 3.0规范的一部分，JPA的第一个版本JPA 1.0于2006年发布。

#### JPA 2.0 
此版本于2009年下半年随JavaEE 6.0发布(JSR 317)。以下是此版本的重要功能: 

- 它支持验证。
- 它扩展了对象关系映射的功能。
- 它共享缓存支持的对象。

#### JPA 2.1
JPA 2.1于2013年发布，具有以下特性: 

- 它允许提取对象。
- 它为条件更新/删除提供支持。
- 它生成模式。

#### JPA 2.2
JPA 2.2在2017年作为维护开发而发布。它的一些重要特性是: 

- 它支持Java 8的日期和时间。
- 它提供了@Repeatable注释，当想要将相同的注释应用到声明或类型用法时可以使用它。
- 它允许JPA注释在元注释中使用。
- 它提供了流式查询结果的功能。

## 核心组件

- EntityManagerFactory: 创建和管理多个EntityManager实例
- EntityManager: 接口，管理对象的操作(create, update, delete, Query)
- Entity: 持久化对象，在数据库中以record存储
- EntityTransaction: 与EntityManager一对一
- Persistence: 包含获取EntityManagerFactory实例的静态方法
- Query: 运营商必须实现的接口，获取满足creteria的关系对象（relational object）