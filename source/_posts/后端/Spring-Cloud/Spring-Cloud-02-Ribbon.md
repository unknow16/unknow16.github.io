---
title: Spring Cloud-02-Ribbon
date: 2018-01-14 11:45:39
tags: Spring Cloud
---

### Client Side Load Balancer: Ribbon
### 基本使用
* 被服务消费者使用，负载均衡的调用服务提供者
* Zuul基于Ribbon请求转发
* Fegin基于Ribbon调用

```
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
	return new RestTemplate();
}

之后在使用restTemplate.getForObject("http://eureka-client-provider/dc", String.class);时
会采用默认配置负载均衡调用该服务的多个实例，默认轮询调用每个服务实例
```

### 主要类
* RibbonClientConfiguration： 默认的RibbonClient配置类
* IRule：为LoadBalancer定义的规则接口，作为负载均衡的策略，常用有轮询，随机，基于时间响应等，可查看它的实现类列表,如：RandomRule，RoundRobinRule
* IPing: 判断server是否up, 默认实现DummyPing，与eureka结合时，为NIWSDiscoveryPing,代理eureka去决定一个server是否up，保持心跳，维护客户端侧可用的服务器清单


```
Spring Cloud Netflix provides the following beans by default for ribbon (BeanType beanName: ClassName):

IClientConfig ribbonClientConfig: DefaultClientConfigImpl
IRule ribbonRule: ZoneAvoidanceRule
IPing ribbonPing: DummyPing
ServerList<Server> ribbonServerList: ConfigurationBasedServerList
ServerListFilter<Server> ribbonServerListFilter: ZonePreferenceServerListFilter
ILoadBalancer ribbonLoadBalancer: ZoneAwareLoadBalancer
ServerListUpdater ribbonServerListUpdater: PollingServerListUpdater
```


### 自定义对某个服务的Ribbon Client配置

```
1. 其中模仿RibbonClientConfiguration类覆盖相应的

@Configuration
public class CustomRibbonClientConfiguration {

    // 配置负载选取规则，随机选择一个服务调用
    @Bean
    public IRule ribbonRule() {
        return new RandomRule();
    }
}
```

```
2. 启动类上配置如下
   对eureka-client-provider服务使用1步中自定义的配置类CustomRibbonClientConfiguration

@RibbonClient(name = "eureka-client-provider", configuration = CustomRibbonClientConfiguration.class)
```
* 默认的均衡选取策略为轮询
* 此时Ribbon策略配置由RibbonClientConfiguration和CustomRibbonClientConfiguration共同组成

```
Note: 
    尽量避免1中的配置类被启动类中的SpringBootApplication注解中的ComponentScan注解扫描到,如果自定义RibbonClient被主类扫描，该配置将在parent context，将对所有的服务的调用生效该配置。
Solve:
    1. 避免1中的配置类和启动类在一个包下，即启动类不会扫描到该配置类
    2. 如果在启动类的扫描范围内，将该配置类从@ComponentScan中排除，实现如下
```

* 配置类在启动类扫描范围内如下解决方案
```
1. 声明一个注解

/**
 * 添加该注解的类将从主类扫描中排除
 * Created by fuyi on 2018/1/13.
 */
public @interface ExculdeFromComponentScan {
}

2. 在启动类上添加如下
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION, value = ExculdeFromComponentScan.class)})

3. 在需要排除的类上添加@ExculdeFromComponentScan注解
```
* 可使用如下代码测试负载均衡策略

```
@RestController
public class DcController {

    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private LoadBalancerClient loadBalancerClient;

    @GetMapping("/consumer")
    public String dc() {

        String url = "http://eureka-client-provider/dc";

        //使用restTemplate请求数据
        return restTemplate.getForObject(url, String.class);
    }


    @GetMapping("/loadBal")
    public void loadBal() {
        ServiceInstance serviceInstance = loadBalancerClient.choose("eureka-client-provider");
        System.out.println("serviceId : " + serviceInstance.getServiceId() + "，port : " + serviceInstance.getPort() + "，Uri : " + serviceInstance.getUri());

        ServiceInstance serviceInstance2 = loadBalancerClient.choose("eureka-client-provider2");
        System.out.println("serviceId : " + serviceInstance2.getServiceId() + "，port : " + serviceInstance2.getPort() + "，Uri : " + serviceInstance2.getUri());

    }
}
```

### 自定义对所有Ribbon Client的默认配置
* 在启动类上使用RibbonClients，为它的defaultConfiguration属性设置一个自定义的配置类

```
@RibbonClients(defaultConfiguration = CustomRibbonClientConfiguration.class)
```


### 使用配置文件Customizing the Ribbon Client
* 对eureka-client-provider服务均衡规则采用RandomRule,如下配置

```
eureka-client-provider:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

### 各种配置优先级
> 官方文档只说明了  配置文件 > @RibbonClient自定义 > 默认配置， 经测试@RibbonClients为最高优先级配置，对所有Ribbon Client的配置

### Ribbon with Eureka
* spring-cloud-starter-eureka包含spring-cloud-starter-ribbon

### Ribbon without Eureka

* 不结合eureka 使用时，服务提供者的地址列表需自己配置，如下
```
stores:
  ribbon:
    listOfServers: example.com,google.com
```

* eureka在classpath下，但不想使用，可配置如下禁用

```
ribbon:
  eureka:
   enabled: false
```

### Ribbon懒加载(Dalston.SR5文档新增)
* 每个ribbon client默认有相应的子application context被spring cloud维护的
* 默认被第一次请求时才被load
* 可配置如下，使该context在startup时被load

```
ribbon:
  eager-load:
    enabled: true
    clients: eureka-client-provider2,eureka-client-provider
```

### 配置属性
* 全局配置

```
请求创建连接的超时时间
ribbon.ConnectTimeout=60000
 
请求处理的超时时间
ribbon.ReadTimeout=60000
```


---
* 指定客户端配置方式

```
<client>.ribbon.<key>=<value>,该client可采用服务名

eureka-client-provider:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule # 配置均衡策略为随机选取一个实例访问
    listOfServers: http://localhost:2001 #手动维护该服务提供者的实例清单
    
spring.cloud.loadbalancer.retry.enabled=true # 开启重试机制

hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 10000 #断路器的超时时间要大于Ribbon重试时间，不然不会触发重试

eureka-client-provider.ribbon.ConnectionTimeout=250
eureka-client-provider.ribbon.ReadTimeout=1000
eureka-client-provider.ribbon.OkToRetryOnAllOperations=true #对所有操作请求都进行重试
eureka-client-provider.ribbon.MaxAutoRetriesNextServer=2 #切换实例的重试次数
eureka-client-provider.ribbon.MaxAutoRetries=1 #对当前实例的重试次数

如果配置，当请求故障时，会在次尝试访问当前实例一次（MaxAutoRetries决定），如果不行，换一个实例访问，如果还不行，再换一次（MaxAutoRetriesNextServer决定），如果依然不行，返回失败信息。


```

### 结合Hystrix的超时设置
* Hystrix的超时时间要大于ribbon.connectionTimeout乘以重试次数
* 如：ribbon.connectionTimeout=1，重试次数3，则Hystrix的超时设置要大于3秒



