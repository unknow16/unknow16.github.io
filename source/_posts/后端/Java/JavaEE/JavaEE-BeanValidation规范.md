---
title: JavaEE-BeanValidation 规范
toc: true
date: 2020-04-11 22:27:11
tags:
categories:
---


官网：https://beanvalidation.org

- JSR-308: Bean Validation 2.0 	<https://jcp.org/en/jsr/detail?id=380>
- JSR-349: Bean Validation 1.1         https://jcp.org/en/jsr/detail?id=349
- JSR-303: Bean Validation 1.0         https://jcp.org/en/jsr/detail?id=303

参考实现 Hibernate Validator：http://www.hibernate.org/subprojects/validator.html

Hibernate Validator 提供了规范中所有内置 constraint 的实现，除此之外还有一些附加的 constraint。

一个 constraint就是一个约束，一个 constraint 通常由 annotation 和相应的 constraint validator 组成，它们是一对多的关系。也就是说一个 annotation 可以有多个 constraint validator 对应。在运行时，Bean Validation 框架本身会根据被注释元素的类型来选择合适的 constraint validator 对数据进行验证。

有些时候，在用户的应用中需要一些更复杂的 constraint。Bean Validation 提供扩展 constraint 的机制。可以通过两种方法去实现，一种是组合现有的 constraint 来生成一个更复杂的 constraint，另外一种是开发一个全新的 constraint。

## Bean Validation 中内置的 constraint

| **Constraint**                | **详细信息**                                             |
| :---------------------------- | :------------------------------------------------------- |
| `@Null`                       | 被注释的元素必须为 `null`                                |
| `@NotNull`                    | 被注释的元素必须不为 `null`                              |
| `@AssertTrue`                 | 被注释的元素必须为 `true`                                |
| `@AssertFalse`                | 被注释的元素必须为 `false`                               |
| `@Min(value)`                 | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| `@Max(value)`                 | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| `@DecimalMin(value)`          | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| `@DecimalMax(value)`          | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| `@Size(max, min)`             | 被注释的元素的大小必须在指定的范围内                     |
| `@Digits (integer, fraction)` | 被注释的元素必须是一个数字，其值必须在可接受的范围内     |
| `@Past`                       | 被注释的元素必须是一个过去的日期                         |
| `@Future`                     | 被注释的元素必须是一个将来的日期                         |
| `@Pattern(value)`             | 被注释的元素必须符合指定的正则表达式                     |

## Hibernate Validator 附加的 constraint

| **Constraint** | **详细信息**                           |
| :------------- | :------------------------------------- |
| `@Email`       | 被注释的元素必须是电子邮箱地址         |
| `@Length`      | 被注释的字符串的大小必须在指定的范围内 |
| `@NotEmpty`    | 被注释的字符串的必须非空               |
| `@Range`       | 被注释的元素必须在合适的范围内         |

## 参考资料
> - []()
> - []()
