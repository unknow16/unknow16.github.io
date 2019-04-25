---
title: Dubbo-13-通信层1-封装框架
date: 2018-05-17 11:45:35
tags: Dubbo
---

目前dubbo已经集成的有netty、mina、grizzly。先来通过案例简单了解下netty、mina编程（grizzly没有了解过）

### Netty和mina的简单案例
netty原本是jboss开发的，后来单独出来了，所以会有两种版本就是org.jboss.netty和io.netty两种包类型的，而dubbo内置的是前者。目前还不是很熟悉，可能稍有差别，但是整体大概都是一样的。

我们先来看下io.netty的案例：
```
public static void main(String[] args){
    EventLoopGroup bossGroup=new NioEventLoopGroup();
    EventLoopGroup workerGroup = new NioEventLoopGroup();
    try {
        ServerBootstrap serverBootstrap=new ServerBootstrap();
        serverBootstrap.group(bossGroup,workerGroup)
            .channel(NioServerSocketChannel.class)
            .childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new TcpServerHandler());
                }
            });
        ChannelFuture f=serverBootstrap.bind(8080).sync();
        f.channel().closeFuture().sync();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }finally {  
        workerGroup.shutdownGracefully();  
        bossGroup.shutdownGracefully();  
    }  
}
```
mina的案例：


```
public static voidmain(String[] args)throws IOException{
    IoAcceptor acceptor = new NioSocketAcceptor();
    acceptor.getFilterChain().addLast("codec",new ProtocolCodecFilter(
            new TextLineCodecFactory(Charset.forName("UTF-8"),"\r\n", "\r\n")));
    acceptor.setHandler(new TcpServerHandler());  
    acceptor.bind(new InetSocketAddress(8080));  
}
```
两者都是使用Reactor模型结构,即基于多路复用。而原始BIO模型，每来一个Socket连接都为该Socket创建一个线程来处理。由于总线程数有限制，导致Socket连接受阻，所以BIO模型并发量并不大。

而Reactor模型，用一个boss线程，创建Selector，用于不断监听Socket连接、客户端的读写操作等，用一个线程池即workers，负责处理Selector派发的读写操作。由于boss线程可以接收更多的Socket连接，同时可以充分利用线程池中的每个线程，减少了BIO模型下每个线程为单独的socket的等待时间。

### 服务器端如何集成netty和mina
先来简单总结下上述netty和mina的相似之处，然后进行抽象概括成接口

1. 各自有各自的编程启动方式
2. 都需要各自的ChannelHandler实现，用于处理各自的Channel或者IoSession的连接、读写等事件。

- 对于netty来说： 需要继承org.jboss.netty.channel.SimpleChannelHandler（或者其他方式），来处理org.jboss.netty.channel.Channel的连接读写事件

- 对于mina来说：需要继承org.apache.mina.common.IoHandlerAdapter（或者其他方式），来处理org.apache.mina.common.IoSession的连接读写事件


为了统一上述问题，dubbo需要做如下事情：

#### 1. 定义dubbo的com.alibaba.dubbo.remoting.Channel接口
```
public interface Channel extends Endpoint {

    InetSocketAddress getRemoteAddress();

    boolean isConnected();

    boolean hasAttribute(String key);

    Object getAttribute(String key);

    void setAttribute(String key,Object value);
    
    void removeAttribute(String key);
}

public interface Endpoint {

    URL getUrl();

    ChannelHandler getChannelHandler();

    InetSocketAddress getLocalAddress();

    void send(Object message) throws RemotingException;

    void send(Object message, boolean sent) throws RemotingException;

    void close();

    void close(int timeout);
    
    boolean isClosed();
}
```


- 针对netty，上述Channel的实现为NettyChannel，内部含有一个netty自己的 org.jboss.netty.channel.Channel channel对象，即该com.alibaba.dubbo.remoting.Channel接口的功能实现全部委托为底层的 org.jboss.netty.channel.Channel channel对象来实现

- 针对mina，上述Channel实现为MinaChannel，内部包含一个mina自己的 org.apache.mina.common.IoSession session对象，即该com.alibaba.dubbo.remoting.Channel接口的功能实现全部委托为底层的 org.apache.mina.common.IoSession session对象来实现

