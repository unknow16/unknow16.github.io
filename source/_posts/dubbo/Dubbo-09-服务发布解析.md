---
title: Dubbo-09-服务发布解析
date: 2018-05-16 13:13:51
tags: Dubbo
---

### 服务发布入口

Dubbo利用DubboNamespaceHandler在Spring中对自己的xml配置命名空间进行解析，其中关于服务发布的配置解析到了ServiceConfig中，为了在Spring的IOC容器启动时，发步dubbo服务，定义了ServiceBean继承ServiceConfig。

* ServiceBean继承图

![image](https://note.youdao.com/yws/api/personal/file/E78EA26634C94167B6592DE18168D89E?method=download&shareKey=adaf150b6f37228b601e34dd63eb0223)

从继承图可见，ServiceBean实现了ApplicationListener接口，源码如下：
```
public void onApplicationEvent(ApplicationEvent event) {
    if (ContextRefreshedEvent.class.getName().equals(event.getClass().getName())) {
    	if (isDelay() && ! isExported() && ! isUnexported()) {
            if (logger.isInfoEnabled()) {
                logger.info("The service ready on spring started. service: " + getInterface());
            }
            
            // ServiceConfig父类中的方法
            // 服务发布的入口
            export();
        }
    }
}
```
当监听到Spring的ContextRefreshedEvent事件，即IOC上下文刷新完毕时，开始调用其父类ServiceConfig的export()方法发布服务。

### 服务提供者配置案例

```
//结合zookeeper作为注册中心

<!-- 提供方应用信息，用于计算依赖关系 -->
<dubbo:application name="provider"/>

<!-- 使用multicast广播注册中心暴露服务地址 -->
<dubbo:registry address="zookeeper://127.0.0.1:2181/"/>

<!-- 用dubbo协议在20880端口暴露服务 -->
<dubbo:protocol name="dubbo" port="20880"/>

<!-- 声明需要暴露的服务接口 -->
<dubbo:service interface="com.netease.dubbo.service.EchoService" ref="echoService"/>

<!-- 和本地bean一样实现服务 -->
<bean id="echoService" class="com.netease.dubbo.service.EchoServiceImpl"/>
```

### 服务发布过程

* 一个服务可以注册到多个注册中心，也可以开启多个服务协议监听。

1. 根据注册中心配置，将注册中心信息聚合在一个URL对象中，registryURLs内容如下：
```
[registry://127.0.0.1:2181/com.alibaba.dubbo.registry.RegistryService?application=provider&dubbo=2.5.3&pid=11388&registry=zookeeper&timestamp=1526282378060]
```
2. 针对<dubbo:protocol>配置采用dubbo协议监听在20881端口，对外提供HelloService接口服务，将注册的协议信息也转化成一个URL对象如下：
```
dubbo://192.168.1.104:20880/com.demo.dubbo.service.HelloService?anyhost=true&application=helloService-app&dubbo=2.0.13&interface=com.demo.dubbo.service.HelloService&methods=hello&prompt=dubbo&revision=

```
3. 根据注册中心配置URL和协议信息URL的组合，依次来进行服务的发布。


* 伪代码如下：

```
// 获取注册中心地址配置，格式如上文1
List<URL> registryURLs = loadRegistries();

// protocols是某个接口服务要用不同协议暴露，格式如上文2
for (ProtocolConfig protocolConfig : protocols) {
    
    // 根据每一个协议配置构建一个URL
    URL url = new URL(name, host, port, (contextPath == null || contextPath.length() == 0 ? "" : contextPath + "/") + path, map);  
    
    // 循环向不同注册中心注册服务
    for (URL registryURL : registryURLs) {
        
        // 服务提供者url, 格式如上文2
        String providerURL = url.toFullString();
        
        // 将HelloServiceImpl封装成一个Invoker
        Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(RpcConstants.EXPORT_KEY, providerURL));
        
        // 将Invoker导出成一个Exporter, 其中建立端口监听服务
        // 此处protocol采用SPI，会根据url的协议头不同，protocol代理内部会调用不同实现
        // 如registry对应RegistryProtocol,duboo对应DubboProtocol
        Exporter<?> exporter = protocol.export(invoker);
    }
}
```

### 服务发布总结

1. 通过ProxyFactory的getInvoker()将HelloServiceImpl封装成一个Invoker
2. 使用Protocol的export()将Invoker导出成一个Exporter

以上两步具体过程，请看解析2