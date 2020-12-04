---
title: Hystrix-12-参数配置与分析
date: 2018-06-13 16:53:58
tags: Hystrix
---

## Command 配置
配置类：HystrixCommandProperties

构造Command时通过Setter进行配置

```
//使用命令调用隔离方式,默认:采用线程隔离,ExecutionIsolationStrategy.THREAD  
private final HystrixProperty<ExecutionIsolationStrategy> executionIsolationStrategy;   
//使用线程隔离时，调用超时时间，默认:1秒  
private final HystrixProperty<Integer> executionIsolationThreadTimeoutInMilliseconds;   
//线程池的key,用于决定命令在哪个线程池执行  
private final HystrixProperty<String> executionIsolationThreadPoolKeyOverride;   
//使用信号量隔离时，命令调用最大的并发数,默认:10  
private final HystrixProperty<Integer> executionIsolationSemaphoreMaxConcurrentRequests;  
//使用信号量隔离时，命令fallback(降级)调用最大的并发数,默认:10  
private final HystrixProperty<Integer> fallbackIsolationSemaphoreMaxConcurrentRequests;   
//是否开启fallback降级策略 默认:true   
private final HystrixProperty<Boolean> fallbackEnabled;   
// 使用线程隔离时，是否对命令执行超时的线程调用中断（Thread.interrupt()）操作.默认:true  
private final HystrixProperty<Boolean> executionIsolationThreadInterruptOnTimeout;   
// 统计滚动的时间窗口,默认:5000毫秒circuitBreakerSleepWindowInMilliseconds  
private final HystrixProperty<Integer> metricsRollingStatisticalWindowInMilliseconds;  
// 统计窗口的Buckets的数量,默认:10个,每秒一个Buckets统计  
private final HystrixProperty<Integer> metricsRollingStatisticalWindowBuckets; // number of buckets in the statisticalWindow  
//是否开启监控统计功能,默认:true  
private final HystrixProperty<Boolean> metricsRollingPercentileEnabled;   
// 是否开启请求日志,默认:true  
private final HystrixProperty<Boolean> requestLogEnabled;   
//是否开启请求缓存,默认:true  
private final HystrixProperty<Boolean> requestCacheEnabled; // Whether request caching is enabled.  
```
## 熔断器（Circuit Breaker）配置
配置类：HystrixCommandProperties

构造Command时通过Setter进行配置,每种依赖使用一个Circuit Breaker
```
// 熔断器在整个统计时间内是否开启的阀值，默认20秒。也就是10秒钟内至少请求20次，熔断器才发挥起作用  
private final HystrixProperty<Integer> circuitBreakerRequestVolumeThreshold;   
//熔断器默认工作时间,默认:5秒.熔断器中断请求5秒后会进入半打开状态,放部分流量过去重试  
private final HystrixProperty<Integer> circuitBreakerSleepWindowInMilliseconds;   
//是否启用熔断器,默认true. 启动  
private final HystrixProperty<Boolean> circuitBreakerEnabled;   
//默认:50%。当出错率超过50%后熔断器启动.  
private final HystrixProperty<Integer> circuitBreakerErrorThresholdPercentage;  
//是否强制开启熔断器阻断所有请求,默认:false,不开启  
private final HystrixProperty<Boolean> circuitBreakerForceOpen;   
//是否允许熔断器忽略错误,默认false, 不开启  
private final HystrixProperty<Boolean> circuitBreakerForceClosed; 
```
## 命令合并(Collapser)配置
配置类：HystrixCollapserProperties

构造Collapser时通过Setter进行配置

```
//请求合并是允许的最大请求数,默认: Integer.MAX_VALUE  
private final HystrixProperty<Integer> maxRequestsInBatch;  
//批处理过程中每个命令延迟的时间,默认:10毫秒  
private final HystrixProperty<Integer> timerDelayInMilliseconds;  
//批处理过程中是否开启请求缓存,默认:开启  
private final HystrixProperty<Boolean> requestCacheEnabled;

/* defaults */
private static final Integer default_maxRequestsInBatch = Integer.MAX_VALUE;
private static final Integer default_timerDelayInMilliseconds = 10;
private static final Boolean default_requestCacheEnabled = true;
```

## 线程池(ThreadPool)配置
配置类：HystrixThreadPoolProperties

构造Command时通过Setter进行配置
```
/* defaults */
private Integer default_coreSize = 10; // size of thread pool
private Integer default_keepAliveTimeMinutes = 1; // minutes to keep a thread alive (though in practice this doesn't get used as by default we set a fixed size)
private Integer default_maxQueueSize = -1; // size of queue (this can't be dynamically changed so we use 'queueSizeRejectionThreshold' to artificially limit and reject)
                                           // -1 turns if off and makes us use SynchronousQueue
private Integer default_queueSizeRejectionThreshold = 5; // number of items in queue 
private Integer default_threadPoolRollingNumberStatisticalWindow = 10000; // milliseconds for rolling number
private Integer default_threadPoolRollingNumberStatisticalWindowBuckets = 10; // number of buckets in rolling number (10 1-second buckets)

private final HystrixProperty<Integer> corePoolSize;
private final HystrixProperty<Integer> keepAliveTime;
private final HystrixProperty<Integer> maxQueueSize;
private final HystrixProperty<Integer> queueSizeRejectionThreshold;
private final HystrixProperty<Integer> threadPoolRollingNumberStatisticalWindowInMilliseconds;
private final HystrixProperty<Integer> threadPoolRollingNumberStatisticalWindowBuckets;
```
