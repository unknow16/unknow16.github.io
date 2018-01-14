---
title: 'Circuit Breaker: Hystrix'
date: 2018-01-14 14:42:35
tags: spring cloud
---

### 基本使用
1. spring-cloud-starter-hystrix
2. 在应用主类中使用@EnableCircuitBreaker或@EnableHystrix注解开启Hystrix的使用
3. 最后，在为具体执行逻辑的函数上增加@HystrixCommand注解的fallbackMethod属性来指定服务降级方法

### 服务降级
* 当调用依赖服务不可用时，通过调用指定的fallback方法返回
* @HystrixCommand注解的fallbackMethod属性指定字符串降级方法名

### 服务熔断
* 在10秒时间窗内达到20次请求，其中50%请求调用失败，断路器打开，服务被熔断
* 在该服务熔断后，会直接调用fallback方法，而不会再去调用请求方法

### 线程信号隔离
* 默认在独立的线程池中执行
* 可设置信号量策略，默认为10

```
@HystrixCommand(fallbackMethod = "stubMyService",
    commandProperties = {
      @HystrixProperty(name="execution.isolation.strategy", value="SEMAPHORE")
    }
)
```

### 命令名称、分组及线程池划分
* 默认情况下，Hystrix会让相同组名的命令使用使用一个线程池，所以我们需要在创建Hystrix命令时为其指定命令组名以实现线程池的划分。
* 除了通过命令组来划分线程池外，还可通过HystrixThreadPoolKey来实现更细粒度的线程池划分。尽量使用此方式。
* 使用@HystrixCommond注解的commondKey, groupKey,threadPoolKey属性分别设置命令名称、分组、线程池划分等。

```
@HystrixCommand(
        fallbackMethod = "defaultAction", 
        commandKey = "getUserById", 
        groupKey = "UserGroup", 
        threadPoolKey = "getUserByIdThread")
@GetMapping("getUserById")
public String getUserById() {
    String url = "http://eureka-client-provider/dc";

    //使用restTemplate请求数据
    return restTemplate.getForObject(url, String.class);
}

public String defaultAction() {
    return "fallback default";
}
```

### zuul中使用ribbon和hystrix
* 如果将zuul.ribbonIsolationStrategy更改为THREAD，则Hystrix的线程隔离策略将用于所有路由。
* 在这种情况下，HystrixThreadPoolKey默认设置为“RibbonCommand”。这意味着所有路由的HystrixCommands将在相同的Hystrix线程池中执行。
* 此行为可以使用以下配置进行更改,这将导致HystrixCommands在每个路由的Hystrix线程池中执行。

```
zuul:
  threadPool:
    useSeparateThreadPools: true
```
* 在这种情况下，默认的HystrixThreadPoolKey与每个路由的服务ID相同。

* 要为HystrixThreadPoolKey添加前缀，请将zuul.threadPool.threadPoolKeyPrefix设置为您要添加的值。 例如：

```
zuul:
  threadPool:
    useSeparateThreadPools: true
    threadPoolKeyPrefix: zuulgw
```


### 请求缓存


### 请求合并

### 服务监控


##### Hystrix Dashboard
* 我们提到断路器是根据一段时间窗内的请求情况来判断并操作断路器的打开和关闭状态的。而这些请求情况的指标信息都是HystrixCommand和HystrixObservableCommand实例在执行过程中记录的重要度量信息，它们除了Hystrix断路器实现中使用之外，对于系统运维也有非常大的帮助。
* 这些指标信息会以“滚动时间窗”与“桶”结合的方式进行汇总，并在内存中驻留一段时间，以供内部或外部进行查询使用，Hystrix Dashboard就是这些指标内容的消费者之一。
* [翟老师博客Dashboard使用](http://blog.didispace.com/spring-cloud-starter-dalston-5-1/)

##### Hystrix Turbine
* [翟老师博客Turbine监控数据聚合](http://blog.didispace.com/spring-cloud-starter-dalston-5-2/)
##### Turbine Stream