---
title: Spring Boot-18-AOP编程
date: 2018-08-30 18:16:48
tags: Spring Boot
---

> 也可称为面向切面编程，是一种编程范式，提供从另一个角度来考虑程序结构从而完善面向对象编程(OOP)。

## AOP的基本概念
在进行AOP开发前，先熟悉几个概念：

#### 连接点（Jointpoint）：
表示需要在程序中插入横切关注点的扩展点，连接点可能是类初始化、方法执行、方法调用、字段调用或处理异常等等，Spring只支持方法执行连接点，在AOP中表示为“在哪里干”；

#### 切入点（Pointcut）：
选择一组相关连接点的模式，即可以认为连接点的集合，Spring支持perl5正则表达式和AspectJ切入点模式，Spring默认使用AspectJ语法，在AOP中表示为“在哪里干的集合”；

#### 通知（Advice）：
在连接点上执行的行为，通知提供了在AOP中需要在切入点所选择的连接点处进行扩展现有行为的手段；包括前置通知（before advice）、后置通知(after advice)、环绕通知（around advice），在Spring中通过代理模式实现AOP，并通过拦截器模式以环绕连接点的拦截器链织入通知；在AOP中表示为“干什么”；

#### 方面/切面（Aspect）：
横切关注点的模块化，比如上边提到的日志组件。可以认为是通知、引入和切入点的组合；在Spring中可以使用Schema和@AspectJ方式进行组织实现；在AOP中表示为“在哪干和干什么集合”；

#### 引入（inter-type declaration）：
也称为内部类型声明，为已有的类添加额外新的字段或方法，Spring允许引入新的接口（必须对应一个实现）到所有被代理对象（目标对象）, 在AOP中表示为“干什么（引入什么）”；

#### 目标对象（Target Object）：
需要被织入横切关注点的对象，即该对象是切入点选择的对象，需要被通知的对象，从而也可称为“被通知对象”；由于Spring AOP 通过代理模式实现，从而这个对象永远是被代理对象，在AOP中表示为“对谁干”；

#### AOP代理（AOP Proxy）：
AOP框架使用代理模式创建的对象，从而实现在连接点处插入通知（即应用切面），就是通过代理来对目标对象应用切面。在Spring中，AOP代理可以用JDK动态代理或CGLIB代理实现，而通过拦截器模型应用切面。

#### 织入（Weaving）：
织入是一个过程，是将切面应用到目标对象从而创建出AOP代理对象的过程，织入可以在编译期、类装载期、运行期进行。

#### 总结
在AOP中，通过切入点选择目标对象的连接点，然后在目标对象的相应连接点处织入通知，而切入点和通知就是切面（横切关注点），而在目标对象连接点处应用切面的实现方式是通过AOP代理对象

## Spring通知类型

#### 前置通知（Before Advice）:
在切入点选择的连接点处的方法之前执行的通知，该通知不影响正常程序执行流程（除非该通知抛出异常，该异常将中断当前方法链的执行而返回）。

#### 后置通知（After Advice）:
在切入点选择的连接点处的方法之后执行的通知，包括如下类型的后置通知：


- 后置返回通知（After returning 
Advice）:在切入点选择的连接点处的方法正常执行完毕时执行的通知，必须是连接点处的方法没抛出任何异常正常返回时才调用后置通知。

- 后置异常通知（After throwing Advice）: 在切入点选择的连接点处的方法抛出异常返回时执行的通知，必须是连接点处的方法抛出任何异常返回时才调用异常通知。

- 后置最终通知（After finally Advice）: 在切入点选择的连接点处的方法返回时执行的通知，不管抛没抛出异常都执行，类似于Java中的finally块。

#### 环绕通知（Around Advices）：

环绕着在切入点选择的连接点处的方法所执行的通知，环绕通知可以在方法调用之前和之后自定义任何行为，并且可以决定是否执行连接点处的方法、替换返回值、抛出异常等等。

## 整合Spring Boot
#### 1. 引入依赖
```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
```

#### 2. 切入有某注解的方法示例

自定义注解如下：
```
package com.fuyi.aop.anno;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME) // 此处只有为RUNTIME时，才能在运行时获取参数值
public @interface TestAnnotation {
	
	String value() default "";

	String name();
}

```
定义切面如下：

```
package com.fuyi.aop.aspect;

import java.lang.reflect.Method;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import com.fuyi.aop.anno.TestAnnotation;

@Aspect 
@Order(-99)  // 控制多个aspect的执行，越小越先执行
@Component("testAspect")
public class TestAspect {
	
	@Pointcut("@annotation(com.fuyi.aop.anno.TestAnnotation)")
	public void testAnnoPointJoin() {}
	
	
	@Before(value="@annotation(com.fuyi.aop.anno.TestAnnotation)")
	public void beforeTest(JoinPoint joinPoint) throws Throwable {
		MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
		Method method = methodSignature.getMethod();
		
		TestAnnotation testAnnotation = method.getAnnotation(TestAnnotation.class);
		
		System.out.println(" beforeTest: name = " + testAnnotation.name() + ", value = " + testAnnotation.value());
	}
	
	@After("@annotation(com.fuyi.aop.anno.TestAnnotation)")
	public void afterTest(JoinPoint joinPoint) {
		MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
		Method method = methodSignature.getMethod();
		
		TestAnnotation testAnnotation = method.getAnnotation(TestAnnotation.class);
		
		System.out.println(" afterTest: name = " + testAnnotation.name() + ", value = " + testAnnotation.value());
	}
	
	
	@Around("testAnnoPointJoin()") 
	public Object aroundTest(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
		MethodSignature methodSignature = (MethodSignature) proceedingJoinPoint.getSignature();
		Method method = methodSignature.getMethod();
		
		TestAnnotation testAnnotation = method.getAnnotation(TestAnnotation.class);
		
		System.out.println(" aroundTestBefore: name = " + testAnnotation.name() + ", value = " + testAnnotation.value());
		Object result = proceedingJoinPoint.proceed();
		System.out.println(" aroundTestAfter: name = " + testAnnotation.name() + ", value = " + testAnnotation.value());
		
		return result + "around";
	}
}

```
#### 3. 总结
- 在application.properties中也不需要添加spring.aop.auto=true，因为这个默认就是true，值为true就是启用@EnableAspectJAutoProxy注解了。 
- 你不需要手工添加 @EnableAspectJAutoProxy 注解。 
- 当你需要使用CGLIB来实现AOP的时候，需要配置spring.aop.proxy-target-class=true，这个默认值是false，不然默认使用的是标准Java的实现。
- 其实aspectj的拦截器会被解析成AOP中的advice，最终被适配成MethodInterceptor，这些都是Spring自动完成的，如果你有兴趣，详细的过程请参考springAOP的实现。

## MethodInterceptor与HandlerInterceptor区别
- MethodInterceptor是AOP项目中的拦截器，它拦截的目标是方法。
- HandlerInterceptor属于Spring MVC项目提供的，用来拦截请求，在MethodInterceptor之前执行

HandlerInterceptoer拦截的是请求地址，所以针对请求地址做一些验证、预处理等操作比较合适。当你需要统计请求的响应时间时MethodInterceptor将不太容易做到，因为它可能跨越很多方法或者只涉及到已经定义好的方法中一部分代码。
