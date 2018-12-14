---
title: Spring-15-BeanDefinitionReader
date: 2018-12-14 15:57:50
tags: Spring
---

## BeanDefinitionReader接口
目前有3种实现
- GroovyBeanDefinitionReader：groovy文件的读取器
- PropertiesBeanDefinitionReader：Properties文件的读取器
- XmlBeanDefinitionReader：xml文件的读取器

这3个实现类都继承AbstractBeanDefinitionReader，AbstractBeanDefinitionReader抽象类内部有个BeanDefinitionRegistry接口类型的属性，BeanDefinitionRegistry接口存在的意义在于对bean数据的管理，包括bean的注册、删除、查找等。这3个实现类内部最终对bean的注册都是通过BeanDefinitionRegistry完成的，不同点在于它们处理过程不一样，比如xml文件的解析和properties文件的解析这个过程不一样

## AnnotatedBeanDefinitionReader类
独立的一个类，用来注册单独的类，也是使用BeanDefinitionRegistry接口类型的属性完成bean的注册。

构造的时候会使用AnnotationConfigUtils的registerAnnotationConfigProcessors方法预先注册一些processor bean，比如ConfigurationClassPostProcessor、AutowiredAnnotationBeanPostProcessor、RequiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor，其中ConfigurationClassPostProcessor中会使用下面提到的解析@Configuration注解的ConfigurationClassParser和ConfigurationClassParser类。


------


## ClassPathScanningCandidateComponentProvider类
独立的一个类，用来找出具体包下的bean信息，内部有2个TypeFilter集合属性，includeFilters和excludeFilters，分别用于对找出的bean信息做匹配，includeFilters中的TypeFilter只要有一个满足，就不会过滤；excludeFilters中的TypeFilter只要有一个满足，就会被过滤。

## ClassPathBeanDefinitionScanner类
ClassPathBeanDefinitionScanner是ClassPathScanningCandidateComponentProvider类的子类，提供了scan方法，这个scan方法会找出包下的bean信息并使用BeanDefinitionRegistry接口类型的属性完成bean的注册

扫描具体包下的类，扫描完之后根据includeAnnotationConfig属性是否使用AnnotationConfigUtils的registerAnnotationConfigProcessors方法预先注册一些processor bean。includeAnnotationConfig属性默认是true，可修改

## ComponentScanAnnotationParser类
独立的一个类，@ComponentScan注解对应的解析器，内部使用ClassPathBeanDefinitionScanner完成


----


## ConfigurationClassParser类
独立的一个类，用来解析被@Configuration注解修饰的配置类。

简单点来说就是ConfigurationClassParser会解析被@Configuration注解修饰的类，然后再处理这个类内部被其它注解修饰的情况，比如@Import注解、@ComponentScan注解、@ImportResource注解、@Bean注解等。这里解析过程中也会遇到其它被@Configuration注解修饰的类，这些类会放到ConfigurationClassParser的configurationClasses属性中然后被ConfigurationClassBeanDefinitionReader处理

ConfigurationClassParser和ConfigurationClassBeanDefinitionReader在ConfigurationClassPostProcessor这个BeanFactoryPostProcessor中使用

## ConfigurationClassBeanDefinitionReader类
独立的一个类，处理ConfigurationClassParser解析出的被@Configuration注解修饰的配置类，会处理配置类内部的被@Bean注解修饰的方法、@ImportResource注解修饰的资源、@Import注解修饰的ImportBeanDefinitionRegistrar接口。最后使用BeanDefinitionRegistry注册


## BeanDefinitionLoader
SpringBoot中的入口，组合上面所列各种BeanDefinitionReader

SpringApplication#run()方法中的prepareContext() # load() 中 new BeanDefinitionLoader，然后去加载各种source。
```
private int load(Object source) {
	Assert.notNull(source, "Source must not be null");
	if (source instanceof Class<?>) {
		return load((Class<?>) source);
	}
	if (source instanceof Resource) {
		return load((Resource) source);
	}
	if (source instanceof Package) {
		return load((Package) source);
	}
	if (source instanceof CharSequence) {
		return load((CharSequence) source);
	}
	throw new IllegalArgumentException("Invalid source type " + source.getClass());
}
```

目前支持4种类型的source，分别是：

1. Class类型：使用AnnotatedBeanDefinitionReader读取器加载
1. Resource类型：使用XmlBeanDefinitionReader读取器加载
1. Package类型：使用ClassPathBeanDefinitionScanner扫描器加载
1. CharSequence：识别这个字符串信息。如果是Class类型，用第1种；如果是Resource类型，用第2种；如果是Package类型，用第3种

#### Class类型：


```
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        // Class类型，使用AnnotatedBeanDefinitionReader加载
        // 加载完毕之后使用ConfigurationClassPostProcessor进行后续的处理
        SpringApplication.run(MyApplication.class, args);
    }
}
```

#### Resource类型：


```
SpringApplication.run(
new Object[] {
        MyApplication.class
        , new ClassPathResource("beans.xml") // 使用XmlBeanDefinitionReader加载
}
, args);
```

#### Package类型：


```
SpringApplication.run(
new Object[] {
        MyApplication.class
        , new ClassPathResource("beans.xml")
        , OtherBean.class.getPackage() // 使用ClassPathBeanDefinitionScanner加载
}
, args);
```
