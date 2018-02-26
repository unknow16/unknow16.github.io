---
title: 并发集合
date: 2018-02-26 21:02:41
tags: Concurrent
---

## ConcurrentMap接口
* V putIfAbsent(K key, V value)
* boolean remove(Object key, Object value)
* V replace(K key, V value)
* boolean replace(K key, V oldValue, V newValue)

#### ConcurrentHashMap类
* 最高支持16个段 segment，细粒度同步
* 

#### ConcurrentSkipListMap
支持并发排序类似treeMap

---
## Copy-On-Write容器
#### CopyOnWriteArrayList/CopyOnWriteArraySet
适用于读多写少
写时复制，指当往一个容器中做写（add,del,update）操作时，不直接往当前容器中添加
而是将当前容器进行copy，复制出一个新的容器，然后对容器进行写操作，
完成之后，再将原容器的引用指向新的容器，
这样做可以对CopyOnWrite容器进行并发的读，而不需要加锁，
提现读写分离的思想，读和写不同的容器、

---
## 并发Queue
#### ConcurrentLinkedQueue类
* 基于链表的无界线程安全队列
* 

#### BlockingQueue
