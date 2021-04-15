---
title: Redis的设计与实现
toc: true
date: 2021-03-18 13:50:19
tags:
categories:
---

编码: 将数据按照一定规则存储在内存中


## String
https://blog.csdn.net/z69183787/article/details/105821424?ivk_sa=1024320u

## List
双向链表

## Hash
https://mp.weixin.qq.com/s/MT1tB2_7f5RuOxKhuEm1vQ

哈希表的数据结构是数组，解决hash冲突用的连地址法
表示它的结构体中有两个hash表，一个有值，另一个用来实现渐进式rehash

大字典的扩容是比较耗时间的，需要重新申请新的数组，然后将旧字典所有链表中的元素重新挂接到新的数组下面，这是一个 O(n) 级别的操作，作为单线程的 Redis 很难承受这样耗时的过程，所以 Redis 使用 渐进式 rehash 小步搬迁：

## Set

## ZSet

## 跳跃表
https://blog.csdn.net/lz710117239/article/details/78408919



## 压缩列表ziplist




## 参考资料
> - []()
> - []()
