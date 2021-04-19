---
title: InnoDB存储引擎
toc: true
date: 2021-04-10 23:28:27
tags:
categories:
---

## 14.1

## InnoDB In-Memory Structures
- Buffer Pool
- Change Buffer
- Adaptive Hash Index
- Log Buffer

## InnoDB On-Disk Structures

### Tablespace
https://blog.csdn.net/qq_44961149/article/details/108420073


### Doublewrite Buffer
Doublewrite Buffer是一个磁盘上的存储缓冲区，存储的是来自内存的Buffer Pool中的被修改过的Page。

一个被修改的数据Page到InnoDB data file流程：
1. 修改的数据页最初存在内存的Buffer Pool中
2. 然后被flush到磁盘的Doublewrite Buffer中
3. 最后从Doublewrite Buffer中把数据页写到数据文件的合适位置

当在上面第3步时，mysqld进程exit，在crash recovery时，InnoDB能从Doublewrite Buffer中获取一个完整的被修改的数据页数据，进行故障恢复。

尽管数据被写了两次，但是不需要两倍的IO代价和IO操作，采用操作系统的fsync()将修改的数据页作为一个大的chunk写到Doublewrite Buffer中的。除了一种情况，数据页直接存放在操作系统的某个磁盘分区上，对应的innodb_flush_method被设置成O_DIRECT_NO_FSYNC。

默认是启用的，想禁用的话可以innodb_doublewrite = 0 

## 参考资料
> - []()
> - []()
