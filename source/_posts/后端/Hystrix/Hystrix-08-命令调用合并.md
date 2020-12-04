---
title: Hystrix-08-命令调用合并
date: 2018-06-13 16:52:59
tags: Hystrix
---

### 命令调用合并:HystrixCollapser
命令调用合并允许多个请求合并到一个线程/信号下批量执行。

* 使用场景

HystrixCollapser用于对多个相同业务的请求合并到一个线程甚至可以合并到一个连接中执行，降低线程交互次和IO数,但必须保证他们属于同一依赖.

执行流程图如下:

![image](https://note.youdao.com/yws/api/personal/file/15B272F2E6CA47E39F179E3B988F6BE7?method=download&shareKey=6fb9281e7ac435c526bbd79ad1d29581)

### 使用demo
```
package com.fuyi.hystrix.command;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.concurrent.Future;

import com.netflix.hystrix.HystrixCollapser;
import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.HystrixCommandKey;
import com.netflix.hystrix.HystrixRequestLog;
import com.netflix.hystrix.strategy.concurrency.HystrixRequestContext;

public class RequestCollapser extends HystrixCollapser<List<String>, String, Integer> {

	private final Integer key;
	public RequestCollapser(Integer key) {
		this.key = key;
	}
	
	@Override
	public Integer getRequestArgument() {
		return key;
	}

	@Override
	protected HystrixCommand<List<String>> createCommand(Collection<CollapsedRequest<String, Integer>> requests) {
		return new BatchCommand(requests);
	}

	@Override
	protected void mapResponseToRequests(
			List<String> batchResponse, 
			Collection<CollapsedRequest<String, Integer>> requests) {
		int count = 0;
		for (CollapsedRequest<String, Integer> request : requests) {
			request.setResponse(batchResponse.get(count++));
		}
	}
	
	private static final class BatchCommand extends HystrixCommand<List<String>> {

		private Collection<CollapsedRequest<String, Integer>> requests;
		
		protected BatchCommand(Collection<CollapsedRequest<String, Integer>> requests) {
			super(Setter
					.withGroupKey(HystrixCommandGroupKey.Factory.asKey("userServiceGroup"))
					.andCommandKey(HystrixCommandKey.Factory.asKey("userServiceKey")));
			this.requests = requests;
		}

		@Override
		protected List<String> run() throws Exception {
			ArrayList<String> responseList = new ArrayList<String>();
			for (CollapsedRequest<String, Integer> request : requests) {
				responseList.add("valueForKey " + request.getArgument());
			}
			return responseList;
		}
		
	}
	
	public static void main(String[] args) throws Exception {
		HystrixRequestContext context = HystrixRequestContext.initializeContext();
		
		Future<String> f1 = new RequestCollapser(1).queue();
		Future<String> f2 = new RequestCollapser(2).queue();
		Future<String> f3 = new RequestCollapser(3).queue();
		Future<String> f4 = new RequestCollapser(4).queue();
		
		System.out.println(f1.get());
		System.out.println(f2.get());
		System.out.println(f3.get());
		System.out.println(f4.get());
		
		int size = HystrixRequestLog.getCurrentRequest().getExecutedCommands().size();
		System.out.println("execute commands size: " + size);
		
		HystrixCommand command = HystrixRequestLog.getCurrentRequest().getExecutedCommands().toArray(new HystrixCommand<?>[1])[0];
		System.out.println(command.getCommandKey().name());
		System.out.println(command.getExecutionEvents());
		
		context.shutdown();
	}

}

```
