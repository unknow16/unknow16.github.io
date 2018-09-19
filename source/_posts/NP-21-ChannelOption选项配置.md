---
title: NP-21-ChannelOption选项配置
date: 2018-09-19 17:45:26
tags: NetworkProgramming
---

## SO_BACKLOG
设置tcp缓冲区

## SO_SNDBUF
设置发送缓冲大小

## SO_RCVBUF
设置接收缓冲大小

## SO_KEEPALIVE
保持连接

## TCP_NODELAY
在有些网络通信的场景下，要求低延迟，就要设置这个选项。

- 在客户端我们需要这样设置：
```
bootstap.option(ChannelOption.TCP_NODELAY, true);
```

- 在服务器端是在worker的Channel端设置属性，
```
boot.childOption(ChannelOption.TCP_NODELAY, true);
```

设置这样做好的好处就是禁用nagle算法。

#### Nagle算法
TCP/IP协议中，无论发送多少数据，总是要在数据前面加上协议头，同时，对方接收到数据，也需要发送ACK表示确认。为了尽可能的利用网络带宽，TCP总是希望尽可能的发送足够大的数据。（一个连接会设置MSS参数，因此，TCP/IP希望每次都能够以MSS尺寸的数据块来发送数据）。
Nagle算法就是为了尽可能发送大块数据，避免网络中充斥着许多小数据块。

Nagle算法的基本定义是任意时刻，最多只能有一个未被确认的小段。 所谓“小段”，指的是小于MSS尺寸的数据块，所谓“未被确认”，是指一个数据块发送出去后，没有收到对方发送的ACK确认该数据已收到。

举个例子，一开始client端调用socket的write操作将一个int型数据(称为A块)写入到网络中，由于此时连接是空闲的（也就是说还没有未被确认的小段），因此这个int型数据会被马上发送到server端，接着，client端又调用write操作写入‘/r/n’（简称B块），这个时候，A块的ACK没有返回，所以可以认为已经存在了一个未被确认的小段，所以B块没有立即被发送，一直等待A块的ACK收到（大概40ms之后），B块才被发送。

#### ACK延迟机制
这里还隐藏了一个问题，就是A块数据的ACK为什么40ms之后才收到？这是因为TCP/IP中不仅仅有nagle算法，还有一个ACK延迟机制 。当Server端收到数据之后，它并不会马上向client端发送ACK，而是会将ACK的发送延迟一段时间（假设为t），它希望在t时间内server端会向client端发送应答数据，这样ACK就能够和应答数据一起发送，就像是应答数据捎带着ACK过去。在我之前的时间中，t大概就是40ms。这就解释了为什么'/r/n'(B块)总是在A块之后40ms才发出。

如果你觉着nagle算法太捣乱了，那么可以通过设置TCP_NODELAY将其禁用 。当然，更合理的方案还是应该使用一次大数据的写操作，而不是多次小数据的写操作。