---
title: Dubbo-14-通信层2-异步请求实现
date: 2018-05-17 11:45:46
tags: Dubbo
---

### 客户端同步调用和异步调用的实现
首先设想一下我们目前的通信方式，使用netty mina等异步事件驱动的通信框架，将Channel中信息都分发到Handler中去处理了，Handler中的send方法只负责不断的发送消息，receive方法只负责不断接收消息，这时候就产生一个问题：

客户端如何对应同一个Channel的接收的消息和发送的消息之间的匹配呢？

这也很简单，就需要在发送消息的时候，必须要产生一个请求id，将调用的信息连同id一起发给服务器端，服务器端处理完毕后，再将响应信息和上述请求id一起发给客户端，这样的话客户端在接收到响应之后就可以根据id来判断是针对哪次请求的响应结果了。

来看下DubboInvoker中的具体实现：

```
protected Result doInvoke(final Invocation invocation) throws Throwable {
    RpcInvocation inv = (RpcInvocation) invocation;
    final String methodName = RpcUtils.getMethodName(invocation);
    inv.setAttachment(Constants.PATH_KEY, getUrl().getPath());
    inv.setAttachment(Constants.VERSION_KEY, version);

    // Client的子接口
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
- 如果不需要返回值，直接使用send方法，发送出去，设置当期和线程绑定RpcContext的future为null
- 如果需要异步通信，使用request方法构建一个ResponseFuture，然后设置到和线程绑定RpcContext中
- 如果需要同步通信，使用request方法构建一个ResponseFuture，阻塞等待请求完成

可以看到的是它把ResponseFuture设置到与当前线程绑定的RpcContext中了，如果我们要获取异步结果，则需要通过RpcContext来获取当前线程绑定的RpcContext，然后就可以获取Future对象。如下所示：
```
// 同步时返回值
String result1 = helloService.hello("World");
// 异步时获取返回值
RpcContext.getContext().getFuture().get();
```

### 异步请求的整个实现过程
然后我们来看下异步请求的整个实现过程，即上述ExchangeClient的request方法的具体内容：
```
// currentClient的实现类为HeaderExchangeClient

// HeaderExchangeClient的channel成员变量
private final ExchangeChannel channel;
// HeaderExchangeClient的request()方法
public ResponseFuture request(Object request) throws RemotingException {
    return channel.request(request);
}

// ExchangeChannel接口的实现类为HeaderExchangeChannel

// HeaderExchangeChannel类的request()方法
public ResponseFuture request(Object request, int timeout) throws RemotingException {
        if (closed) {
            throw new RemotingException(this.getLocalAddress(), null, "Failed to send request " + request + ", cause: The channel " + this + " is closed!");
        }
        // create request.
        Request req = new Request();
        req.setVersion("2.0.0");
        req.setTwoWay(true);
        req.setData(request);
        DefaultFuture future = new DefaultFuture(channel, req, timeout);
        try {
            // 此处channel的为com.alibaba.dubbo.remoting.Channel
            // 其实现为NettyChannel
            channel.send(req);
        } catch (RemotingException e) {
            future.cancel();
            throw e;
        }
        return future;
    }
```
1. 创建出一个request对象，创建过程中就自动产生了requestId,如下
```
    // Request的无参构造方法
    public Request() {
        mId = newId();
    }
    
    // newId()方法
    private static final AtomicLong INVOKE_ID = new AtomicLong(0);
    private static long newId() {
        // getAndIncrement() When it grows to MAX_VALUE, it will grow to MIN_VALUE, and the negative can be used as ID
        return INVOKE_ID.getAndIncrement();
    }
```
2. 根据request请求封装成一个DefaultFuture对象，通过该对象的get方法就可以获取到请求结果。该方法会阻塞一直到请求结果产生。同时DefaultFuture对象会被存至DefaultFuture类如下结构中：


```
// DefaultFuture中的成员变量，key就是请求id
private static final Map<Long, DefaultFuture> FUTURES = new ConcurrentHashMap<Long, DefaultFuture>();
```
3. 将上述请求对象发送给服务器端，同时将DefaultFuture对象返给上一层函数，即DubboInvoker中，然后设置到当前线程中,返回空RpcResult对象，如下：

```
ResponseFuture future = currentClient.request(inv, timeout); 
RpcContext.getContext().setFuture(new FutureAdapter<Object>(future)); 
return new RpcResult();
```

4. 用户通过RpcContext来获取上述DefaultFuture对象，调用get()来获取请求结果，会阻塞至服务器端返产生结果给客户端。

```
// DefaultFuture类get()方法
public Object get() throws RemotingException {
    return get(timeout);
}

// 锁的条件
private final Condition done = lock.newCondition();
// 存放服务端响应对象
private volatile Response response;

public boolean isDone() {
    return response != null;
}

