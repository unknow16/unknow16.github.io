---
title: Hystrix-02-执行命令方式
date: 2018-06-13 16:51:57
tags: Hystrix
---

### 前言
使用HystrixCommand或HystrixObservableCommand 对要调用的依赖服务进行包装后，便具有了依赖隔离和断路器的功能。

- HystrixCommand 用于返回单一的响应
- HystrixObservableCommand 用于返回多个可自定义的响应

### 执行命令方式
HystrixCommand执行调用依赖服务的方法有下列4个

而HystrixObservableCommand只有后两个

方法 | 简介
---|---
execute() | 同步调用，在超时时间内，阻塞等待至结果返回
queue() | 	异步调用，返回 java.util.concurrent.Future
observe() | 异步调用，返回 rx.Observable ，热观察，可以被立即执行，如果订阅了那么会重新通知
toObservable() | 	未调用，冷观察，返回一个Observable对象，当调用此接口，还需要自己加入订阅者，才能接受到信息

如果是继承HystrixObservableCommand，那么就调用construct()函数，如果是继承HystrixCommand，那么就调用run()函数

### 引入依赖

```
	<properties>
		<hystrix.version>1.3.16</hystrix.version>
		<hystrix-metrics-event-stream.version>1.1.2</hystrix-metrics-event-stream.version>
	</properties>

    <!-- hystrix核心 -->
	<dependency>
		<groupId>com.netflix.hystrix</groupId>
		<artifactId>hystrix-core</artifactId>
		<version>${hystrix.version}</version>
	</dependency>
	
    <!-- hystrix监控 -->
	<dependency>
		<groupId>com.netflix.hystrix</groupId>
		<artifactId>hystrix-metrics-event-stream</artifactId>
		<version>${hystrix-metrics-event-stream.version}</version>
	</dependency>
	
	 <!-- hystrix注解相关 -->
	<dependency>
	    <groupId>com.netflix.hystrix</groupId>
	    <artifactId>hystrix-javanica</artifactId>
	    <version>1.5.9</version>
	</dependency>
```
### 代码使用

```
package com.fuyi.hystrix.command;

import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;

import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;

import rx.Observable;
import rx.Observer;
import rx.functions.Action1;

public class HelloCommand extends HystrixCommand<String> {
	
	private final String name;
	
	public HelloCommand(String name) {
	    // 最少配置:指定命令组名(CommandGroup)  
	    // 相同的CommandGroup的Command会使用相同的线程池
	    // 应用中一般采用全接口名作参数，即同一个接口中的所有方法共用一个线程池
	    // 此处配置产生的线程名格式为：hystrix-userService-1，1是递增的数字
		super(HystrixCommandGroupKey.Factory.asKey("userService"));
		this.name = name;
	}

	/**
	 * 调用需隔离的 依赖逻辑写在该方法中
	 * 该方法会在一个隔离的子线程中执行
	 */
	@Override
	protected String run() throws Exception {
		// 请求依赖的远程服务，获取数据，如：rpc,http等方式
		return "Hello " + name + ", thread: " + Thread.currentThread().getName();
	}

	/**
	 * 每个command的对像只能调用一次，不可以重复调用
	 * 重复调用对应异常信息: This instance can only be executed once. Please instantiate a new instance.
	 */
	public static void main(String[] args) throws Exception {
		HelloCommand helloCommand = new HelloCommand("sync-hystrix");
		
		// 1. 同步调用，即同步执行上面run()
		//    execute()方法内部为：helloCommand.queue().get()
		String result = helloCommand.execute(); 
		System.out.println("result: " + result);
		
		// 2. 异步调用，即异步执行上面run()
		// 	      异步调用,可自由控制获取结果时机, 
		helloCommand = new HelloCommand("async-hystrix");
		Future<String> future = helloCommand.queue();
		
		// get操作不能超过command定义的超时时间,默认:1秒 
		result = future.get(100, TimeUnit.MILLISECONDS);
		System.out.println("result: " + result);
		
		// 3. 异步调用返回rx.Observable
		helloCommand = new HelloCommand("rx-async-hystrix");
		Observable<String> observe = helloCommand.observe();
		
		// 仅订阅获取结果事件，进行处理
		observe.subscribe(new Action1<String>() {
			@Override
			public void call(String result) {
				// 此处result为helloCommand的run()方法返回的结果
				// 可以对结果进行二次处理
				System.out.println("single : " + result);
			}
		});
		
		// 订阅完整的事件过程，进行处理
		observe.subscribe(new Observer<String>() {

            // onNext/onError完成之后最后回调  
			@Override
			public void onCompleted() {
				System.out.println("onCompleted");
			}

            // 当产生异常时回调  
			@Override
			public void onError(Throwable t) {
				System.out.println("error: " + t.getMessage());
			}

            // 获取结果后回调
			@Override
			public void onNext(String result) {
				System.out.println("all : " + result);
			}
		});
		
		// 主线程中
		System.out.println("mainThread=" + Thread.currentThread().getName());
	}
}

```
### 执行结果

```
result: Hello sync-hystrix, thread: hystrix-userService-1
result: Hello async-hystrix, thread: hystrix-userService-2
mainThread=main
single : Hello rx-async-hystrix, thread: hystrix-userService-3
all : Hello rx-async-hystrix, thread: hystrix-userService-3
onCompleted

```

### 源码实现
```
@ThreadSafe
public abstract class HystrixCommand<R> implements HystrixExecutable<R> {

    // 1. toObservable()方法中为核心实现
    // 未做订阅，返回干净的 Observable 。这就是为什么上文说“未调用” 。
    public Observable<R> toObservable() {
        if (properties.executionIsolationStrategy().get().equals(ExecutionIsolationStrategy.THREAD)) {
            return toObservable(Schedulers.computation());
        } else {
            // 如果隔离策略是信号量，则所有请求阻塞同步执行
            // 在当前调用该方法的线程中执行
            // semaphore isolation is all blocking, no new threads involved
            // so we'll use the calling thread
            return toObservable(Schedulers.immediate());
        }
    }
    
    // 2. observe()方法
    public Observable<R> observe() {
        // us a ReplaySubject to buffer the eagerly subscribed-to Observable
        ReplaySubject<R> subject = ReplaySubject.create();
        
        // 利用toObservable()方法返回的Observable让subject订阅，并返回其
        // ReplaySubject 会发射所有来自原始 Observable 的数据给观察者，无论它们是何时订阅的。
        // eagerly kick off subscription
        toObservable().subscribe(subject);
        // return the subject that can be subscribed to later while the execution has already started
        return subject;
    }
    
    // 3. queue()方法
    public Future<R> queue() {
    
        // 利用toObservable()的toBlockingObservable()阻塞到Future中返回
        final ObservableCommand<R> o = toObservable(Schedulers.immediate(), false);
        final Future<R> f = o.toBlockingObservable().toFuture();
        
        // ...
        return f;   
    }
    
    // 4. execute()方法
    public R execute() {
        try {
            // 调用queue()方法返回的Future阻塞的get()方法获取结果
            return queue().get();
        } catch (Exception e) {
            throw decomposeException(e);
        }
    }
}


```
