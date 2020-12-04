---
title: Dubbo-12-服务引用解析2
date: 2018-05-16 13:14:37
tags: Dubbo
---

### 服务引用集群容错实现
Protocol要解决的问题就是：根据url中指定的协议（没有指定的话使用默认的dubbo协议）对 外公布这个HelloService服务，当客户端根据协议调用这个服务时，将客户端传递过来的Invocation参数交给服务器端的Invoker来 执行。所以Protocol加入了远程通信协议的这一块，根据客户端的请求来获取参数Invocation invocation。

服务引用的第一个过程就是使用协议Protocol根据url和服务接口来引用服务，创建出一个Invoker对象，如下：
```
Invoker invoker = refprotocol.refer(interfaceClass, url);
```
需要根据服务器开放的协议（服务器端在注册中心注册的url地址中含有该信息）来创建相应的协议的Invoker对象，如

- DubboInvoker
- InjvmInvoker
- ThriftInvoker

如服务器端在注册中心中注册的url地址为：


```
dubbo://192.168.1.104:20880/com.demo.dubbo.service.HelloService?
anyhost=true&
application=helloService-app&dubbo=2.5.3&
interface=com.demo.dubbo.service.HelloService&
methods=hello&
pid=3904&
side=provider&
timestamp=1444003718316
```
会看到上述服务是以dubbo协议注册的，所以这里产生的Invoker就是DubboInvoker。

refer(interfaceClass, url)的过程即根据url的配置信息来最终选择的Protocol实现，默认实现是"dubbo"的扩展实现即DubboProtocol，然后再对 DubboProtocol进行依赖注入，进行wrap包装。

在返回DubboProtocol之前，经过了ProtocolFilterWrapper、ProtocolListenerWrapper、RegistryProtocol的包装，使用装饰器模式，类似AOP的功能。

#### refprotocol的refer()干了哪些事呢？
1. 会先经过RegistryProtocol(先暂时忽略ProtocolFilterWrapper、ProtocolListenerWrapper)

2. 根据注册中心的registryUrl获取注册服务Registry，将自身的consumer信息注册到注册中心上

```
// 先根据客户端的注册中心配置找到对应注册服务 
// 注意registryFactory使用了SPI机制
Registry registry = registryFactory.getRegistry(url);

//使用注册服务将客户端的信息注册到注册中心上 
registry.register(subscribeUrl.addParameters(Constants.CATEGORY_KEY, Constants.CONSUMERS_CATEGORY,Constants.CHECK_KEY, String.valueOf(false)));

// 上述subscribeUrl地址如下：
consumer://192.168.1.104/com.demo.dubbo.service.HelloService?
   application=consumer-of-helloService&
   dubbo=2.5.3&
   interface=com.demo.dubbo.service.HelloService&
   methods=hello&
   pid=6444&
   side=consumer&
   timestamp=1444606047076
// 该url表述了自己是consumer，同时自己的ip地址是192.168.1.104
// 引用的服务是com.demo.dubbo.service.HelloService，以及注册时间等等
```
3. 创建一个RegistryDirectory，从注册中心中订阅自己引用的服务，将订阅到的url在RegistryDirectory内部转换成Invoker

```
RegistryDirectory<T> directory = new RegistryDirectory<T>(type, url); 
directory.setRegistry(registry); 
directory.setProtocol(protocol); 
directory.subscribe(subscribeUrl.addParameter(Constants.CATEGORY_KEY,Constants.PROVIDERS_CATEGORY + "," + Constants.CONFIGURATORS_CATEGORY + "," + Constants.ROUTERS_CATEGORY));
```
Directory接口如下：

```
public interface Directory<T> extends Node {

    Class<T> getInterface();

    // list invokers.
    List<Invoker<T>> list(Invocation invocation) throws RpcException;

}
```

上述RegistryDirectory是Directory的实现，Directory代表多个Invoker，可以把它看成List类型的Invoker，但与List不同的是，它的值可能是动态变化的，比如注册中心推送变更。

RegistryDirectory内部含有两者重要属性：

- 注册中心服务: Registry registry
- 协议服务：Protocol protocol

它会利用注册中心服务Registry来获取最新的服务器端注册的url地址，然后再利用协议服务Protocol将这些url地址转换成一个具有远程通信功能的Invoker对象，如DubboInvoker

4. 然后使用Cluster对象将上述Directory中的多个Invoker对象（此时还没有真正创建出来，异步订阅，订阅成功之后，回调时才会创建出Invoker）聚合成一个集群版的Invoker对象。

```
Cluster cluster = ExtensionLoader.getExtensionLoader(Cluster.class).getAdaptiveExtension();
cluster.join(directory)
```
Cluster接口如下：

```
@SPI(FailoverCluster.NAME)
public interface Cluster {
    
    // Merge the directory invokers to a virtual invoker.
    @Adaptive
    <T> Invoker<T> join(Directory<T> directory) throws RpcException;

}
```

Cluster只有一个功能就是把上述Directory（相当于一个List类型的Invoker）聚合成一个Invoker，同时也可以对List进行过滤处理（这些过滤操作也是配置在注册中心的）等实现路由的功能，主要是对用户进行透明。

默认采用的是FailoverCluster：失败转移，当出现失败，重试其它服务器，通常用于读操作，但重试会带来更长延迟。其实现如下：

```
public class FailoverCluster implements Cluster {

    public final static String NAME = "failover";

    public <T> Invoker<T> join(Directory<T> directory) throws RpcException {
        return new FailoverClusterInvoker<T>(directory);
    }
}
```
仅仅是创建了一个FailoverClusterInvoker，具体的逻辑留在调用的时候即调用该Invoker的 invoke(final Invocation invocation)方法时来进行处理。其中又会涉及到另一个接口LoadBalance（从众多的Invoker中挑选出一个Invoker来执行此 次调用任务），接口如下：

