---
title: NP-17-TCP粘包拆包
date: 2018-09-19 17:42:38
tags: NetworkProgramming
---

## TCP粘包拆包场景
TCP编程底层默认实现了粘包和拆包机制。

因为我们在C/S这种传输模型下，以TCP协议传输的时候，在网络中的byte其实就像是河水，TCP就像一个搬运工,将这流水从一端转送到另一端，这时又分两种情况：

1. 如果客户端的每次制造的水比较多，也就是我们常说的客户端给的包比较大，TCP这个搬运工就会分多次去搬运。

2. 如果客户端每次制造的水比较少的话，TCP可能会等客户端多次生产之后，把所有的水一起再运输到另一端

上述第一种情况，就是需要我们进行粘包，在另一端接收的时候，需要把多次获取的结果粘在一起，变成我们可以理解的信息，第二种情况，我们在另一端接收的时候，就必须进行拆包处理，因为每次接收的信息，可能是另一个远程端多次发送的包，被TCP粘在一起的。

#### 粘包
当客户端与服务端发送的数据包大于单次能发送的最大数据包时，会将数据包拆成多个部分，分别发送

当收到所有数据包后，将各个数据包粘成一个数据包给用户使用。

#### 拆包
当发送的数据包较小时，会对多个数据包进行合并成一个，进行一次发送，

当收到该包时，会对数据包拆成发送前的多个数据包，给用户使用。

#### 解决方案

根据业界主流协议，有三种方案：

1. 消息定长，例如每个报文的大小固定为200个字节 ，如果不够，空格补位
2. 在包尾部增加特殊字符进行分割，例如加回车
3. 将消息分为消息头和消息体，在消息头中包含表示消息总长度的字段，然后进行业务逻辑的处理


Netty提供了几个常用的解码器，帮助我们解决这些问题，其实上述的粘包和拆包的问题，归根结底的解决方案就是发送端给远程端一个标记，告诉远程端，每个信息的结束标志是什么，这样，远程端获取到数据后，根据跟发送端约束的标志，将接收的信息分切或者合并成我们需要的信息，这样我们就可以获取到正确的信息了。

netty提供前两种：
* LineBasedFrameDecoder(行分割符)/DellmiterBasedFrameDecoder(自定义分隔符)
* FixedLengthFrameDecoder(定长)


## LineBasedFrameDecoder
我们可以在发送的信息中，加一个结束标志，例如两个远程端规定以行来切分数据，那么发送端，就需要在每个信息体的末尾加上行结束的标志，部分代码如下：
```
// 行分割
System.getProperty("line.separator")
```
打完标记，其实Server中还不知道是以行为结尾的，所以我们需要修改server的handler链，在inbound链中加一个额外的处理链，判断一下，获取的信息按照行来切分，我们很庆幸，这样枯燥的代码Netty已经帮我们完美地完成了，Netty提供了一个LineBasedFrameDecoder这个类，顾名思义，这个类名字中有decoder，说明是一个解码器，它是继承ByteToMessageDecoder的，是将byte类型转化成Message的，所以我们应该将这个解码器放在inbound处理器链的第一个，添加代码如下：
```
ch.pipeline().addLast(new LineBasedFrameDecoder(2048));
```
其中2048是规定一行数据最大的字节数，如果一行数据太大，会报TooLongFrameException异常

## DelimiterBasedFrameDecoder
作用：按照某个固定字符切分。

服务端在添加如下handler:
```
ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024,Unpooled.copiedBuffer("$$__".getBytes())));
```


## FixedLengthFrameDecoder
作用：定长数据帧的解码器。

服务端在添加如下handler:

```
 ch.pipeline().addLast(new FixedLengthFrameDecoder(23));
```
如上我们就是在channelhandler链中，加入了FixedLengthFrameDecoder，且参数是23，告诉Netty，获取的帧数据有23个字节就切分一次
