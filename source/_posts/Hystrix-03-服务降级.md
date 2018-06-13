---
title: Hystrix-03-服务降级
date: 2018-06-13 16:52:07
tags: Hystrix
---

### 简介
* 实现降级，即重写HystrixCommand的getFallback()方法，其中返回调用依赖失败时返回的降级数据。

* 除了HystrixBadRequestException异常之外，所有从run()方法抛出的异常都算作失败，并触发降级getFallback()和断路器逻辑。

* HystrixBadRequestException用在非法参数或非系统故障异常等不应触发回退逻辑的场景

### 使用代码
```
package com.fuyi.hystrix.command;

import java.util.concurrent.TimeUnit;

import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.HystrixCommandKey;
import com.netflix.hystrix.HystrixCommandProperties;
import com.netflix.hystrix.HystrixThreadPoolKey;

public class FallbackCommand extends HystrixCommand<String> {

	private final String name;
	
	protected FallbackCommand(String name) {
		super(Setter
				.withGroupKey(HystrixCommandGroupKey.Factory.asKey("userServiceGroup"))
				.andCommandPropertiesDefaults(
						// 配置调用依赖超时时间为500，超时后调用降级方法
						HystrixCommandProperties.Setter().withExecutionIsolationThreadTimeoutInMilliseconds(500)
					));
		this.name = name;
	}

	@Override
	protected String run() throws Exception {
		// 此处睡眠1秒，模拟调用超时，配置超时时间为500毫秒
		TimeUnit.MILLISECONDS.sleep(1000);
		return "Hello " + name +"  thread:" + Thread.currentThread().getName();
	}

	// 超时后调用的降级逻辑方法
	@Override
	protected String getFallback() {
		return "execute falled";
	}
	
	public static void main(String[] args) {
		FallbackCommand c = new FallbackCommand("fuyi");
		String result = c.execute();
		System.out.println(result);
	}
}

```
* 执行结果

```
execute falled

//如果把run()中的睡眠1秒去掉，则返回run()方法结果
Hello fuyi  thread:hystrix-userServiceGroup-1
```
