---
title: 阻塞队列
date: 2018-02-26 21:02:52
tags: Concurrent
---



## 概述
* 阻塞队列常用于生产者消费者的场景，生产者往队列中添加元素，消费者从队列中获取元素，支持两个附加操作：当阻塞队列满时，生产者将阻塞直到队列可用，当阻塞队列空时，消费者将阻塞直到队列非空。
* 常见队列为一端插入元素，另一端获取元素，存在一种双向队列，两端均可插入和获取元素
* 都是线程安全的
## 处理方法

方法\处理方式 | 抛出异常 | 返回特殊值 | 一直阻塞 | 超时退出
---|---|---|---|---|
插入方法 | add(e) | offer(e) | put(e) | offer(e, time, unit)
移除方法 | remove(e) | poll(e) | take() | poll(time, unit)
检查方法 | element() | peek() | 不可用 | 不可用

* 抛出异常：是指当阻塞队列满时候，再往队列里插入元素，会抛出IllegalStateException(“Queue full”)异常。当队列为空时，从队列里获取元素时会抛出NoSuchElementException异常 。
* 返回特殊值：插入方法会返回是否成功，成功则返回true。移除方法，则是从队列里拿出一个元素，如果没有则返回null
* 一直阻塞：当阻塞队列满时，如果生产者线程往队列里put元素，队列会一直阻塞生产者线程，直到拿到数据，或者响应中断退出。当队列空时，消费者线程试图从队列里take元素，队列也会阻塞消费者线程，直到队列可用。
* 超时退出：当阻塞队列满时，队列会阻塞生产者线程一段时间，如果超过一定的时间，生产者线程就会退出。

## 类
* ArrayBlockingQueue: 一个由数组结构组成的有界阻塞队列
* LinkedBlockingQueue: 一个由链表结构组成的有界阻塞队列
* PriorityBlockingQueue: 一个支持优先级排序的无界阻塞队列
* 
* SynchronousQueue: 一个不存储元素的阻塞队列
* 
* DelayQueue： 一个使用优先级队列实现的无界阻塞队列
* LinkedBlockingDeque: 一个由链表结构组成的双向阻塞队列
* 
* LinkedTransferQueue: 一个由链表结构组成的无界阻塞队列

## SynchronousQueue
SynchronousQueue是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。SynchronousQueue可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常适合于传递性场景,比如在一个线程中使用的数据，传递给另外一个线程使用，SynchronousQueue的吞吐量高于LinkedBlockingQueue 和 ArrayBlockingQueue。

## 参考
[ifeve](http://ifeve.com/java-blocking-queue/)


