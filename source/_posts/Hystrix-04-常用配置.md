---
title: Hystrix-04-常用配置
date: 2018-06-13 16:52:18
tags: Hystrix
---

### 依赖命名:CommandKey
每个CommandKey代表一个依赖抽象,相同的依赖要使用相同的CommandKey名称。依赖隔离的根本就是对相同CommandKey的依赖做隔离.
```
public HelloWorldCommand(String name) {  
    super(Setter
            .withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))  
            /* HystrixCommandKey工厂定义依赖名称 */  
            .andCommandKey(HystrixCommandKey.Factory.asKey("HelloWorld")));  
    this.name = name;  
}  
```

### 依赖分组:CommandGroup
1. 命令分组用于对依赖操作分组,便于统计,汇总等.

2. CommandGroup是每个命令最少配置的必选参数,在不指定ThreadPoolKey的情况下，字面值用于对不同依赖的线程池/信号区分.


```
// 使用HystrixCommandGroupKey工厂定义  
public HelloWorldCommand(String name) {  
    Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("userServiceGroup"))  
}  

// 调用的依赖在的隔离线程名如下
hystrix-userServiceGroup-1 // 1是递增自然数
```


### 线程池/信号:ThreadPoolKey
当对同一业务依赖做隔离时使用CommandGroup做区分,但是对同一依赖的不同远程调用如(一个是redis 一个是http),可以使用HystrixThreadPoolKey做隔离区分.

虽然在业务上都是相同的组，但是需要在资源上做隔离时，可以使用HystrixThreadPoolKey区分.


```
	public HelloCommand(String name) {
		super(Setter
				.withGroupKey(HystrixCommandGroupKey.Factory.asKey("userService"))
				.andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("userServiceThreadPool")));
		this.name = name;
	}
	
	//线程名格式如下：
	hystrix-userServiceThreadPool-1
```
