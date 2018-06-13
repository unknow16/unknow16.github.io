---
title: Hystrix-16-缓存相关注解使用
date: 2018-06-13 16:54:54
tags: Hystrix
---

### 前言
在SpringMVC中使用时，在一次请求Controller时，其中多次调用被HystrixCommand封装的依赖时，如果配置了缓存，除第一次外，后面访问会走缓存。本次请求返回后，缓存即失效。

再一次请求Controller时，调用被HystrixCommand封装的依赖时,缓存又重新建立，返回后又被销毁。

即缓存的生命周期只限一次请求中。

请求缓存在run()和construct()执行之前生效，所以可以有效减少不必要的线程开销

### 通过HystrixCommand类实现
继承HystrixCommand或HystrixObservableCommand，覆盖getCacheKey()方法，指定缓存的key，开启缓存配置。

具体代码，可见前文。

### 通过注解实现
*  首先，编写Filter初始化HystrixRequestContext
```
package com.fuyi.hystrix.filter;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;

import com.netflix.hystrix.strategy.concurrency.HystrixRequestContext;

public class HystrixContextInitFilter implements Filter {

	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		HystrixRequestContext context = HystrixRequestContext.initializeContext();
		try {
			chain.doFilter(request, response);
		} finally {
			context.shutdown();
		}
	}

	@Override
	public void destroy() {
	}

}

```

*  service实现
```
@Service("userService")
public class UserService {

	// 该注解标记请求依赖的结果应该被缓存
	// 默认缓存key使用方法的所有参数，可自定义，下文说
	@CacheResult
	@HystrixCommand(groupKey = "getUserById")
	public String getUserById(String id) {
		System.out.println("from db query user in thread : " + Thread.currentThread().getName());
		return "fuyi";
	}
	
	// 该注解让指定的commandKey的缓存结果失效
	@CacheRemove(commandKey = "getUserById")
	@HystrixCommand(groupKey = "updateUserById")
	public void updateUserById(String id) {
		System.out.println("update user");
	}
}

```

*  controller
```
@Controller
public class HelloController {
	
	@Autowired
	private UserService userService;
	
	@RequestMapping("/getUserById")
	@ResponseBody
	public String getUserById(String id) {
		userService.getUserById(id); // out
		userService.getUserById(id);
		
		userService.updateUserById(id);
		
		userService.getUserById(id); // out
		
		return userService.getUserById(id);
	}
	
}
```

*  启动类

```
@SpringBootApplication
public class Main  {

	public static void main(String[] args) {
		SpringApplication.run(Main.class, args);
	}
	
	@Bean
    public ServletRegistrationBean servletRegistrationBean() {
		// ServletName默认值为首字母小写，即myServlet
        return new ServletRegistrationBean(new HystrixMetricsStreamServlet(), "/hystrix.stream");
    }
	
	// 使用缓存时，要初始化HystrixRequestContext上下文
	@Bean
	public FilterRegistrationBean filterRegistrationBean() {
		return new FilterRegistrationBean(new HystrixContextInitFilter());
	}
	
	// 使用注解时，要向spring容器，注入HystrixCommandAspect
	@Bean(name = "hystrixAspect")
	public HystrixCommandAspect hystrixCommandAspect() {
		return new HystrixCommandAspect();
	}
}

```
### 测试
* 访问

http://localhost:1111/getUserById?id=1

* 结果

```
from db query user in thread : hystrix-getUserById-3
update user
from db query user in thread : hystrix-getUserById-4
```
第一次访问时，查询db, 更新后，会再查一次，其他时不会查db.

### 自定义缓存key
* 通过@CacheResult和@CacheRemove的cacheKeyMethod属性定义
```
@Service("userService")
public class UserService {

	// 该注解标记请求依赖的结果应该被缓存
	// 缓存key通过cacheKeyMethod属性指定的方法生成
	@CacheResult(cacheKeyMethod = "getCacheKey")
	@HystrixCommand(groupKey = "getUserById")
	public String getUserById(String id) {
		System.out.println("from db query user in thread : " + Thread.currentThread().getName());
		return "fuyi";
	}
	
	// 生成缓存key的方法
	public String getCacheKey(String id) {
		return id;
	}
	
	// 该注解让指定的commandKey的缓存结果失效
	@CacheRemove(commandKey = "getUserById", cacheKeyMethod = "getCacheKey")
	@HystrixCommand(groupKey = "updateUserById")
	public void updateUserById(String id) {
		System.out.println("update user");
	}
}
```

* 通过@CacheKey注解配合方法参数
```
@Service("userService")
public class UserService {

	// 该注解标记请求依赖的结果应该被缓存
	// 通过@CacheKey指定参数id为缓存key
	@CacheResult
	@HystrixCommand(groupKey = "getUserById")
	public String getUserById( @CacheKey(value = "id") String id) {
		System.out.println("from db query user in thread : " + Thread.currentThread().getName());
		return "fuyi";
	}

	
	// 该注解让指定的commandKey的缓存结果失效
	@CacheRemove(commandKey = "getUserById")
	@HystrixCommand(groupKey = "updateUserById")
	public void updateUserById(String id) {
		System.out.println("update user");
	}
}
```

参数为对象时：
```
@Service("userService")
public class UserService {

	// 该注解标记请求依赖的结果应该被缓存
	// 通过@CacheKey指定User对象中的id属性为缓存key
	@CacheResult
	@HystrixCommand(groupKey = "getUserById")
	public String getUserById( @CacheKey(value = "id") User user) {
		System.out.println("from db query user in thread : " + Thread.currentThread().getName());
		return "fuyi";
	}

	
	// 该注解让指定的commandKey的缓存结果失效
	@CacheRemove(commandKey = "getUserById")
	@HystrixCommand(groupKey = "updateUserById")
	public void updateUserById(String id) {
		System.out.println("update user");
	}
}
```


* 需要注意，如果以上两者同时指定，则前者优先级高于后者，即cacheKeyMethod属性配置优先级高于@CacheKey配置。