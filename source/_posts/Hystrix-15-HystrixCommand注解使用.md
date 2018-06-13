---
title: Hystrix-15-@HystrixCommand注解使用
date: 2018-06-13 16:54:42
tags: Hystrix
---

因为使用的是注解方式，所以我们需要设置Aspect
* 依赖（hystrix-core至少要1.5.9以上）

```
		<dependency>
			<groupId>com.netflix.hystrix</groupId>
			<artifactId>hystrix-core</artifactId>
			<version>1.5.9</version>
		</dependency>
		<dependency>
			<groupId>com.netflix.hystrix</groupId>
			<artifactId>hystrix-metrics-event-stream</artifactId>
			<version>1.1.2</version>
		</dependency>
		<dependency>
		    <groupId>com.netflix.hystrix</groupId>
		    <artifactId>hystrix-javanica</artifactId>
		    <version>1.5.9</version>
		</dependency>
```

* 启动类
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
	
	// 向Spring容器注入HystrixCommandAspect
	@Bean(name = "hystrixAspect")
	public HystrixCommandAspect hystrixCommandAspect() {
		return new HystrixCommandAspect();
	}
}
```
* 控制器

```
@Controller
public class HelloController {
	
	// 注解配置
	@RequestMapping("/anno")
	@ResponseBody
	@HystrixCommand(groupKey = "annoGroup", commandKey = "annoCommand", fallbackMethod = "annoFallback", 
					commandProperties = {
							@HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
							@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "500")
					})
	public Object anno() {
		try {
			TimeUnit.MILLISECONDS.sleep(600);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("anno method thread: " + Thread.currentThread().getName());
		return "anno";
	}
	
	// 失败回调方法
	public Object annoFallback() {
		System.out.println("annoFallback method thread: " + Thread.currentThread().getName());
		return "annoFallback";
	}
}

```
