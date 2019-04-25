---
title: NP-04-Actor模型
date: 2018-09-11 15:21:48
tags: NetworkProgramming
---

## 并发线程通信策略
- 共享数据：如java中的同步，锁等机制都是基于共享内存数据实现多线程通信
- 消息传递：本文将提到的AKKA框架实现的Actor模型就是基于消息传递的

使用共享数据方式的并发编程面临的最大的一个问题就是数据条件竞争（data race）。处理各种锁的问题是让人十分头痛的一件事。

和共享数据方式相比，消息传递机制最大的优点就是不会产生数据竞争状态（data race）。实现消息传递有两种常见的类型：基于channel的消息传递和基于Actor的消息传递。本文主要是来分享Scala的Actor模型。

## Actor模型概念
Actor是计算机科学领域中的一个并行计算模型。在Actor理论中，一切都被认为是Actor，这和面向对象语言里一切都被看成对象很类似。但包括面向对象语言在内的软件通常是顺序执行的，而Actor模型本质上则是并发的。

Actor的概念来自于Erlang，在AKKA中，可以认为一个Actor就是一个容器，用以存储状态、行为、Mailbox以及子Actor与Supervisor策略。Actor之间并不直接通信，而是通过Mail来互通有无。

每个Actor都有一个(恰好一个)Mailbox。Mailbox相当于是一个小型的队列，一旦Sender发送消息，就是将该消息入队到接受者的Mailbox中。入队的顺序按照消息发送的时间顺序。Mailbox有多种实现，默认为FIFO。但也可以根据优先级考虑出队顺序，实现算法则不相同。

每个Actor可以异步的发送消息到接受者的Mailbox而不会被阻塞，虽然多个Actor可同时发送消息，但接受的Actor只会按照它的Mailbox中接受消息的顺序依次来处理消息，且仅在当前消息处理完成后才会处理下一个消息，因此我们只需要关心发送消息时的并发问题即可。

## 一个Actor的组成
一个Actor由三个重要部分组成，它们是状态（state），行为（Behavior）和邮箱（Mailbox），Actor与Actor之间的交互通过消息发送来完成。

- 状态：指的是Actor对象的变量信息，它可以是Actor对象中的局部变量、占用的机器资源等，状态只会根据Actor接受的消息而改变，从而避免并发环境下的死锁等问题
- 行为：指的是Actor的计算行为逻辑，它通过处理Actor接收的消息而改变Actor状态
- 邮箱：建立起Actor间的连接，即Actor发送消息后，另外一个Actor将接收的消息放入到邮箱中待后期处理，邮箱的内部实现是通过队列来实现的，队列可以是有界的（Bounded）也可以是无界的（Unbounded），有界队列实现的邮箱容量固定，无界队列实现的邮箱容易不受限制。 

不难看出，Actor模型是对现实世界的高度抽象，它具有如下特点：
1. Actor之间使用消息传递机制进行通信，传递的消息使用的是不可变消息，Actor之间并不共享数据结构，如果有数据共享则通过消息发送的方式进行
2. 各Actor都有对应的mailbox，如果其它Actor向该Actor发送消息，消息将入队待后期处理
3. Actor间的消息传递通过异步的方式进行，即消息的发送者发送完消息后不必等待回应便可以返回继承处理其它任务。

## Akka并发编程框架
Scala语言中原生地支持Actor模型，只不过功能还不够强大，从Scala 2.10版本之后，Akka框架成为Scala包的一部分，可以在程序中直接使用。

Akka是一个以Actor模型为基础构建的基于事件的并发编程框架，底层使用Scala语言实现，提供Java和Scala两种API。

Akka框架意在简化高并发、可扩展及分布式应用程序的设计，它具有如下优势： 
1. 使用Akka框架编写的应用程序既可以横向扩展（Scale Out）、也可纵向扩展（Scale Up）。 
2. 编写并发应用程序更简单，Akka提供了更高的抽象，开发人员只需要专注于业务逻辑，而无需像Java语言那样需要处理底级语义如线程、锁及非阻塞IO等。 3. 高容错，Akka使用“let it crashes”机制，当Actor出错时可以快速恢复。 
4. 事件驱动的架构，Akka中的Actor之间的通信采用异步消息发送，能够完美支持事件驱动。 
5. 位置透明，无论是Actor运行在本地机器还是远程机器上，对于用户来说都是透明的，这极大地简化了多核处理器和分布式系统上的应用程序编程。 
6. 事务支持能力，支持软件事务内存（software transactional memory，STM），使Actor具有原子消息流的操作能力。 

Akka框架由下列十个组件构成： 
- akka-actor ：包括经典的Actor、Typed Actors、IO Actor等 
- akka-remote：远程Actor 
- akka-testkit：测试Actor系统的工具箱 
- akka-kernel ：Akka微内核，用于运行精简的微型应用程序服务器，无需运行于Java应用服务器上。 
- akka-transactor ：Transactors 即支持事务的 actors，集成了Scala STM 
- akka-agent – 代理, 同样集成了Scala STM 
- akka-camel – 集成Apache Camel 
- akka-zeromq – 集成ZeroMQ 消息队列 
- akka-slf4j – 支持SLF4J 日志功能 
- akka-filebased-mailbox – 支持基于文件的mailbox


http://ifeve.com/concurrency-modle-seven-week-actor-5/