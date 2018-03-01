---
title: Bean的作用域
date: 2018-03-01 09:38:19
tags: Spring
---

### Bean的作用域类型
* spring提供了“singleton”和“prototype”两种基本的作用域
* 另外提供了三种web作用域：“request”,"session","global session"
* 还支持自定义作用域

### Singleton
单例的bean在Spring IoC容器中只存在一个实例，其生命周期由spring完全控制，任何获取bean的操作将返回同一个实例。
> GoF中的单例设计模式指出：保证一个类仅有一个实例，并提供一个获取它的全局访问点，介绍了两种实现，通过在类上声明静态成员变量持有它或通过静态注册表方式。

Spring采用了静态注册表的方式实现，通过实现SingleBeanRegistry接口，属于无侵入的实现，而前者属于侵入的。Spring不仅会缓存Bean实例，也会缓存Bean的定义，用于对懒加载的bean，在使用时进行实例化，并放入单例缓存池中。

### Prototype
每次向Spring IoC容器请求获取一个bean时，都会创建一个新的bean返回，不会被缓存。
> GoF中的原型设计模式指出：原型实例是创建对象的种类，并且通过拷贝创建新实例，即克隆操作。

Spring中则略有不同，通过Bean定义创建新实例，用户只需配置即可，属于无侵入式的。

### SpringMVC的Controller类默认Scope是单例的
只要Controller中不存在成员变量，多线程时就不存在共享资源竞争，单例时就是线程安全的，提高资源利用率和性能。单例中的成员方法被多线程访问时是线程安全的。

如果必须需要成员变量，则支持将Controller声明为原型（prototype）的，即每次请求的每个线程中都会创建新的Controller实例，但应尽量避免声明成员变量。用法为在Controller类上加@Scope(value="prototype")即可。

因为struts2中的Action是采用成员变量接收请求参数的，所以的基于原型的，来避免多线程访问时的安全问题。SpringMVC采用了方法的形参接收请求参数，每个请求一个线程，每个线程的自己独立的堆栈空间存储方法内的局部变量，对其操作不会影响其他线程，所以不存在多线程访问安全问题。

### Spring IoC容器中创建的Bean默认Scope都是单例的
