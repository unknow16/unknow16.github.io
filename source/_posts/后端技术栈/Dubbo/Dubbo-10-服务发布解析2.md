---
title: Dubbo-10-服务发布解析2
date: 2018-05-16 13:14:23
tags: Dubbo
---

### 服务端中接口实现到Invoker的转换源码解析
前面说过有三种类型的Invoker:

- 服务端本地执行的invoker
- 远程通信执行的invoker
- 集群版的远程执行invoker

服务端发布服务的第一个过程就是将接口实现转换成Invoker,此处的invoker就是服务端本地执行的invoker，即AbstractProxyInvoker抽象类。

```
Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, url);
```

proxyFactory采用SPI机制，此处为扩展代理类，以下以jdk代理为例说明转换过程，所以代理内部采用JdkProxyFactory实现。

```
// 抽象父类AbstractProxyFactory中无相关实现代码
// 具体实现为JdkProxyFactory类的getInvoker()方法中
public class JdkProxyFactory extends AbstractProxyFactory {

    @SuppressWarnings("unchecked")
    public <T> T getProxy(Invoker<T> invoker, Class<?>[] interfaces) {
        return (T) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), interfaces, new InvokerInvocationHandler(invoker));
    }

    public <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) {
        // 直接new了一个AbstractProxyInvoker返回
        return new AbstractProxyInvoker<T>(proxy, type, url) {
            @Override
            protected Object doInvoke(T proxy, String methodName, 
                                      Class<?>[] parameterTypes, 
                                      Object[] arguments) throws Throwable {
                Method method = proxy.getClass().getMethod(methodName, parameterTypes);
                return method.invoke(proxy, arguments);
            }
        };
    }

}
```
在JdkProxyFactory的getInvoker()方法中直接new了一个AbstractProxyInvoker返回，并实现了doInvoke()抽象方法，该方法在执行返回的invoker的invoke()方法时回调，AbstractProxyInvoker的invoke()方法源码如下：

```
public Result invoke(Invocation invocation) throws RpcException {
    try {
        // 回调doInvoke()抽象方法
        return new RpcResult(doInvoke(proxy, invocation.getMethodName(), invocation.getParameterTypes(), invocation.getArguments()));
    } catch (InvocationTargetException e) {
        return new RpcResult(e.getTargetException());
    } catch (Throwable e) {
        throw new RpcException("Failed to invoke remote proxy method " + invocation.getMethodName() + " to " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}
```
所以这样把该invoker的具体执行逻辑转移到了AbstractProxyInvoker的抽象方法中，在JdkProxyFactory的getInvoker()方法中new AbstractProxyInvoker时被实现，其实现如上，通过反射执行接口实现相应方法。

### 通过协议Protocol建立监听服务
以上使用ProxyFactory将接口实现HelloServiceImpl封装成一个本地执行的Invoker。

执行这个服务，即执行这个本地Invoker，即调用这个本地Invoker的invoke(Invocation invocation)方法，方法的执行过程就是通过反射执行了HelloServiceImpl的内容。现在的问题是：这个方法的参数 Invocation invocation的来源问题。

针对server端来说，Protocol要解决的问题就是：根据指定协议对外公布这个HelloService服务，当客户端根据协议调用这个服 务时，将客户端传递过来的Invocation参数交给上述的Invoker来执行。所以Protocol加入了远程通信协议的这一块，根据客户端的请求 来获取参数Invocation invocation。

服务发布的第二个过程就是将invoker转换成Exporter，并建立监听服务

```
Exporter<?> exporter = protocol.export(invoker);
```
protocol采用SPI机制，此处为protocol接口的扩展代理实现，export(Invoker invoker)的过程即根据Invoker中url的配置信息来最终选择的Protocol实现，默认实现是"dubbo"的扩展实现即 DubboProtocol，然后再对DubboProtocol进行依赖注入，进行wrap包装。

在返回DubboProtocol之前，经过了ProtocolFilterWrapper、ProtocolListenerWrapper、RegistryProtocol的包装。使用装饰器模式，类似AOP的功能。

下面主要讲解RegistryProtocol和DubboProtocol，先暂时忽略ProtocolFilterWrapper、ProtocolListenerWrapper。

#### protocol的export()干了哪些事呢？
* 先提一下Exporter, Exporter前面已经提过，负责维护invoker的生命周期。该接口定义如下：

```
public interface Exporter<T> {

    Invoker<T> getInvoker();

    void unexport();
}
```
*  会先经过RegistryProtocol的export()
1. 利用内部的Protocol即DubboProtocol，将服务进行导出，如下

```
 final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker);
 
 private <T> ExporterChangeableWrapper<T>  doLocalExport(final Invoker<T> originInvoker){
    String key = getCacheKey(originInvoker);
    ExporterChangeableWrapper<T> exporter = (ExporterChangeableWrapper<T>) bounds.get(key);
    if (exporter == null) {
        synchronized (bounds) {
            exporter = (ExporterChangeableWrapper<T>) bounds.get(key);
            if (exporter == null) {
                final Invoker<?> invokerDelegete = new InvokerDelegete<T>(originInvoker, getProviderUrl(originInvoker));
                
                // 此处protocol为扩展代理实现，其真正底层实现为DubboProtocol
                // 即调用DubboProtocol.export()建立监听服务
                exporter = new ExporterChangeableWrapper<T>((Exporter<T>)protocol.export(invokerDelegete), originInvoker);
                bounds.put(key, exporter);
            }
        }
    }
    return (ExporterChangeableWrapper<T>) exporter;
}
```
2. 根据注册中心的registryUrl获取注册服务Registry，然后将serviceUrl注册到注册中心上,供客户端订阅
```
//registry provider
final Registry registry = getRegistry(originInvoker);
final URL registedProviderUrl = getRegistedProviderUrl(originInvoker);
registry.register(registedProviderUrl);
```
注意区分registryUrl和serviceUrl,前者是注册中心url，后者服务提供者url

