---
title: Dubbo-15-通信层3-上层调用
date: 2018-05-17 11:45:56
tags: Dubbo
---

### 通信层与dubbo的结合
从上面可以了解到如何对不同的通信框架进行抽象，屏蔽底层细节，统一将逻辑交给ChannelHandler接口实现来处理。然后我们就来了解下如何与dubbo的业务进行对接，也就是在什么时机来使用上述通信功能：

### 服务的发布过程使用通信功能
如DubboProtocol在发布服务的过程中：

1. DubboProtocol中有一个如下结构

```
Map<String, ExchangeServer> serverMap
```


在发布一个服务的时候会先根据服务的url获取要发布的服务所在的host和port，以此作为key来从上述结构中寻找是否已经有对应的ExchangeServer（上面已经说明）。

2. 如果没有的话，则会创建一个，创建过程如下：


```
ExchangeServer server = Exchangers.bind(url, requestHandler);
```
其中requestHandler就是DubboProtocol自身实现的ChannelHandler。

获取一个ExchangeServer，它的实现主要是Server的装饰类，依托外部传递的Server来实现Server功能，而自己加入一些额外的功能，如ExchangeServer的实现HeaderExchangeServer，就是加入了心跳检测的功能。

所以此时我们可以自定义扩展功能来实现Exchanger。接口定义如下：

```
@SPI(HeaderExchanger.NAME) // header
public interface Exchanger {

    @Adaptive({Constants.EXCHANGER_KEY}) // exchanger
    ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException;

    @Adaptive({Constants.EXCHANGER_KEY})
    ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException;

}
```
默认使用的就是HeaderExchanger，它创建的ExchangeServer是HeaderExchangeServer如下所示：
```
public class HeaderExchanger implements Exchanger {
    
    public static final String NAME = "header";

    public ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
        return new HeaderExchangeClient(Transporters.connect(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
    }

    public ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
        return new HeaderExchangeServer(Transporters.bind(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
    }

}
```
HeaderExchangeServer仅仅是一个Server接口的装饰类，需要依托外部传递Server实现来完成具体的功能。此 Server实现可以是netty也可以是mina等。所以我们可以自定义Transporter实现来选择不同底层通信框架，接口定义如下：

```
@SPI("netty")
public interface Transporter {

    @Adaptive({Constants.SERVER_KEY, Constants.TRANSPORTER_KEY})
    Server bind(URL url, ChannelHandler handler) throws RemotingException;

    @Adaptive({Constants.CLIENT_KEY, Constants.TRANSPORTER_KEY})
    Client connect(URL url, ChannelHandler handler) throws RemotingException;

}
```
默认采用netty实现，如下：

```
public class NettyTransporter implements Transporter {

    public static final String NAME = "netty";
    
    public Server bind(URL url, ChannelHandler listener) throws RemotingException {
        return new NettyServer(url, listener);
    }

    public Client connect(URL url, ChannelHandler listener) throws RemotingException {
        return new NettyClient(url, listener);
    }

}
```
至此就到了我们上文介绍的内容了。同时DubboProtocol的ChannelHandler实现经过层层装饰器包装，最终传给底层通信Server。

客户端发送请求给服务器端时，底层通信Server会将请求经过层层处理最终传递给DubboProtocol的ChannelHandler实 现，在该实现中，会根据请求参数找到对应的服务器端本地Invoker，然后执行，再将返回结果通过底层通信Server发送给客户端。
### 客户端的引用服务使用通信功能
在DubboProtocol引用服务的过程中：

1 使用如下方式创建client


```
ExchangeClient client=Exchangers.connect(url ,requestHandler)；
```

requestHandler还是DubboProtocol中ChannelHandler实现。

和Server类似，我们可以通过自定义Exchanger实现来创建出不同功能的ExchangeClient。默认的Exchanger实现是HeaderExchanger,源码如下
```
public class HeaderExchanger implements Exchanger {
    
    public static final String NAME = "header";

    public ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
        return new HeaderExchangeClient(Transporters.connect(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
    }

    public ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
        return new HeaderExchangeServer(Transporters.bind(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
    }

}
```
创建出来的ExchangeClient是HeaderExchangeClient，它也是Client的包装类，仅仅在Client外层加上心跳检测的功能，向它所连接的服务器端发送心跳检测。

HeaderExchangeClient需要外界给它传一个Client实现，这是由Transporter接口实现来定的，默认是NettyTransporter，源码如下
```
public class NettyTransporter implements Transporter {

    public static final String NAME = "netty";
    
    public Server bind(URL url, ChannelHandler listener) throws RemotingException {
        return new NettyServer(url, listener);
    }

    public Client connect(URL url, ChannelHandler listener) throws RemotingException {
        return new NettyClient(url, listener);
    }

}
```
创建出来的的Client实现是NettyClient。

同时DubboProtocol的ChannelHandler实现经过层层装饰器包装，最终传给底层通信Client。

客户端的DubboInvoker调用远程服务的时候，会将调用信息通过ExchangeClient发送给服务器端，然后返回一个 ResponseFuture，根据客户端选择的同步还是异步方式，来决定阻塞还是直接返回，这一部分在上文同步调用和异步调用的实现中已经详细说过了。