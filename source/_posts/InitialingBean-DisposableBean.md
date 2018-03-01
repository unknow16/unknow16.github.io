---
title: InitialingBean-DisposableBean
date: 2018-03-01 15:16:40
tags: Spring
---

### 简介
- InitialingBean是一个接口，提供了一个唯一的方法afterPropertiesSet()。

- DisposableBean也是一个接口，提供了一个唯一的方法destory()。

这两个接口是一组的，功能类似，因此放在一起：前者顾名思义在Bean属性都设置完毕后调用afterPropertiesSet()方法做一些初始化的工作，后者在Bean生命周期结束前调用destory()方法做一些收尾工作。下面看一下例子，为了能明确地知道afterPropertiesSet()方法的调用时机，加上一个属性，给属性set方法，在set方法中打印一些内容：

### 自定义bean
```
package com.fuyi.test;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

public class Foobar implements InitializingBean, DisposableBean{
	
	@SuppressWarnings("unused")
	private String name;
	
	public void setName(String name) {
		this.name = name;
		System.out.println("进入setter方法中");
	}

	//该bean中的属性都set完后调用,即setter方法调用完执行
	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("进入afterPropertiesSet方法");
	}

	//在bean的生命周期结束前，调用该方法做一些收尾工作
	@Override
	public void destroy() throws Exception {
		System.out.println("进入destroy方法");
	}
	
	//业务方法
	public String sayHello() {
		return "I am " + name;
	}
}

```
### 配置

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd 
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

	<bean id="foobar" class="com.fuyi.test.Foobar">
		<property name="name" value="fuyi"></property>
	</bean>
</beans>
```
### 启动Spring容器测试
```
@Test
public void testInitialingBean(){
	AbstractApplicationContext ac = new ClassPathXmlApplicationContext("classpath:bean.xml");
	Foobar foobar = (Foobar) ac.getBean("foobar");

	//基于web的ApplicationContext会自动在web应用关闭时，自动关闭IOC容器
	//非web时，可以在JVM中注册一个“关闭钩子”，确保IOC容器被恰当关闭，单例bean持有的资源被释放，回调destory()方法
	ac.registerShutdownHook();
	ac.start();
	
	String sayHello = foobar.sayHello();
	System.out.println(sayHello);
}
	
//执行的结果为：
进入setter方法中
进入afterPropertiesSet方法
I am fuyi
进入destroy方法
```
### 关于这两个接口，我总结几点：

1. InitializingBean接口、Disposable接口可以和init-method、destory-method配合使用，接口执行顺序优先于配置

2. InitializingBean接口、Disposable接口底层使用类型强转.方法名()进行直接方法调用，init-method、destory-method底层使用反射，前者和Spring耦合程度更高但效率高，后者解除了和Spring之间的耦合但是效率低，使用哪个看个人喜好

3. afterPropertiesSet()方法是在Bean的属性设置之后才会进行调用，某个Bean的afterPropertiesSet()方法执行完毕才会执行下一个Bean的afterPropertiesSet()方法，因此不建议在afterPropertiesSet()方法中写处理时间太长的方法