#### 2. 定义自己的com.alibaba.dubbo.remoting.ChannelHandler接口，用于处理com.alibaba.dubbo.remoting.Channel接口的连接读写事件，如下所示
```
@SPI
public interface ChannelHandler {

    void connected(Channel channel) throws RemotingException;

    void disconnected(Channel channel) throws RemotingException;

    void sent(Channel channel, Object message) throws RemotingException;

    void received(Channel channel, Object message) throws RemotingException;

    void caught(Channel channel, Throwable exception) throws RemotingException;
}
```
- 针对Netty, 先定义用于处理netty的NettyHandler，需要按照netty的方式继承netty的 org.jboss.netty.channel.SimpleChannelHandler，此时NettyHandler就可以委托dubbo的 com.alibaba.dubbo.remoting.ChannelHandler接口实现来完成具体的功能，在交给 com.alibaba.dubbo.remoting.ChannelHandler接口实现之前，需要先将netty自己的 org.jboss.netty.channel.Channel channel转化成上述的NettyChannel，见NettyHandler
- 针对Mina, 先定义用于处理mina的MinaHandler，需要按照mina的方式继承mina的 org.apache.mina.common.IoHandlerAdapter，此时MinaHandler就可以委托dubbo的 com.alibaba.dubbo.remoting.ChannelHandler接口实现来完成具体的功能，在交给 com.alibaba.dubbo.remoting.ChannelHandler接口实现之前，需要先将mina自己的 org.apache.mina.common.IoSession转化成上述的MinaChannel，见MinaHandler


做了上述事情之后，全部逻辑就统一到dubbo自己的com.alibaba.dubbo.remoting.ChannelHandler接口如何来处理自己的com.alibaba.dubbo.remoting.Channel接口。

这就需要看下com.alibaba.dubbo.remoting.ChannelHandler接口的实现有哪些：
![image](https://note.youdao.com/yws/api/personal/file/C1CE4BE00E8B48D9BE5CA709D38B4103?method=download&shareKey=cb9958ee0cda72618035924d9363d8d3)
#### 3. 定义Server接口用于统一大家的启动流程

先来看下整体的Server接口实现情况
![image](https://note.youdao.com/yws/api/personal/file/E8577798CABA431BBFBFE2C089FA3BE1?method=download&shareKey=bbb29c3585ba229fafc20787759ebdc4)
如NettyServer的启动流程： 按照netty自己的API启动方式，然后依据外界传递进来的com.alibaba.dubbo.remoting.ChannelHandler接口实现，创建出NettyHandler，最终对用户的连接请求的处理全部交给NettyHandler来处理，NettyHandler又交给了外界传递进来的com.alibaba.dubbo.remoting.ChannelHandler接口实现。

至此就将所有底层不同的通信实现全部转化到了外界传递进来的com.alibaba.dubbo.remoting.ChannelHandler接口的实现上了。

而上述Server接口的另一个分支实现HeaderExchangeServer则充当一个装饰器的角色，为所有的Server实现增添了如下功能：

向该Server所有的Channel依次进行心跳检测：

- 如果当前时间减去最后的读取时间大于heartbeat时间或者当前时间减去最后的写时间大于heartbeat时间，则向该Channel发送一次心跳检测
- 如果当前时间减去最后的读取时间大于heartbeatTimeout，则服务器端要关闭该Channel，如果是客户端的话则进行重新连接（客户端也会使用这个心跳检测任务）
 
### 客户端如何集成netty和mina
服务器端了解了之后，客户端就也非常清楚了，整体类图如下：

![image](https://note.youdao.com/yws/api/personal/file/7EC061A69C9D4E1AB74D5DDA8E990343?method=download&shareKey=9259f17867b0f37fd8cd7dffd0edd39b)

如NettyClient在使用netty的API开启客户端之后，仍然使用NettyHandler来处理。还是最终转化成com.alibaba.dubbo.remoting.ChannelHandler接口实现上了，如ExchangeHandlerAdapter具体实现类被HeatbeatHandler装饰类装饰。

HeaderExchangeClient和上面的HeaderExchangeServer非常类似，就不再提了。

我们可以看到这样集成完成之后，就完全屏蔽了底层通信细节，将逻辑全部交给了com.alibaba.dubbo.remoting.ChannelHandler接口的实现上了。从上面我们也可以看到，该接口实现也会经过层层装饰类的包装，才会最终交给底层通信。

