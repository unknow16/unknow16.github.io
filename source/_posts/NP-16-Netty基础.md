---
title: NP-16-Netty基础
date: 2018-09-19 17:41:32
tags: NetworkProgramming
---

## 流程
```
/**
 *                   ________________________                                 __________________________
 *                  |                        |                               |                          |    
 *                  |   <-----Inbound-----   |                               |   ---inbound------- >    |   ________
 *                  |   _____        ______  |                               |    _______      ____     |  |        |
 *      _______     |  |     |       |    |  |                               |    |     |     |    |    |  |        |  
 *     |       |    |  |  ②  |       |  ③ |  |      ___________________      |    |  ⑤  |     |  ⑥ |    |  |        |
 *     |       |    |  |_____|       |____|  |     |                   |     |    |_____|     |____|    |  |        |     
 *     |client |----|-------______-----------|-----|      network      |-----|--------------------------|--| server |
 *     |       |    |       |     |          |     |___________________|     |          ______          |  |        |
 *     |       |    |       |  ①  |          |                               |          |     |         |  |        |         
 *     |       |    |       |_____|          |                               |          |  ④  |         |  |________|
 *     |       |    |                        |                               |          |_____|         |
 *     |_______|    |   -----Outbound--->    |                               |    <-----outbound----    | 
 *                  |___ChannelPipeline______|                               |______ChannelPipeline_____| 
 *                                                                               
 *  ①：StringEncoder继承于MessageToMessageEncoder，而MessageToMessageEncoder又继承于ChannelOutboundHandlerAdapter
 *  ②：ClientHandler.java
 *  ③：StringDecoder继承于MessageToMessageDecoder，而MessageToMessageDecoder又继承于ChannelInboundHandlerAdapter
 *  ④：StringEncoder 编码器
 *  ⑤：StringDecoder 解码器
 *  ⑥：ServerHandler.java
 *  
 */

```


## SimpleChannelInboundHandler
其中指定的范型即解码后的对象，会自动释放资源。

----

## ChannelHandler/ChannelHandlerContext/ChannelPipeline

- 一个Channel对应一个ChannelPipe
- 一个ChannelPipeline中包含多个ChannelHandler
- 一个ChannelHandler对应一个ChannelHandlerContext

这三者的关系很特别，相辅相成，一个ChannelPipeline中可以有多个ChannelHandler实例，而每一个ChannelHandler实例与ChannelPipeline之间的桥梁就是ChannelHandlerContext实例，如图所示：
![image](https://note.youdao.com/yws/api/personal/file/A36AC1D87862444EBF3DEAF7C8E0EAE3?method=download&shareKey=6b2d1541573480d4106d8af6bed1d8c7)

看图就知道，ChannelHandlerContext的重要性了，如果你获取到了ChannelHandlerContext的实例的话，你可以获取到你想要的一切，你可以根据ChannelHandlerContext执行ChannelHandler中的方法，我们举个例子来说，我们可以看下ChannelHandlerContext部分API:
```
ChannelHandlerContext fireChannelRegistered();
ChannelHandlerContext fireChannelUnregistered();
ChannelHandlerContext fireChannelActive();
ChannelHandlerContext fireChannelInactive();
ChannelHandlerContext fireExceptionCaught(Throwable cause);
```
这几个API都是使用比较频繁的，都是调用当前handler之后同一类型的channel中的某个方法，这里的同一类型指的是同一个方向，比如inbound调用inbound，outbound调用outbound类型的channel，一般来说，都是一个channel的ChannnelActive方法中调用fireChannelActive来触发调用下一个handler中的ChannelActive方法。

也就是说如果一个channelPipeline中有多个channelHandler时，且这些channelHandler中有同样的方法时，例如这里的channelActive方法，只会调用处在第一个的channelHandler中的channelActive方法，如果你想要调用后续的channelHandler的同名的方法就需要调用以“fire”为开头的方法了，这样做很灵活。

## ByteBuf
网络传输的载体是byte，这是任何框架谁也逃脱不了的一种规定，JAVA的NIO提供了ByteBuffer，用来完成这项任务，当然ByteBuffer也很好的完成了这个任务，Netty也提供了一个名字很相似的载体叫做ByteBuf，相比于ByteBuf而言，它有着更加更多友善的API,也更加易于维护，并且它可以扩容。

一般来说，ByteBuf都是维护一个byte数组的，它的内部格式是长成这个样子的：
```
 *      +-------------------+------------------+------------------+
 *      | discardable bytes |  readable bytes  |  writable bytes  |
 *      |                   |     (CONTENT)    |                  |
 *      +-------------------+------------------+------------------+
 *      |                   |                  |                  |
 *      0      <=      readerIndex   <=   writerIndex    <=    capacity

```
与原生态的ByteBuffer相比，它维护了两个指针，一个是读指针，一个是写指针，而原生态的ByteBuffer只维护了一个指针，你需要调用flip方法来切换读写的状态，不易用户管理维护

读的时候，可读的区域是下标区间是[readerIndex，writeIndex)，可写区间的是[writerIndex,capacity-1]，但是discardable这段区间就会变得相对无用，既不能读，也不能写。

#### discardReadBytes()
所以我们可以使用discardReadBytes的方法进行内存空间的回收，回收之后是这样的：
```
 *      +------------------+--------------------------------------+
 *      |  readable bytes  |    writable bytes (got more space)   |
 *      +------------------+--------------------------------------+
 *      |                  |                                      |
 * readerIndex (0) <= writerIndex (decreased)        <=        capacity

```

discardReadBytes之后，可读段被移到了该内存空间的最左端，可写段从空间容量来说，变大了，变成了回收之前的可写段+discard段内存之和，这样做的唯一问题就是性能问题，因为可读段的字节迁移问题，如果大量调用该方法，会产生很多的复制操作，所以除非能获取discard的很大空间，一般情况下，高并发的情况下，不适合多调用

#### clear()
当然还有clear方法，这个方法简单易懂，调用之后ByteBuf是长成这样的：
```
 *      +---------------------------------------------------------+
 *      |             writable bytes (got more space)             |
 *      +---------------------------------------------------------+
 *      |                                                         |
 *      0 = readerIndex = writerIndex            <=            capacity
```
#### duplicate()
复制当前对象，复制后的对象与前对象共享缓冲区，且维护自己的独立索引

#### copy()
复制一份全新的对象，内容和缓冲区都不是共享的

#### slice()
获取调用者的子缓冲区，且与原缓冲区共享缓冲区

## ByteBuf分类

![image](https://note.youdao.com/yws/api/personal/file/CB8C736A6E274D9DA19BEC551FFF2E0F?method=download&shareKey=8f44dc4e5b976a12024cc14d9de05819)

![image](https://note.youdao.com/yws/api/personal/file/7CE66B79FDAE4417BE4CD536B36E3593?method=download&shareKey=2e92b7e8f7058ffa546958034664374f)

![image](https://note.youdao.com/yws/api/personal/file/8167E58450774273BD1A72D75A197452?method=download&shareKey=efd349a61573686af90a0137040ee878)