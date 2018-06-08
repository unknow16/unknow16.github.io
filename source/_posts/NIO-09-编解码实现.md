---
title: 编解码实现
date: 2018-02-27 14:58:10
tags: NIO
---

### 编解码
* 说白了就是java序列化技术，序列化目的就是两个，第一进行网络传输，第二对象持久化
* 虽然我们可以使用java进行对象序列化，netty去传输，但是java序列号的硬伤太多，比如java序列化没法跨语言、序列化后码流太大、序列化性能太低等等
### 主流的编解码框架：
* JBoss的Marshalling包
* Google的Protobuf
* 基于Protobuf的Kyro
* MessagePack框架