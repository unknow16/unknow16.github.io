---
title: Java-注解@PostConstruct和@PreDestroy
date: 2018-07-05 16:26:28
tags: Java
---

@PostConstruct和@PreDestroy注解都是在jdk的rt.jar的javax.annotation包中。

## @PostConstruct
其源码如下：

```
package javax.annotation;

import java.lang.annotation.*;
import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.RetentionPolicy.*;

@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PostConstruct {
}
```

- 由@Target知，该注解限定被用在方法上

#### 执行时机
- 见名知意，被该注解标注的方法在构造方法执行后才执行，一般用来做一些初始化工作
- 如果该类中有需要依赖注入的成员，则会在依赖注入完成后才执行该方法。

#### 被修饰方法签名要求
- 不能有方法参数
- 返回为void
- 不能是static的
- 可以是public, protected, package private or private
- 暂未考虑用于interceptors规范的情况，具体可参考其javadoc注释

## @PreDestroy
其源码如下：

```
package javax.annotation;

import java.lang.annotation.*;
import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.RetentionPolicy.*;

@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PreDestroy {
}
```
- 由@Target知，该注解限定被用在方法上

#### 执行时机
- 被修饰的方法是一个回调通知方法，当该方法所属类的实例从容器中移除时调用的。
- 该方法通常用来释放一些持有的资源。

#### 被修饰方法签名要求
同@PostConstruct一样。