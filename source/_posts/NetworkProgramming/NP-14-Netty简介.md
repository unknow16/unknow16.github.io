---
title: NP-14-Netty简介
date: 2018-09-11 15:23:47
tags: NetworkProgramming
---

Netty是一款异步的事件驱动的网络应用程序框架，支持快速地开发可维护的高性能的面向协议的服务器和客户端。

* Hadoop的RPC框架Avro，JMS，RocketMQ，Dubbo底层都采用netty实现

### Linux的TCP内核模块实现
* tcp模块中有两个队列，如A,B
1. 客户端调用connect(host,port)向服务端发送SYN标志，请求连接（第一次握手）
2. 服务端收到请求，向客户端回送SYN ACK标识，同意连接（第二次握手）
3. 将该客户端连接加入A队列中。
4. 客户端向服务端发送ACK确认（第三次握手）
5. tcp模块会把A队列中的链接移动到B队列中，链接完成，
6. 服务端应用程序从accept()方法返回，即从B队列中取出完成三次握手的链接。
