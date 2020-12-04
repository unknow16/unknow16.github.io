---
title: Dubbo-08-主要接口概念简介
date: 2018-05-16 13:13:36
tags: Dubbo
---

本篇分别介绍下Invoker、ProxyFactory、Protocol、Exporter的概念

## Invoker概念
* 一个可执行的对象，能够根据方法名称、参数得到相应的执行结果。
* 接口源码如下：
```
public interface Invoker<T> extends Node {

    // get service interface.
    Class<T> getInterface();

    // invoke方法，返回执行结果
    Result invoke(Invocation invocation) throws RpcException;

}

public interface Node {

    URL getUrl();
    
    boolean isAvailable();

    void destroy();
}
```
* 其中invocation包含了需要执行的方法、参数等信息，目前只有一个RpcInvocation实现，仅仅提供给Invoker执行时需要的参数。
* 接口源码如下：
```
public interface Invocation {

	String getMethodName();

	Class<?>[] getParameterTypes();

	Object[] getArguments();

	Map<String, String> getAttachments();
	
	String getAttachment(String key);
	
	String getAttachment(String key, String defaultValue);

    Invoker<?> getInvoker();
}
```

#### Invoker类型
这个可执行对象的执行过程分成三种类型：
1. 本地执行类的Invoker
2. 远程通信执行类的Invoker
3. 多个类型的Invoker聚合成的集群版的Invoker

以HelloService接口方法为例：

* 本地执行类的Invoker： server端，含有对应的HelloServiceImpl实现，要执行该接口方法，仅仅只需要通过反射执行HelloServiceImpl对应的方法即可
* 远程通信执行类的Invoker： client端，要想执行该接口方法，需要需要进行远程通信，发送要执行的参数信息给server端，server端利用上述本地执行的Invoker执 行相应的方法，然后将返回的结果发送给client端。这整个过程算是该类Invoker的典型的执行过程
* 集群版的Invoker：client端，拥有某个服务的多个Invoker，此时client端需要做的就是将这个多个Invoker聚 合成一个集群版的Invoker，client端使用的时候，仅仅通过集群版的Invoker来进行操作。集群版的Invoker会从众多的远程通信类型 的Invoker中选择一个来执行（从中加入负载均衡策略），还可以采用一些失败转移策略等

#### Invoker实现类继承图
![image](https://note.youdao.com/yws/api/personal/file/039051E2456F4938ACE10C4359AF8304?method=download&shareKey=c38eb18da431458b8d7299713facff5c)

## ProxyFactory概念

* 接口源码如下：
```
@SPI("javassist")
public interface ProxyFactory {

    // 客户端创建接口的代理类
    @Adaptive({Constants.PROXY_KEY})
    <T> T getProxy(Invoker<T> invoker) throws RpcException;

    // 服务端创建接口实现的invoker
    @Adaptive({Constants.PROXY_KEY})
    <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) throws RpcException;

}
```
* ProxyFactory的接口实现有JdkProxyFactory、JavassistProxyFactory，默认是JavassistProxyFactory 

#### 服务端使用
* 对于server端，主要调用getInvoker()方法，负责将服务如HelloServiceImpl统一进行包装成一个Invoker，这些Invoker通过反射来执行具体的HelloServiceImpl对象的方法。

#### 客户端使用
* 对于client端，主要调用getProxy()方法，以用Protocol.refer()的到的invoker为参数，返回一个要调用接口的代理对象。

## Protocol概念

* Protocol接口源码如下：
```
@SPI("dubbo")
public interface Protocol {
    
    int getDefaultPort();

    @Adaptive
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;

    @Adaptive
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;

    void destroy();
}
```
#### 服务端
* 针对服务端来说，Protocol要解决的问题就是：根据指定协议向外暴露HelloService，当客户端发起调用时，接收客户端传过来的Invocation对象，调用本地Invoker执行并返回结果。
* 所以Protocol加入了远程通信协议这块。

#### 客户端
* 根据注册中心或直连服务提供者配置，生成url，通过协议获取一个落地可执行的Invoker来执行远程方法调用。

## Exporter概念
* 负责维护Invoker的生命周期
* 接口源码如下：
```
public interface Exporter<T> {
    
    Invoker<T> getInvoker();
    
    void unexport();
}
```
