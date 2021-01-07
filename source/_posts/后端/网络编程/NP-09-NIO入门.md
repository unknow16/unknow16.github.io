---
title: NP-09-NIO入门
date: 2018-09-11 15:22:53
tags: NetworkProgramming
---

Java NIO是从JDK1.4开始，NIO提供了与标准IO不同的IO工作方式。标准的IO基于字节流和字符流进行操作的，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。

## 入门示例
使用Buffer读写数据一般遵循以下四个步骤：

1. 写入数据到Buffer
1. 调用flip()方法
1. 从Buffer中读取数据
1. 调用clear()方法或者compact()方法

```
ByteBuffer bf = ByteBuffer.allocate(2);

// 关联文件到通道
RandomAccessFile randomAccessFile = new RandomAccessFile("C:\\test.txt", "rw");
FileChannel channel = randomAccessFile.getChannel();

// 把channel中的数据读到bf中
// index为返回读到的bytes数，读完的话会返回0或-1
int index = channel.read(bf); // 1
while(index != -1) { 
	System.out.println("已经读了" + index + "字节");
	
	bf.flip(); // 从头读
	while(bf.hasRemaining()) {
		System.out.println((char)bf.get());
	}
	
	bf.clear(); // 清空缓冲
	index = channel.read(bf);
}

System.out.println("退出");
```

## Channel
基本上，所有的 IO 在NIO 中都从一个Channel开始。NIO的通道类似流，但又有些不同：

- 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
- 通道可以异步地读写。
- 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。

NIO中的一些主要Channel的实现：

- FileChannel ：从文件中读写数据。
- DatagramChannel ：能通过UDP读写网络中的数据。
- SocketChannel ：能通过TCP读写网络中的数据。
- ServerSocketChannel ：可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

正如你所看到的，这些通道涵盖了UDP 和 TCP 网络IO，以及文件IO。

## Scatter/Gather
- 分散（scatter）: 将从Channel中读取的数据“分散（scatter）”到多个Buffer中。
- 聚集（gather）: 将多个Buffer中的数据“聚集（gather）”后发送到Channel。

scatter / gather经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。

#### Scattering Reads
Scattering Reads是指数据从一个channel读取到多个buffer中。

代码示例如下：
```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
```
注意buffer首先被插入到数组，然后再将数组作为channel.read() 的输入参数。read()方法按照buffer在数组中的顺序将从channel中读取的数据写入到buffer，当一个buffer被写满后，channel紧接着向另一个buffer中写。

Scattering Reads在移动下一个buffer前，必须填满当前的buffer，这也意味着它不适用于动态消息(译者注：消息大小不固定)。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如 128byte），Scattering Reads才能正常工作。

#### Gathering Writes
Gathering Writes是指数据从多个buffer写入到同一个channel。

代码示例如下：
```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers

ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);
```
buffers数组是write()方法的入参，write()方法会按照buffer在数组中的顺序，将数据写入到channel，注意只有position和limit之间的数据才会被写入。因此，如果一个buffer的容量为128byte，但是仅仅包含58byte的数据，那么这58byte的数据将被写入到channel中。因此与Scattering Reads相反，Gathering Writes能较好的处理动态消息。

## 通道之间的数据传输
在Java NIO中，如果两个通道中有一个是FileChannel，那你可以直接将数据从一个channel（译者注：channel中文常译作通道）传输到另外一个channel。

#### transferFrom()
FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中。下面是一个简单的例子：
```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

toChannel.transferFrom(position, count, fromChannel);
```
方法的输入参数position表示从position处开始向目标文件写入数据，count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。

此外要注意，在SoketChannel的实现中，SocketChannel只会传输此刻准备好的数据（可能不足count字节）。因此，SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中。

#### transferTo()

transferTo()方法将数据从FileChannel传输到其他的channel中。下面是一个简单的例子：
```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```
上面所说的关于SocketChannel的问题在transferTo()方法中同样存在。SocketChannel会一直传输数据直到目标buffer被填满。

## Pipe
 管道是2个线程之间的单向数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。
 
#### 创建管道
通过Pipe.open()方法打开管道。例如：


```
Pipe pipe = Pipe.open();
```

#### 向管道写数据
要向管道写数据，需要访问sink通道。像这样：



```
Pipe.SinkChannel sinkChannel = pipe.sink();
```

通过调用SinkChannel的write()方法，将数据写入SinkChannel,像这样：
```
String newData = "New String to write to file..." + System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}

```
#### 从管道读取数据
从读取管道的数据，需要访问source通道，像这样：

```
Pipe.SourceChannel sourceChannel = pipe.source();
```

调用source通道的read()方法来读取数据，像这样：


```
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = sourceChannel.read(buf);
```

read()方法返回的int值会告诉我们多少字节被读进了缓冲区。