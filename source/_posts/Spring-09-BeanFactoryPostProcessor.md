---
title: Spring-09-BeanFactoryPostProcessor
date: 2018-03-01 15:29:16
tags: Spring
---

### 简介
接下来看另外一个PostProcessor----BeanFactoryPostProcessor。

Spring允许在Bean创建之前，读取Bean的元属性，并根据自己的需求对元属性进行改变，比如将Bean的scope从singleton改变为prototype，最典型的应用应当是PropertyPlaceholderConfigurer，替换xml文件中的占位符，替换为properties文件中相应的key对应的value，这将会在下篇文章中专门讲解PropertyPlaceholderConfigurer的作用及其原理。

BeanFactoryPostProcessor就可以帮助我们实现上述的功能，下面来看一下BeanFactoryPostProcessor的使用，定义一个BeanFactoryPostProcessor的实现类：

### BeanFactoryPostProcessor接口

```
public interface BeanFactoryPostProcessor {
    
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}
```

### 自定义bean

测试的PostProcessorBean增加实现了BeanFactoryPostProcessor接口如下，其他和BeanFactoryProcessor中测试用例一样。
```
public class PostProcessorBean implements BeanPostProcessor, BeanFactoryPostProcessor {

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

	//后处理beanFactory
	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
		System.out.println("configurableListableBeanFactory = " + configurableListableBeanFactory);
	}

}

```
### 测试结果

```
configurableListableBeanFactory = org.springframework.beans.factory.support.DefaultListableBeanFactory@6d4b1c02: defining beans [commonBean0,commonBean1,com.fuyi.test.beanpostprocessor.PostProcessorBean#0]; root of factory hierarchy
postProcessBeforeInitialization, beanName = commonBean0
init methon
postProcessAfterInitialization, beanName = commonBean0
postProcessBeforeInitialization, beanName = commonBean1
init methon
postProcessAfterInitialization, beanName = commonBean1
```
* 从执行结果中可以看出两点：

1. BeanFactoryPostProcessor的执行优先级高于BeanPostProcessor

2. BeanFactoryPostProcessor的postProcessBeanFactory()方法只会执行一次

注意到postProcessBeanFactory方法是带了参数ConfigurableListableBeanFactory的，这就和我之前说的可以使用BeanFactoryPostProcessor来改变Bean的属性相对应起来了。ConfigurableListableBeanFactory功能非常丰富，最基本的，它携带了每个Bean的基本信息，比如我简单写一段代码：
```
	//后处理beanFactory
	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
		System.out.println("configurableListableBeanFactory = " + configurableListableBeanFactory);
		
		BeanDefinition beanDefinition = configurableListableBeanFactory.getBeanDefinition("commonBean0");
	    MutablePropertyValues beanProperty = beanDefinition.getPropertyValues();
	   
	    System.out.println("scope before change：" + beanDefinition.getScope());
	    beanDefinition.setScope("singleton"); //设置作用域为单例
	    System.out.println("scope after change：" + beanDefinition.getScope());
	    System.out.println("beanProperty：" + beanProperty);
	}
```
结果为

```
configurableListableBeanFactory = org.springframework.beans.factory.support.DefaultListableBeanFactory@6d4b1c02: defining beans [commonBean0,commonBean1,com.fuyi.test.beanpostprocessor.PostProcessorBean#0]; root of factory hierarchy
scope before change：
scope after change：singleton
beanProperty：PropertyValues: length=1; bean property 'name'
postProcessBeforeInitialization, beanName = commonBean0
init methon
postProcessAfterInitialization, beanName = commonBean0
postProcessBeforeInitialization, beanName = commonBean1
init methon
postProcessAfterInitialization, beanName = commonBean1
```
这样就获取了Bean的生命周期以及重新设置了Bean的生命周期。ConfigurableListableBeanFactory还有很多的功能，比如添加BeanPostProcessor，可以自己去查看。