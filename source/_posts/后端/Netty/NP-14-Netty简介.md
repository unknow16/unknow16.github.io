---
title: NP-14-Netty简介
date: 2018-09-11 15:23:47
toc: true
tags: NetworkProgramming
---

官网：https://netty.io/

```
Netty is an asynchronous event-driven network application framework
for rapid development of maintainable high performance protocol servers & clients.
```

Netty 是一个异步的、基于事件驱动的网络应用框架，支持快速地开发可维护的高性能的面向协议的服务器和客户端。

Netty 在 Java 网络应用框架中的地位就好比：Spring 框架在 JavaEE 开发中的地位
以下的框架都使用了 Netty，因为它们有网络通信需求！
- Cassandra - nosql 数据库
- Spark - 大数据分布式计算框架
- Hadoop - 大数据分布式存储框架
- RocketMQ - ali 开源的消息队列
- ElasticSearch - 搜索引擎
- gRPC - rpc 框架
- Dubbo - rpc 框架
- Spring 5.x - flux api 完全抛弃了 tomcat ，使用 netty 作为服务器端
- Zookeeper - 分布式协调框架


## 组件
### EventLoop
EventLoop 即事件循环对象, 本质是一个单线程执行器（同时维护了一个 Selector），里面有 run 方法处理 Channel 上源源不断的 io 事件。
它的继承关系比较复杂
- 一条线是继承自 j.u.c.ScheduledExecutorService 因此包含了线程池中所有的方法
- 另一条线是继承自 netty 自己的 OrderedEventExecutor，
  - 提供了 boolean inEventLoop(Thread thread) 方法判断一个线程是否属于此 EventLoop
  - 提供了 parent 方法来看看自己属于哪个 EventLoopGroup

### EventLoopGroup
EventLoopGroup 即事件循环组, 是一组 EventLoop，Channel 一般会调用 EventLoopGroup 的 register 方法来绑定其中一个 EventLoop，后续这个 Channel 上的 io 事件都由此 EventLoop 来处理（保证了 io 事件处理时的线程安全）
- 继承自 netty 自己的 EventExecutorGroup
  - 实现了 Iterable 接口提供遍历 EventLoop 的能力
  - 另有 next 方法获取集合中下一个 EventLoop

以一个简单的实现为例：

```java
// 内部创建了两个 EventLoop, 每个 EventLoop 维护一个线程
DefaultEventLoopGroup group = new DefaultEventLoopGroup(2);
System.out.println(group.next());
System.out.println(group.next());
System.out.println(group.next());
```

输出

```
io.netty.channel.DefaultEventLoop@60f82f98
io.netty.channel.DefaultEventLoop@35f983a6
io.netty.channel.DefaultEventLoop@60f82f98
```

也可以使用 for 循环

```java
DefaultEventLoopGroup group = new DefaultEventLoopGroup(2);
for (EventExecutor eventLoop : group) {
    System.out.println(eventLoop);
}
```

输出

```
io.netty.channel.DefaultEventLoop@60f82f98
io.netty.channel.DefaultEventLoop@35f983a6
```

💡 优雅关闭 `shutdownGracefully` 方法。
该方法会首先切换 `EventLoopGroup` 到关闭状态从而拒绝新的任务的加入，然后在任务队列的任务都处理完成后，停止线程的运行。从而确保整体应用是在正常有序的状态下退出的


## Linux的TCP内核模块实现
* tcp模块中有两个队列，如A,B
1. 客户端调用connect(host,port)向服务端发送SYN标志，请求连接（第一次握手）
2. 服务端收到请求，向客户端回送SYN ACK标识，同意连接（第二次握手）
3. 将该客户端连接加入A队列中。
4. 客户端向服务端发送ACK确认（第三次握手）
5. tcp模块会把A队列中的链接移动到B队列中，链接完成，
6. 服务端应用程序从accept()方法返回，即从B队列中取出完成三次握手的链接。
