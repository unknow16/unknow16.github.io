---
title: Spring Cloud-03-Feign
date: 2018-01-15 01:04:49
tags: Spring Cloud
---

### Declarative REST Client: Feign
### 基本使用
* 启动类添加@EnableFeignClients注解开启扫描Spring Cloud Feign客户端的功能
* 创建一个Feign的客户端接口定义。使用@FeignClient注解中的value或name属性来指定这个接口所要调用的服务名称（serviceId属性过时）
* 接口中定义的各个函数使用SpringMVC的注解就可以来绑定服务提供方的REST接口

```
@FeignClient("stores")
public interface StoreClient {
    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    List<Store> getStores();

    @RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
    Store update(@PathVariable("storeId") Long storeId, Store store);
}
```
* 以上Feign会通过Ribbon使用stores作为client name去创建一个Load Balancer负载均衡调用，它会去发现“store”服务的物理地址，如果结合eureka,它将会从注册中心自动获取，否则你要指定一个servers列表。
* 除了指定服务名外，还可指定@FeignClient的url属性，可以是绝对的或hostname, e.g. http://localhost:5001/或域名
* 在application context中，这个bean的名称中一个接口的全限定名。也能使用qualifier属性去指定你自己的别名。

---
* Feign默认使用了Ribbon进行客户端侧负载均衡
* 另外，Feign还整合的Hystrix来实现服务的容错保护，在Dalston版本中，Feign的Hystrix默认是关闭的,需添加hystrix-starter到classpath,并设置feign.hystrix.enabled=true
* 如果你需要使用ThreadLoca在你的RequestInterceptor中绑定变量，你需要设置hystrix的线程隔离策略为信号量，或失效在feign中的Hystrix
 
```
# To disable Hystrix in Feign
feign:
  hystrix:
    enabled: false

# To set thread isolation to SEMAPHORE
hystrix:
  command:
    default:
      execution:
        isolation:
          strategy: SEMAPHORE
```


### 自定义FeiginClient配置
* 类似于RibbonClient的自定义配置，默认配置类FeignClientsConfiguration
* 支持对单个服务指定@Configuration配置类为@FeignClient的属性configuration
* 支持对所有服务的默认配置自定义，使用@EnableFeignClients的属性defaultConfiguration
* 支持使用属性文件自定义

```
对名为feignName的Feign Client的自定义配置

feign:
  client:
    config:
      feignName:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: full
        errorDecoder: com.example.SimpleErrorDecoder
        retryer: com.example.SimpleRetryer
        requestInterceptors:
          - com.example.FooRequestInterceptor
          - com.example.BarRequestInterceptor
        decode404: false
        

对所有Feign Client的配置

feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
```

### 结合Hystrix
* 开启全局feign.hystrix.enabled=true
* 在Dalston版本中，Feign的Hystrix默认是关闭的
```
1. 定义fallback类，实现feignClient的接口

@Component //记得加该注解
public class HystrixClientFallback implements DcClient {

    @Override
    public String consumer() {
        return "default-provider";
    }
}

2. @FeignClient中fallback属性设置1中定义的类

@FeignClient(name = "eureka-client-provider", fallback = HystrixClientFallback.class)
public interface DcClient {

    @GetMapping("/dc")
    String consumer();
}
```

### 使用fallbackFactory属性打印fallback异常
* fallback和fallbackFactory不能同时配置使用
```
1. 定义fallbackFactory类，该类实现FallbakFactory<DcClient>接口，范型为FeignClient接口名

@Component
public class HystrixClientFallbackFactory implements FallbackFactory<DcClient> {
    @Override
    public DcClient create(Throwable throwable) {
        return new DcClient() {
            @Override
            public String consumer() {
                return "fallback cause: == " + throwable.getMessage();
            }
        };
    }
}

2. 

@FeignClient(name = "eureka-client-provider", fallbackFactory = HystrixClientFallbackFactory.class)
```

### 禁用单个FeignClient的Hystrix支持
* 自定义@FeignClient的配置，默认Feign.Builder 为 HystrixFeign.Builder
* 为@FeignClient的configuration属性，指定FooConfiguration
* 使用Feign.builder替代默认的HystrixFeign.Builder
```
@Configuration
public class FooConfiguration {
    @Bean
	@Scope("prototype")
	public Feign.Builder feignBuilder() {
		return Feign.builder();
	}
}
```

### 访问设置基础认证的API
* 如访问认证的eureka信息，需自定义FeignClient配置

```
1. 自定义配置认证用户名和密码

@Configuration
public class CustomFeignClientConfiguration {

    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new BasicAuthRequestInterceptor("user", "password");
    }
}

2. 将该配置设置给@FeignClient的configuration属性

@FeignClient(name = "eureka-client-provider", url = "http://localhost:1001/", configuration = CustomFeignClientConfiguration.class)
public interface DcClient2 {

    @RequestMapping("/eureka/apps/{serviceId}")
    String getInfoFromEureka(@PathVariable("serviceId") String serviceId);
}
```

### Feign日志级别
* 对每一个Feign client创建一个Logger。
* 默认logger的名称是feign接口的全类名。
* Feign日志记录只响应DEBUG级别，配置级别
* 
* 以下四个类型是告诉feign怎样记录哪些日志信息。
* None : no logging 默认
* Basic： 仅记录请求方法和url和响应状态码和执行时间
* Headers： 记录基础信息外，附加请求和响应头
* full: 记录请求和响应的头，体，元数据
* 如下，设置Logger.Level为full
```
1. 设置日志级别，目前只响应debug级别

logging.level.com.fuyi.feign.DcClient: DEBUG

2. 自定义配置显示哪些信息，默认NONE，都不显示
    配置给@FeignClient的configuration属性

@Configuration
public class FooConfiguration {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```
