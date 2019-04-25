---
title: Hystrix-06-信号量隔离
date: 2018-06-13 16:52:38
tags: Hystrix
---

### 信号量隔离:SEMAPHORE
隔离本地代码或可快速返回远程调用(如memcached,redis)可以直接使用信号量隔离,降低线程隔离开销.

### 使用demo

```
package com.fuyi.hystrix.command;

import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.HystrixCommandProperties;

public class SemaphoreCommand extends HystrixCommand<String> {

	private final String name;
	
	public SemaphoreCommand(String name) {
		super(Setter
				.withGroupKey(HystrixCommandGroupKey.Factory.asKey("userServiceGroup"))
				.andCommandPropertiesDefaults(
						HystrixCommandProperties.Setter()
							.withExecutionIsolationStrategy(
									HystrixCommandProperties.ExecutionIsolationStrategy.SEMAPHORE
									)
						)
				);
		this.name = name;	
	}
	
	@Override
	protected String run() throws Exception {
		System.out.println("thread : " + Thread.currentThread().getName());
		return "hello";
	}

	public static void main(String[] args) {
		SemaphoreCommand c = new SemaphoreCommand("hh");
		String result = c.execute();
		System.out.println(result);
		System.out.println("thread : " + Thread.currentThread().getName());
	}
}

```
### 执行结果

```
thread : main
hello
thread : main
```
