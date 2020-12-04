---
title: Dubbo-11-服务引用解析
date: 2018-05-16 13:14:30
tags: Dubbo
---

### 服务引用入口
Dubbo利用DubboNamespaceHandler在Spring中对自己的xml配置的命名空间进行解析，其中将服务引用相关配置解析到ReferenceConfig中，为了使用Spring的IOC容器，Dubbo又定义了一个ReferenceBean继承自ReferenceConfig。

ReferenceBean实现了Spring的FactoryBean接口，Spring会调用其getObject()方法将返回的bean注入IOC容器中，如下：
```
    public Object getObject() throws Exception {
        // get()为其父类ReferenceConfig中的方法
        return get();
    }
    
    // ReferenceConfig中的get()方法实现
    public synchronized T get() {
        if (destroyed){
            throw new IllegalStateException("Already destroyed!");
        }
    	if (ref == null) {
    		// 服务引用初始化入口
    		init();
    	}
    	return ref;
    }
```
其中ref为要调用接口的代理类，init()方法中对创建代理并为ref赋值返回。

### 服务消费者配置案例

```
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer"/>

    <!-- 使用zk注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181/"/>

    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="echoService" interface="com.netease.dubbo.service.EchoService" check="false"> </dubbo:reference>
```

### init()方法伪代码

```
// 通过注册中心

// 1. 获取所有注册中心URL，并添加refer_key，即要引用的服务的配置信息
List<URL> us = loadRegistries(false);
if (us != null && us.size() > 0) {
	for (URL u : us) {
	    // map为要拼接到URL的公共参数
	    // urls存放拼接refer_key后的注册中心url
	    urls.add(u.addParameterAndEncoded(Constants.REFER_KEY, StringUtils.toQueryString(map)));
    }
}
            	
// 2. 用refprotocol从注册中心获取获取引用服务的invoker            	
invoker = refprotocol.refer(interfaceClass, urls.get(0));

// 3. 根据invoker创建要调用接口服务的代理
ref = (T) proxyFactory.getProxy(invoker);
```

### 服务消费者代码中使用
```
// 从Spring的IOC容器中自动注入一个代理实现
// 此处EchoService为ReferenceBean的getObject()方法返回的代理
@Autowired 
private EchoService echoService;
```
* 此处代理创建有两种方式：javassist和jdk，下文以jdk代理为例。

使用jdk创建代理时，需要传入一个java.lang.reflect.InvocationHandler的实现类，dubbo中为InvokerInvocationHandler，创建代理时将invoker传递进了InvokerInvocationHandler。

通过该代理对象调用某方法时，都会执行InvocationHandler的invoke()方法，该方法中通过调用invoker对象的invoke()执行rpc远程调用，调用时将调用参数封装成RpcInvocation对象作为Invoker对象的invoke()方法参数，执行完毕返回结果。

### 服务引用总结
1. 从注册中心引用服务，通过Protocol的refer()创建出Invoker对象
2. 使用ProxyFactory的getProxy()创建出一个接口的代理对象，该代理对象的方法的执行都交给上述Invoker来执行

以上两步具体过程，请看解析2