---
title: NP-06-Actor模型和CSP模型的区别
date: 2018-09-11 15:22:12
tags: NetworkProgramming
---

Akka/Erlang的actor模型与Go语言中通过协程Goroutine与通道Channel实现的CSP(Communicating Sequential Processes)模型有什么区别呢？

首先这两者都是并发模型的解决方案，我们看看Actor和Channel这两个方案的不同：

## Actor模型
在Actor模型中，主角是Actor，类似一种worker，Actor彼此之间直接发送消息，不需要经过什么中介，消息是异步发送和处理的：

![image](https://note.youdao.com/yws/api/personal/file/BC182AF296084368B8CC9E618181E3ED?method=download&shareKey=fdb364bc88468a3f2ea6f7f6b63baa0b)

Actor模型描述了一组为了避免并发编程的常见问题的公理:

1. 所有Actor状态是Actor本地的，外部无法访问。
2. Actor必须只有通过消息传递进行通信。　　
3. 一个Actor可以响应消息:推出新Actor,改变其内部状态,或将消息发送到一个或多个其他参与者。
4. Actor可能会堵塞自己,但Actor不应该堵塞它运行的线程。

## 基于Channel的CSP模型
Channel模型中，worker之间不直接彼此联系，而是通过不同channel进行消息发布和侦听。消息的发送者和接收者之间通过Channel松耦合，发送者不知道自己消息被哪个接收者消费了，接收者也不知道是哪个发送者发送的消息。

![image](https://note.youdao.com/yws/api/personal/file/A81BD2BDD45A4B27836673A5395B91FD?method=download&shareKey=5c2d06ac8d052a067ad069a8d23f3499)

Go语言的CSP模型是由协程Goroutine与通道Channel实现：

- Go协程goroutine: 是一种轻量线程，它不是操作系统的线程，而是将一个操作系统线程分段使用，通过调度器实现协作式调度。是一种绿色线程，微线程，它与Coroutine协程也有区别，能够在发现堵塞后启动新的微线程。
- 通道channel: 类似Unix的Pipe，用于协程之间通讯和同步。协程之间虽然解耦，但是它们和Channel有着耦合。

## Actor模型和CSP区别

Actor模型和CSP区别图如下：

![image](https://note.youdao.com/yws/api/personal/file/3232E12B018C403A91A2A688253073A4?method=download&shareKey=10bf2b5bf79b16495fbe1e44818550c8)

Actor之间直接通讯，而CSP是通过Channel通讯，在耦合度上两者是有区别的，后者更加松耦合。

同时，它们都是描述独立的流程通过消息传递进行通信。主要的区别在于：在CSP消息交换是同步的(即两个流程的执行"接触点"的，在此他们交换消息)，而Actor模型是完全解耦的，可以在任意的时间将消息发送给任何未经证实的接受者。由于Actor享有更大的相互独立,因为他可以根据自己的状态选择处理哪个传入消息。自主性更大些。

在Go语言中为了不堵塞流程，程序员必须检查不同的传入消息，以便预见确保正确的顺序。CSP好处是Channel不需要缓冲消息，而Actor理论上需要一个无限大小的邮箱作为消息缓冲。 

[[并发之痛 Thread，Goroutine，Actor]]: http://jolestar.com/parallel-programming-model-thread-goroutine-actor/

https://blog.csdn.net/hotdust/article/details/72475630