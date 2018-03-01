---
title: InstantiationAwareBeanPostProcessor
date: 2018-03-01 15:29:32
tags: Spring
---

### 简介
InstantiationAwareBeanPostProcessor又代表了Spring的另外一段生命周期：实例化。先区别一下Spring Bean的实例化和初始化两个阶段的主要作用：

1. 实例化----实例化的过程是一个创建Bean的过程，即调用Bean的构造函数，单例的Bean放入单例池中

2. 初始化----初始化的过程是一个赋值的过程，即调用Bean的setter，设置Bean的属性

之前的BeanPostProcessor作用于过程（2）前后，现在的InstantiationAwareBeanPostProcessor则作用于过程（1）前后

### InstantiationAwareBeanPostProcessor接口

注意该接口继承了BeanPostProcessor接口
```
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {

	Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException;

	boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException;

	PropertyValues postProcessPropertyValues(
			PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException;

}

```

### 自定义bean
* 给前面的CommonBean加上构造函数：
```
package com.fuyi.test.beanpostprocessor;

public class CommonBean {
	
	public CommonBean(){
		System.out.println("构造方法");
	}

	private String name;
	public void setName(String name) {
		this.name = name;
	}
	
	public void init() {
		System.out.println("init method");
	}
}

```
* 定义实例化bean后处理器

```
package com.fuyi.test.beanpostprocessor;

import java.beans.PropertyDescriptor;

import org.springframework.beans.BeansException;
import org.springframework.beans.PropertyValues;
import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor;

public class InstantiationBeanPostProcessor implements InstantiationAwareBeanPostProcessor {

	/*****BeanPostProcessor接口方法*******************/
	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("init before");
		return bean;
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("init after");
		return bean;
	}

	/*****InstantiationAwareBeanPostProcessor继承BeanPostProcessor，新增的方法****************************/
	@Override
	public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		System.out.println("instantiation before");
		return null;
	}

	@Override
	public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		System.out.println("instantication after");
		return true;
	}

	@Override
	public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean,
			String beanName) throws BeansException {
		System.out.println("postProcessPropertyValues" + pvs);
		return pvs;
	}

}

```
### 配置

```
	<bean id="commonBean0" class="com.fuyi.test.beanpostprocessor.CommonBean" init-method="init">
		<property name="name" value="commonBean0"></property>
	</bean>
	
	<bean id="commonBean1" class="com.fuyi.test.beanpostprocessor.CommonBean" init-method="init">
		<property name="name" value="commonBean1"></property>
	</bean>
	
	<bean class="com.fuyi.test.beanpostprocessor.InstantiationBeanPostProcessor"/>
```
### 启动Spring容器后，结果如下：

因为配置了2个CommonBean，所以实例化了2个该对象实例。
```
instantiation before
构造方法
instantication after
postProcessPropertyValuesPropertyValues: length=1; bean property 'name'
init before
init method
init after
instantiation before
构造方法
instantication after
postProcessPropertyValuesPropertyValues: length=1; bean property 'name'
init before
init method
init after
```
### 总结

1. Bean构造出来之前调用postProcessBeforeInstantiation()方法

2. Bean构造出来之后调用postProcessAfterInstantiation()方法

不过通常来讲，我们不会直接实现InstantiationAwareBeanPostProcessor接口，而是会采用继承InstantiationAwareBeanPostProcessorAdapter这个抽象类的方式来使用。

