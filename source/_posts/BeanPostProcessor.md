---
title: BeanPostProcessor
date: 2018-03-01 15:28:21
tags: Spring
---

### 简介
之前的InitializingBean、DisposableBean、FactoryBean包括init-method和destory-method，针对的都是某个Bean控制其初始化的操作，而似乎没有一种办法可以针对每个Bean的生成前后做一些逻辑操作，PostProcessor则帮助我们做到了这一点，先看一个简单的BeanPostProcessor。

网上有一张图画了Bean生命周期的过程，画得挺好
![image](https://note.youdao.com/yws/api/personal/file/BBE9E4B8D22B491AB28ABCEC176A81B7?method=download&shareKey=3f8451258cff9847b8f668c2d0a3ae7f)

### BeanPostProcessor接口

针对每个bean生成前后都会调用
```

package org.springframework.beans.factory.config;

import org.springframework.beans.BeansException;

public interface BeanPostProcessor {
    //见名知意，在初始化Bean之前
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
    
    //见名知意，在初始化Bean之前
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```
这两个方法是有返回值的，不要返回null，否则getBean的时候拿不到对象。

### 自定义bean

```
//普通bean
public class CommonBean {

	private String name;
	public void setName(String name) {
		this.name = name;
	}
	
	public void init() {
		System.out.println("init method");
	}
}

//bean后处理器
public class PostProcessorBean implements BeanPostProcessor {

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("postProcessAfterInitialization, beanName = " + beanName);
		return bean;
	}

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("postProcessBeforeInitialization, beanName = " + beanName);
		return bean;
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
	
	<bean class="com.fuyi.test.beanpostprocessor.PostProcessorBean"></bean>
```
### 测试

```
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
	public void testBeanPostProcessor(){
		Object bean = ac.getBean("commonBean0");
	}
}

```
### 结果
```
postProcessBeforeInitialization, beanName = commonBean0
init method
postProcessAfterInitialization, beanName = commonBean0
postProcessBeforeInitialization, beanName = commonBean1
init method
postProcessAfterInitialization, beanName = commonBean1
```