* 下面来详细看看上述DubboProtocol的服务导出功能：

1. 创建一个DubboExporter，封装invoker。然后根据url的port、path（接口的名称）、版本号、分组号作为 key，将DubboExporter存至Map<String, Exporter<?>> exporterMap中
```
// DubboProtocol中成员变量
protected final Map<String, Exporter<?>> exporterMap = new ConcurrentHashMap<String, Exporter<?>>();

// export service.
String key = serviceKey(url);
DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);
exporterMap.put(key, exporter);
```

2. 首先根据Invoker的url获取ExchangeServer通信对象（负责与客户端的通信模块），以url中的host和port作 为key存至Map<String, ExchangeServer> serverMap中。既可以采用全部服务的通信交给这一个ExchangeServer通信对象，也可以某些服务单独使用新的 ExchangeServer通信对象。
```
// DubboProtocol中成员变量
private final Map<String, ExchangeServer> serverMap = new ConcurrentHashMap<String, ExchangeServer>(); // <host:port,Exchanger>

// DubboProtocol中的export()方法中
openServer(url);

private void openServer(URL url) {
    // find server.
    String key = url.getAddress();
    //client can export a service which's only for server to invoke
    boolean isServer = url.getParameter(Constants.IS_SERVER_KEY, true);
    if (isServer) {
        ExchangeServer server = serverMap.get(key);
        if (server == null) {
            // ExchangeServer不存在，调用createServer创建
            serverMap.put(key, createServer(url));
        } else {
            // server supports reset, use together with override
            server.reset(url);
        }
    }
}

private ExchangeServer createServer(URL url) {
    // send readonly event when server closes, it's enabled by default
    url = url.addParameterIfAbsent(Constants.CHANNEL_READONLYEVENT_SENT_KEY, Boolean.TRUE.toString());
    // enable heartbeat by default
    url = url.addParameterIfAbsent(Constants.HEARTBEAT_KEY, String.valueOf(Constants.DEFAULT_HEARTBEAT));
    String str = url.getParameter(Constants.SERVER_KEY, Constants.DEFAULT_REMOTING_SERVER);

    if (str != null && str.length() > 0 && !ExtensionLoader.getExtensionLoader(Transporter.class).hasExtension(str))
        throw new RpcException("Unsupported server type: " + str + ", url: " + url);

    url = url.addParameter(Constants.CODEC_KEY, DubboCodec.NAME);
    ExchangeServer server;
    try {
        // 绑定监听，并设置接收请求处理器requestHandler，返回server
        // ExchangeServer为通信层相关概念，后面单独开篇介绍
        server = Exchangers.bind(url, requestHandler);
    } catch (RemotingException e) {
        throw new RpcException("Fail to start server(url: " + url + ") " + e.getMessage(), e);
    }
    str = url.getParameter(Constants.CLIENT_KEY);
    if (str != null && str.length() > 0) {
        Set<String> supportedTypes = ExtensionLoader.getExtensionLoader(Transporter.class).getSupportedExtensions();
        if (!supportedTypes.contains(str)) {
            throw new RpcException("Unsupported client type: " + str);
        }
    }
    return server;
}
```


现在我们要搞清楚我们的目的：通过通信对象获取客户端传来的Invocation invocation参数，然后找到对应的DubboExporter（即能够获取到本地Invoker）就可以执行服务了。

上述每一个ExchangeServer通信对象都绑定了一个ExchangeHandler requestHandler对象，内容简略如下：
```
// DubboProtocol类中成员变量
private ExchangeHandler requestHandler = new ExchangeHandlerAdapter() {

    public Object reply(ExchangeChannel channel, Object message) throws RemotingException {
        if (message instanceof Invocation) {
            Invocation inv = (Invocation) message;
            Invoker<?> invoker = getInvoker(channel, inv);
            
            RpcContext.getContext().setRemoteAddress(channel.getRemoteAddress());
            return invoker.invoke(inv);
        }
    }

    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        if (message instanceof Invocation) {
            reply((ExchangeChannel) channel, message);
        } else {
            super.received(channel, message);
        }
    }

    @Override
    public void connected(Channel channel) throws RemotingException {
        invoke(channel, Constants.ON_CONNECT_KEY);
    }

    @Override
    public void disconnected(Channel channel) throws RemotingException {
        
    }

    private void invoke(Channel channel, String methodKey) {
        
    }

    private Invocation createInvocation(Channel channel, URL url, String methodKey) {
        
    }
};
```
可以看到在获取到Invocation参数后，调用getInvoker(channel, inv)来获取本地Invoker。获取过程就是根据channel获取port，根据Invocation inv信息获取要调用的服务接口、版本号、分组号等，以此组装成key，从上述Map<String, Exporter<?>> exporterMap中获取Exporter，然后就可以找到对应的Invoker了，就可以顺利的调用服务了。

而对于通信这一块，接下来会专门来详细的说明。