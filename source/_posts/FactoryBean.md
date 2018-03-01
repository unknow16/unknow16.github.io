---
title: FactoryBean
date: 2018-03-01 15:27:33
tags: Spring
---

### 简介
传统的Spring容器加载一个Bean的整个过程，都是由Spring控制的，换句话说，开发者除了设置Bean相关属性之外，是没有太多的自主权的。FactoryBean改变了这一点，开发者可以个性化地定制自己想要实例化出来的Bean，方法就是实现FactoryBean接口。

### FactoryBean接口
```
package org.springframework.beans.factory;

public interface FactoryBean<T> {
    //返回生成的实例对象
    T getObject() throws Exception;
    //返回生成bean实例的class类型
    Class<?> getObjectType();
    //返回是否为单例
    boolean isSingleton();
}

```
### 自定义bean

```
//Animal接口
public interface Animal {
	public String sayHello();
}

//cat实现
public class Cat implements Animal {

	@Override
	public String sayHello() {
		return "Hello cat";
	}
}

//tiger实现
public class Tiger implements Animal {

	@Override
	public String sayHello() {
		return "hello tiger";
	}
}

//生产animal的工厂bean
public class AnimalFactoryBean implements FactoryBean<Animal> {
    
    //注入类型
	private String name;
	public void setName(String name) {
		this.name = name;
	}
	
	//根据类型生产相应实例
	@Override
	public Animal getObject() throws Exception {
		if (name.equals("tiger")) {
			return new Tiger();
		} else if(name.equals("cat")) {
			return new Cat();
		}
		return null;
	}

	@Override
	public Class<?> getObjectType() {
		return Animal.class;
	}

	@Override
	public boolean isSingleton() {
		return true;
	}
}

```
### 配置

返回的实例不是AnimalFactoryBean的实例，而是AnimalFactoryBean中的getObject()方法返回的对象
```
<bean id="animalFactoryBean" class="com.fuyi.test.factorybean.AnimalFactoryBean">
	<property name="name" value="cat"/>
</bean>
```
### 测试

```
package com.fuyi.test;

import org.junit.Before;
import org.junit.Test;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.fuyi.test.aware.AwareBean;
import com.fuyi.test.factorybean.Animal;
import com.fuyi.test.factorybean.AnimalFactoryBean;
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
	public void testFactoryBean() {
		Animal bean = ac.getBean(Animal.class);
		String sayHello = bean.sayHello();
		System.out.println(sayHello);
	}
}

//执行结果：Hello cat
```