public Object get(int timeout) throws RemotingException {
    if (timeout <= 0) {
        timeout = Constants.DEFAULT_TIMEOUT;
    }
    // 1. isDone()中判读response != null
    if (!isDone()) {
        long start = System.currentTimeMillis();
        lock.lock();
        try {
            while (!isDone()) {
                // 2. 阻塞等待超时后while循环
                // 直至isDone()为true,即response != null
                done.await(timeout, TimeUnit.MILLISECONDS); <1> 
                if (isDone() || System.currentTimeMillis() - start > timeout) {
                    break;
                }
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
        if (!isDone()) {
            throw new TimeoutException(sent > 0, channel, getTimeoutMessage(false));
        }
    }
    return returnFromResponse();
}
```

5. 服务器端产生结果，返回给客户端会在客户端的channelHandler的receive方法中接收到，接收到之后判别接收的信息是Response后，就会根据response的id从上述FUTURES结构中查出对应的DefaultFuture对象，并把结果设置进去。此时DefaultFuture的get方法则不再阻塞，返回刚刚设置好的结果。看下面实现：

```
// HeaderExchangeHandler类的received()方法
public void received(Channel channel, Object message) throws RemotingException {
    channel.setAttribute(KEY_READ_TIMESTAMP, System.currentTimeMillis());
    ExchangeChannel exchangeChannel = HeaderExchangeChannel.getOrAddChannel(channel);
    try {
        if (message instanceof Request) {
            // handle request.
            Request request = (Request) message;
            if (request.isEvent()) {
                handlerEvent(channel, request);
            } else {
                if (request.isTwoWay()) {
                    Response response = handleRequest(exchangeChannel, request);
                    channel.send(response);
                } else {
                    handler.received(exchangeChannel, request.getData());
                }
            }
        } else if (message instanceof Response) {
            // 1. 当相应的message是Response时处理
            handleResponse(channel, (Response) message);
        } else if (message instanceof String) {
            if (isClientSide(channel)) {
                Exception e = new Exception("Dubbo client can not supported string message: " + message + " in channel: " + channel + ", url: " + channel.getUrl());
                logger.error(e.getMessage(), e);
            } else {
                String echo = handler.telnet(channel, (String) message);
                if (echo != null && echo.length() > 0) {
                    channel.send(echo);
                }
            }
        } else {
            handler.received(exchangeChannel, message);
        }
    } finally {
        HeaderExchangeChannel.removeChannelIfDisconnected(channel);
    }
}
    
static void handleResponse(Channel channel, Response response) throws RemotingException {
    if (response != null && !response.isHeartbeat()) {
        // 2. 将Response响应对象传给DefaultFuture
        DefaultFuture.received(channel, response);
    }
}

public static void received(Channel channel, Response response) {
    try {
        DefaultFuture future = FUTURES.remove(response.getId());
        if (future != null) {
            // 3. 去doReceived()
            future.doReceived(response);
        } else {
            logger.warn("The timeout response finally returned at "
                    + (new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS").format(new Date()))
                    + ", response " + response
                    + (channel == null ? "" : ", channel: " + channel.getLocalAddress()
                    + " -> " + channel.getRemoteAddress()));
        }
    } finally {
        CHANNELS.remove(response.getId());
    }
}

// 4. 锁定将相应对象赋值给response成员变量
private void doReceived(Response res) {
    lock.lock();
    try {
        response = res;
        if (done != null) {
            // 5. 为response赋完值
            // 通过done锁条件发送信号释放get()方法处的阻塞
            // 从而get()方法返回服务端执行结果
            done.signal();
        }
    } finally {
        lock.unlock();
    }
    if (callback != null) {
        invokeCallback(callback);
    }
}
```

至此异步通信大致就了解了，但是我们会发现一个问题：

当某个线程多次发送异步请求时，都会将返回的DefaultFuture对象设置到当前线程绑定的RpcContext中，就会造成了覆盖问题，如下调用方式：


```
String result1 = helloService.hello("World");
String result2 = helloService.hello("java");
System.out.println("result :"+result1);
System.out.println("result :"+result2);
System.out.println("result : "+RpcContext.getContext().getFuture().get());
System.out.println("result : "+RpcContext.getContext().getFuture().get());
```

即异步调用了hello方法，再次异步调用，则前一次的结果就被冲掉了，则就无法获取前一次的结果了。必须要调用一次就立马将DefaultFuture对象获取走，以免被冲掉。即这样写：


```
String result1 = helloService.hello("World");
Future<String> result1Future=RpcContext.getContext().getFuture();
String result2 = helloService.hello("java");
Future<String> result2Future=RpcContext.getContext().getFuture();
System.out.println("result :"+result1);
System.out.println("result :"+result2);
System.out.println("result : "+result1Future.get());
System.out.println("result : "+result2Future.get());
```

最后来张dubbo的解释图片：
![image](https://note.youdao.com/yws/api/personal/file/5D42066217EF48B88456F2714936FE5E?method=download&shareKey=f1bbf32e08c16fff443712cdea703026)