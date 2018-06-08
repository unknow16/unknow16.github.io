---
title: Spring-06-BeanNameAware-ApplicationContextAware-BeanFactoryAware
date: 2018-03-01 15:26:42
tags: Spring
---

### 简介
这三个接口放在一起写，是因为它们是一组的，作用相似。

"Aware"的意思是"感知到的"，那么这三个接口的意思也不难理解：

1. 实现BeanNameAware接口的Bean，在Bean加载的过程中可以获取到该Bean的id

2. 实现ApplicationContextAware接口的Bean，在Bean加载的过程中可以获取到Spring的ApplicationContext，这个尤其重要，ApplicationContext是Spring应用上下文，从ApplicationContext中可以获取包括任意的Bean在内的大量Spring容器内容和信息

3. 实现BeanFactoryAware接口的Bean，在Bean加载的过程中可以获取到加载该Bean的BeanFactory

### 自定义bean
```
package com.fuyi.test.aware;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.BeanFactoryAware;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class AwareBean implements BeanNameAware, BeanFactoryAware, ApplicationContextAware, InitializingBean, DisposableBean {

	private String beanName;
	private BeanFactory beanFactory;
	private ApplicationContext applicationContext;
	
	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		this.applicationContext = applicationContext;
	}

	@Override
	public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
		this.beanFactory = beanFactory;
	}

	@Override
	public void setBeanName(String beanName) {
		this.beanName = beanName;
	}

    //使用注入的相应类，处理业务
	public void sayHello() {
		System.out.println("beanName = " + this.beanName);
		System.out.println("beanFactory = " + this.beanFactory);
		System.out.println("applicationContext = " + this.applicationContext);
	}

	@Override
	public void destroy() throws Exception {
		System.out.println("destroy");
	}

	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("afterPropertiesSet");
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

<!-- 	<bean id="foobar" class="com.fuyi.test.lifecycle.Foobar">
		<property name="name" value="fuyi"></property>
	</bean> -->
	
	<bean id="awareBean" class="com.fuyi.test.aware.AwareBean"></bean>
</beans>
```

### 测试

```
package com.fuyi.test;

import org.junit.Before;
import org.junit.Test;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.fuyi.test.aware.AwareBean;
import com.fuyi.test.lifecycle.Foobar;

public class BeanTest {
	
	AbstractApplicationContext ac;
	
	@Before
	public void init() {
		ac = new ClassPathXmlApplicationContext("classpath:bean.xml");
		
		//基于web的ApplicationContext会自动在web应用关闭时，自动关闭IOC容器
		//非web时，可以在JVM中注册一个“关闭钩子”，确保IOC容器被恰当关闭，单例bean持有的资源被释放，回调destory()方法
		ac.registerShutdownHook();
		ac.start();
	}
	
	@Test
	public void testAware() {
		AwareBean bean = ac.getBean(AwareBean.class);
		bean.sayHello();
	}
}

```
### 结果

可见afterPropertiesSet先于Aware执行。
```
afterPropertiesSet
beanName = awareBean
beanFactory = org.springframework.beans.factory.support.DefaultListableBeanFactory@6a4f787b: defining beans [awareBean]; root of factory hierarchy
applicationContext = org.springframework.context.support.ClassPathXmlApplicationContext@4c3e4790: startup date [Thu Mar 01 11:30:24 CST 2018]; root of context hierarchy
destroy
```


