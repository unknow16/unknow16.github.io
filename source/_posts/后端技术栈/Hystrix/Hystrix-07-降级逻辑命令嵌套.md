---
title: Hystrix-07-降级逻辑命令嵌套
date: 2018-06-13 16:52:49
tags: Hystrix
---

### 降级逻辑命令嵌套
适用场景:用于fallback逻辑涉及网络访问的情况,如缓存访问。

依赖调用和降级调用使用不同的线程池做隔离，防止上层线程池跑满，影响二级降级逻辑调用

### 使用demo

```
package com.fuyi.hystrix.command;

import java.util.concurrent.TimeUnit;

import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.HystrixCommandProperties;

public class MultiFallbackCommand extends HystrixCommand<String> {

	protected MultiFallbackCommand() {
		super(Setter
				.withGroupKey(HystrixCommandGroupKey.Factory.asKey("userServiceGroup"))
				.andCommandPropertiesDefaults(
						// 配置调用依赖超时时间为500，超时后调用降级方法
						HystrixCommandProperties.Setter().withExecutionIsolationThreadTimeoutInMilliseconds(500)
					));
	}

	
	
	/**
	 * 1. 从缓存获取数据
	 */
	@Override
	protected String run() throws Exception {
		throw new RuntimeException();
	}

	/**
	 * 2. 请求缓存超时，请求db
	 */
	@Override
	protected String getFallback() {
		System.out.println("fallback thread: " + Thread.currentThread().getName());
		return new FallbackCommand().execute();
	}
	
	/**
	 * 嵌套容错包装command
	 */
	static class FallbackCommand extends HystrixCommand<String> {

		protected FallbackCommand() {
			super(Setter
					.withGroupKey(HystrixCommandGroupKey.Factory.asKey("userServiceGroup"))
					.andCommandPropertiesDefaults(
							// 配置调用依赖超时时间为500，超时后调用降级方法
							HystrixCommandProperties.Setter().withExecutionIsolationThreadTimeoutInMilliseconds(500)
						));
		}

		/**
		 * 3. 请求db超时
		 */
		@Override
		protected String run() throws Exception {
			throw new RuntimeException();
		}
		
		/**
		 * 4. 返回默认值
		 */
		@Override
		protected String getFallback() {
			return "defalut value";
		}
	}
	
	public static void main(String[] args) {
		MultiFallbackCommand c = new MultiFallbackCommand();
		String result = c.execute();
		
		System.out.println(result);
	}
}

```
### 执行结果

```
fallback thread: hystrix-userServiceGroup-1
defalut value
```

