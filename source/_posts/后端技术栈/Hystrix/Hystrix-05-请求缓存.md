---
title: Hystrix-05-请求缓存
date: 2018-06-13 16:52:26
tags: Hystrix
---

### 请求缓存 Request-Cache
请求缓存可以让(CommandKey/CommandGroup)相同的情况下,直接共享结果，降低依赖调用次数，在高并发和CacheKey碰撞率高场景下可以提升性能.

### 使用demo
```
package com.fuyi.hystrix.command;

import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.HystrixCommandKey;
import com.netflix.hystrix.HystrixRequestCache;
import com.netflix.hystrix.strategy.concurrency.HystrixConcurrencyStrategy;
import com.netflix.hystrix.strategy.concurrency.HystrixConcurrencyStrategyDefault;
import com.netflix.hystrix.strategy.concurrency.HystrixRequestContext;

public class CacheCommand extends HystrixCommand<String> {

	private static int id;
	private static final HystrixCommandKey COMMAND_KEY = HystrixCommandKey.Factory.asKey("getUserById");
	
	public CacheCommand(int id) {
		super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("userServiceGroup"))
					.andCommandKey(COMMAND_KEY));
		this.id = id;
	}
	
	@Override
	protected String run() throws Exception {
		System.out.println("thread: " + Thread.currentThread().getName());
		return "executed = " + id;
	}
	
	/**
	 * 重写该方法，实现区分不同请求的逻辑
	 */
	@Override
	protected String getCacheKey() {
		return String.valueOf(id);
	}
	
	// 清理缓存
	public static void clearCache() {
		// 从默认的Hystrix并发策略中，根据COMMAND_KEY获取到该命令的请求缓存对象HystrixRequestCache的实例
		HystrixRequestCache instance = HystrixRequestCache.getInstance(COMMAND_KEY, HystrixConcurrencyStrategyDefault.getInstance());
		// 调用请求缓存对象实例的clear方法，对Key为id的缓存值进行清理
		instance.clear(String.valueOf(id));
	}
	
	public static void main(String[] args) {
		HystrixRequestContext ctx = HystrixRequestContext.initializeContext();
		try {
			CacheCommand ca = new CacheCommand(2);
			CacheCommand cb = new CacheCommand(2);
			
			// isResponseFromCache判定是否是在缓存中获取结果  
			String result = ca.execute(); 
			System.out.println(ca.isResponseFromCache()); //false
			System.out.println("result : " + result);
			
			// 清理缓存代码，使用场景：getUserById配置了缓存，在UpdateUserById中要清理缓存
			// CacheCommand.clearCache();
			
			result = cb.execute();
			System.out.println(cb.isResponseFromCache()); //true
			System.out.println("result : " + result);
		} finally {
			ctx.shutdown();
		}
		
		// 在不同context中的缓存是不共享的，还有这个request内部一个ThreadLocal，所以request只能限于当前线程。
		ctx = HystrixRequestContext.initializeContext();
		
		try {
			CacheCommand cc = new CacheCommand(2); 
			String result = cc.execute();
			System.out.println(cc.isResponseFromCache()); // false
			System.out.println("result : " + result);
		} finally {
			ctx.shutdown();
		}
		
	}
}


```
### 运行结果

```
thread: hystrix-userServiceGroup-1
false
result : executed = 2
true
result : executed = 2
thread: hystrix-userServiceGroup-2
false
result : executed = 2
```
