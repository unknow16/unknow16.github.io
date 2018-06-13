---
title: Hystrix-13-HystrixCommand执行流程分析
date: 2018-06-13 16:54:08
tags: Hystrix
---

### 流程图
![image](https://note.youdao.com/yws/api/personal/file/3333672620AB476B88FA93B3C97D1641?method=download&shareKey=9f42b0121a5f40e8005183268d52ecb4)

### 流程说明
流程说明. 

1. 每次调用创建一个新的HystrixCommand,把依赖调用封装在run()方法中.

2. 执行execute()/queue做同步或异步调用.

3. 判断熔断器(circuit-breaker)是否打开,如果打开跳到步骤8,进行降级策略,如果关闭进入步骤.

4. 判断线程池/队列/信号量是否跑满，如果跑满进入降级步骤8,否则继续后续步骤.

5. 调用HystrixCommand的run方法.运行依赖逻辑

    5a. 依赖逻辑调用超时,进入步骤8.

6. 判断逻辑是否调用成功

    6a. 返回成功调用结果
    
    6b. 调用出错，进入步骤8.

7. 计算熔断器状态,所有的运行状态(成功, 失败, 拒绝,超时)上报给熔断器，用于统计从而判断熔断器状态.

8. getFallback()降级逻辑.

  以下四种情况将触发getFallback调用：

-   run()方法抛出非HystrixBadRequestException异常。
-   run()方法调用超时
-   熔断器开启拦截调用
-   线程池/队列/信号量是否跑满

    8a. 没有实现getFallback的Command将直接抛出异常
    
    8b. fallback降级逻辑调用成功直接返回
    
    8c. 降级逻辑调用失败抛出异常

9. 返回执行成功结果