```
@SPI(RandomLoadBalance.NAME)
public interface LoadBalance {
    
    // select one invoker in list.
    @Adaptive("loadbalance")
    <T> Invoker<T> select(List<Invoker<T>> invokers, URL url, Invocation invocation) throws RpcException;

}
```
默认采用的是随机策略，具体的内容就请各自详细去研究。

### DubboInvoker具体实现
服务引用的第二个过程是，当我们调用创建出来的代理对象如HelloService helloService的方法时，会执行InvokerInvocationHandler中的逻辑：

```
public class InvokerInvocationHandler implements InvocationHandler {

    private final Invoker<?> invoker;

    public InvokerInvocationHandler(Invoker<?> handler) {
        this.invoker = handler;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        Class<?>[] parameterTypes = method.getParameterTypes();
        if (method.getDeclaringClass() == Object.class) {
            return method.invoke(invoker, args);
        }
        if ("toString".equals(methodName) && parameterTypes.length == 0) {
            return invoker.toString();
        }
        if ("hashCode".equals(methodName) && parameterTypes.length == 0) {
            return invoker.hashCode();
        }
        if ("equals".equals(methodName) && parameterTypes.length == 1) {
            return invoker.equals(args[0]);
        }
        return invoker.invoke(new RpcInvocation(method, args)).recreate();
    }

}
```
可以看到还是交给目标对象Invoker来执行

---

前面说过有三种类型的Invoker:
* 服务端本地执行的invoker
* 远程通信执行的invoker
* 集群版的远程执行invoker

对于服务引用的客户端来说是后两者，我们先来看下非集群版的远程通信执行invoker，以DubboInvoker为例，执行该invoker时执行其invoke()方法，如下：
```
// 继承关系如下：
public class DubboInvoker<T> extends AbstractInvoker<T> 
public abstract class AbstractInvoker<T> implements Invoker<T>

// 1. 先执行抽象父类AbstractInvoker的invoke,接收invocation参数
public Result invoke(Invocation inv) throws RpcException {
    if (destroyed.get()) {
        throw new RpcException("Rpc invoker for service " + this + " on consumer " + NetUtils.getLocalHost()
                + " use dubbo version " + Version.getVersion()
                + " is DESTROYED, can not be invoked any more!");
    }
    RpcInvocation invocation = (RpcInvocation) inv;
    invocation.setInvoker(this);
    if (attachment != null && attachment.size() > 0) {
        invocation.addAttachmentsIfAbsent(attachment);
    }
    Map<String, String> context = RpcContext.getContext().getAttachments();
    if (context != null) {
        invocation.addAttachmentsIfAbsent(context);
    }
    if (getUrl().getMethodParameter(invocation.getMethodName(), Constants.ASYNC_KEY, false)) {
        invocation.setAttachment(Constants.ASYNC_KEY, Boolean.TRUE.toString());
    }
    RpcUtils.attachInvocationIdIfAsync(getUrl(), invocation);

    // 2. 抽象方法，子类DubboInvoker中实现
    return doInvoke(invocation);

}

// 3. DubboInvoker中具体执行逻辑
protected Result doInvoke(final Invocation invocation) throws Throwable {
    RpcInvocation inv = (RpcInvocation) invocation;
    final String methodName = RpcUtils.getMethodName(invocation);
    inv.setAttachment(Constants.PATH_KEY, getUrl().getPath());
    inv.setAttachment(Constants.VERSION_KEY, version);

    // 4. remoting层--交换层通信对象
    ExchangeClient currentClient;
    if (clients.length == 1) {
        currentClient = clients[0];
    } else {
        currentClient = clients[index.getAndIncrement() % clients.length];
    }
    try {
        boolean isAsync = RpcUtils.isAsync(getUrl(), invocation);
        boolean isOneway = RpcUtils.isOneway(getUrl(), invocation);
        int timeout = getUrl().getMethodParameter(methodName, Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
        if (isOneway) {
            boolean isSent = getUrl().getMethodParameter(methodName, Constants.SENT_KEY, false);
            currentClient.send(inv, isSent);
            RpcContext.getContext().setFuture(null);
            return new RpcResult();
        } else if (isAsync) {
            ResponseFuture future = currentClient.request(inv, timeout);
            RpcContext.getContext().setFuture(new FutureAdapter<Object>(future));
            return new RpcResult();
        } else {
            RpcContext.getContext().setFuture(null);
            return (Result) currentClient.request(inv, timeout).get();
        }
    } catch (TimeoutException e) {
        throw new RpcException(RpcException.TIMEOUT_EXCEPTION, "Invoke remote method timeout. method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
    } catch (RemotingException e) {
        throw new RpcException(RpcException.NETWORK_EXCEPTION, "Failed to invoke remote method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}
```
* 执行逻辑

将通过远程通信将Invocation信息传递给服务器端，服务器端接收到该Invocation信息后，找到对应的本地Invoker，然后通过反射执行相应的方法，将方法的返回值再通过远程通信将结果传递给客户端。

* ExchangeClient:  remoting层--交换层通信对象
* 这里分成3种情况：

1. 执行的方法不需要返回值：直接使用ExchangeClient的send方法
1. 执行的方法的结果需要异步返回：使用ExchangeClient的request方法，返回一个ResponseFuture，通过ThreadLocal方式与当前线程绑定，未等服务器端响应结果就直接返回
1. 执行的方法的结果需要同步返回：使用ExchangeClient的request方法，返回一个ResponseFuture，一直阻塞到服务器端返回响应结果