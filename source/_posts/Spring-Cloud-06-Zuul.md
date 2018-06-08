---
title: Spring Cloud-06-Zuul
date: 2018-01-18 12:54:44
tags: Spring Cloud
---

### Router and Filter: Zuul
### 基本使用
* spring-cloud-starter-zuul  #默认包含ribbon,hystrix, 但不包含discovery client,如果要基于serviceID路由，要引入spring-cloud-starter-eureka
* @EnableZuulProxy添加到启动类上，并将调用转发给相应服务，代理使用Ribbon找到要转发的实例。
* 转发通过RibbonRoutingFilter实现，在转发之前，zuul利用Hystrix将此次转发请求包装成一个HystrixCommond，正因为这样zuul具有了降级的功能，同时HystrixCommand具有超时时间，默认为1s。
* 而且Zuul默认采用的隔离级别是信号量。
* 避免了独立管理所有后端的CORS和身份验证问题。


```
1. 启动类配置
@EnableZuulProxy
@SpringCloudApplication
public class Application {
  
  public static void main(String[] args) {
    new SpringApplicationBuilder(Application.class).web(true).run(args);
  }
}

2. 配置文件：
spring:
  application:
    name: api-gateway

server:
  port: 1101

eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka.didispace.com/eureka/
```
由于Spring Cloud Zuul在整合了Eureka之后，具备默认的服务路由功能，即：当我们这里构建的api-gateway应用启动并注册到eureka之后，服务网关会发现上面我们启动的两个服务eureka-client和eureka-consumer，这时候Zuul就会创建两个路由规则。每个路由规则都包含两部分，一部分是外部请求的匹配规则，另一部分是路由的服务ID。针对当前示例的情况，Zuul会创建下面的两个路由规则：
* 转发到eureka-client服务的请求规则为：/eureka-client/**
* 转发到eureka-consumer服务的请求规则为：/eureka-consumer/**

### Zuul Http Client
* Dalston.SR4默认使用 Apache HTTP Client，替代了过时的Ribbon RestClient
* ribbon.restclient.enabled=true or ribbon.okhttp.enabled=true相应client

### 编写Filter
* Zuul允许开发者在API网关上通过定义过滤器来实现对请求的拦截与过滤，实现的方法非常简单，我们只需要继承ZuulFilter抽象类并实现它定义的四个抽象函数就可以完成对请求的拦截和过滤了。导包均为com.netflix.zuul.下的。
* 比如下面的代码，我们定义了一个简单的Zuul过滤器，它实现了在请求被路由之前检查HttpServletRequest中是否有accessToken参数，若有就进行路由，若没有就拒绝访问，返回401 Unauthorized错误。

```
public class AccessFilter extends ZuulFilter  {

    private static Logger log = LoggerFactory.getLogger(AccessFilter.class);

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();

      	log.info("send {} request to {}", request.getMethod(), request.getRequestURL().toString());

        Object accessToken = request.getParameter("accessToken");
        if(accessToken == null) {
            log.warn("access token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            return null;
        }
        log.info("access token ok");
        return null;
    }

}
```
在上面实现的过滤器代码中，我们通过继承ZuulFilter抽象类并重写了下面的四个方法来实现自定义的过滤器。这四个方法分别定义了：
* filterType：过滤器的类型，它决定过滤器在请求的哪个生命周期中执行。这里定义为pre，代表会在请求被路由之前执行。
* filterOrder：过滤器的执行顺序。当请求在一个阶段中存在多个过滤器时，需要根据该方法返回的值来依次执行。
* shouldFilter：判断该过滤器是否需要被执行。这里我们直接返回了true，因此该过滤器对所有请求都会生效。实际运用中我们可以利用该函数来指定过滤器的有效范围。
* run：过滤器的具体逻辑。这里我们通过ctx.setSendZuulResponse(false)令zuul过滤该请求，不对其进行路由，然后通过ctx.setResponseStatusCode(401)设置了其返回的错误码，当然我们也可以进一步优化我们的返回，比如，通过ctx.setResponseBody(body)对返回body内容进行编辑等。

```
@EnableZuulProxy
@SpringCloudApplication
public class Application {

	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}

	@Bean
	public AccessFilter accessFilter() {
		return new AccessFilter();
	}
}
```
在对api-gateway服务完成了上面的改造之后，我们可以重新启动它，并发起下面的请求，对上面定义的过滤器做一个验证：
* http://localhost:1101/api-a/hello：返回401错误
* http://localhost:1101/api-a/hello?accessToken=token：正确路由到hello-service的/hello接口，并返回Hello World

