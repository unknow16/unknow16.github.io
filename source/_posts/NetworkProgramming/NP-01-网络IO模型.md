---
title: NP-01-网络IO模型
date: 2018-09-11 15:21:02
tags: NetworkProgramming
---

## 前言
在进行本文之前先普及一些基础知识。

首先一个计算机的内存是划分为用户空间和内核空间的。程序只能使用用户空间的内存。

内核空间主要是指操作系统运行时所使用的用于程序调度、虚拟内存的使用或者连接硬件资源等的程序逻辑。内核代码有特别的权利，比如它能与设备控制器通讯，控制着整个用于区域进程的运行状态。

![image](https://note.youdao.com/yws/api/personal/file/C4713F1AD3ED429B8EB1E51A09F45AA4?method=download&shareKey=ba909557166c77279c55aecdc97eb283)

如上图，程序读取磁盘上文件的数据时，都是先通过DMA(磁盘控制器)从磁盘读出存放到到内核空间的缓冲区，再复制到用户空间中该程序的缓冲区中的。另外从网络中读取到的数据也是先到达内核空间的缓冲区中，再复制到用户空间中该程序的缓冲区中的。

再具体说一下IO发生时涉及的对象和步骤。对于一个network IO (这里我们以read举例)，它会涉及到两个系统对象，一个是调用这个IO的process (or thread)，另一个就是系统内核(kernel)。当一个read操作发生时，它会经历两个阶段：
1. 等待数据准备 (Waiting for the data to be ready)
2. 将数据从内核拷贝到进程中(Copying the data from the kernel to the process)
    
记住这两点很重要，因为这些IO模型的区别就是在两个阶段上各有不同的情况。

#### 同步/异步
该概念比较广，不仅在IO领域，还如同步调用/异步调用、同步请求/异步请求，都是一个意思。关注的是消息通知机制。

同步：在发布一个“调用请求”时，在没有得到结果之前，该“调用请求”不会返回，但一旦返回就得到了返回值。即“调用者”主动等待“调用”的结果。

异步：正好和同步相反，“调用”发出之后，这个调用就直接返回了，所以没有返回结果。即调用者不能立即得到结果，因此适用于那些对数据一致性要求不高的场景。获取异步调用的结果，被调用者可通过状态、通知来通知调用者，或通过回调函数处理这个调用，Java中Future/FutureTask、wait/notify。

#### 阻塞/非阻塞
关注的是程序所在线程在等待调用结果的状态。

阻塞调用指的是调用结果返回之前，当前调用线程会被挂起，调用线程只有在得到结果之后才会返回。

非阻塞调用指的是在不能立即得到结果之前，该调用不会阻塞当前线程。

## 网络IO模型
本文讨论的背景是Linux环境下的network IO。

Richard Stevens博士在《UNIX网络编程》(第1卷)(套接口API第3版)书中一共比较了以下五种IO Model，目前讨论的都基于这五种，可惜他已不在。
* blocking IO
* nonblocking IO
* IO multiplexing()
* sig)nal driven IO
* asynchronous IO

由signal driven IO在实际中并不常用，所以主要介绍其余四种IO Model。

recvfrom()：下文会提到该函数，此处提一下，他的作用是用于从（已连接）套接口上接收数据，并捕获数据发送源的地址。

### blocking IO(阻塞IO模型)
在linux中，默认情况下所有的socket都是blocking，一个典型的读操作流程大概是这样：

![image](https://note.youdao.com/yws/api/personal/file/279FB8BFFE9042809C84A665CCAFDFE2?method=download&shareKey=35c6255e1f2e9bbf4abab89e7c72400e)

1. 当用户进程调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：准备数据。
 
2. 对于network io来说，很多时候数据在一开始还没有到达（比如，还没有收到一个完整的UDP包），这个时候kernel就要等待足够的数据到来。
 
3. 而在用户进程这边，整个进程会被阻塞。当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态，重新运行起来。

所以，blocking IO的特点就是在IO执行的两个阶段（等待数据和拷贝数据两个阶段）都被block了。

几乎所有的程序员第一次接触到的网络编程都是从listen()、send()、recv() 等接口开始的，这些接口都是阻塞型的。使用这些接口可以很方便的构建服务器/客户机的模型。如下一问一答的模型：

![image](https://note.youdao.com/yws/api/personal/file/F195438D05EF4C9D860C47D61AEDCC08?method=download&shareKey=266eb1a389ad83bd3e23a237c4aab059)



我们注意到，大部分的socket接口都是阻塞型的。所谓阻塞型接口是指系统调用（一般是IO接口）不返回调用结果并让当前线程一直阻塞，只有当该系统调用获得结果或者超时出错时才返回。

实际上，除非特别指定，几乎所有的IO接口 ( 包括socket接口 ) 都是阻塞型的。这给网络编程带来了一个很大的问题，如在调用send()的同时，线程将被阻塞，在此期间，线程将无法执行任何运算或响应任何的网络请求。

一个简单的改进方案是在服务器端使用多线程（或多进程）。多线程（或多进程）的目的是让每个连接都拥有独立的线程（或进程），这样任何一个连接的阻塞都不会影响其他的连接。具体使用多进程还是多线程，并没有一个特定的模式。传统意义上，进程的开销要远远大于线程，所以如果需要同时为较多的客户机提供服务，则不推荐使用多进程；如果单个服务执行体需要消耗较多的CPU资源，譬如需要进行大规模或长时间的数据运算或文件访问，则进程较为安全。通常，使用pthread_create ()创建新线程，fork()创建新进程。

我们假设对上述的服务器/客户机模型，提出更高的要求，即让服务器同时为多个客户机提供一问一答的服务。于是有了如下的模型。

![image](https://note.youdao.com/yws/api/personal/file/4087F18CFB4D4B1DA4927E314D8B8E64?method=download&shareKey=59ef2ce00380b01ea23db859bf154af5)
    

在上述的线程 / 时间图例中，主线程持续等待客户端的连接请求，如果有连接，则创建新线程，并在新线程中提供为前例同样的问答服务。

很多初学者可能不明白为何一个socket可以accept多次。实际上socket的设计者可能特意为多客户机的情况留下了伏笔，让accept()能够返回一个新的socket。
    
下面是 accept 接口的原型：

```
int accept(int s, struct sockaddr *addr, socklen_t *addrlen);
```
  
输入参数s是从socket()，bind()和listen()中沿用下来的socket句柄值。执行完bind()和listen()后，操作系统已经开始在指定的端口处监听所有的连接请求，如果有请求，则将该连接请求加入请求队列。调用accept()接口正是从 socket s 的请求队列抽取第一个连接信息，创建一个与s同类的新的socket返回句柄。新的socket句柄即是后续read()和recv()的输入参数。如果请求队列当前没有请求，则accept() 将进入阻塞状态直到有请求进入队列。

上述多线程的服务器模型似乎完美的解决了为多个客户机提供问答服务的要求，但其实并不尽然。如果要同时响应成百上千路的连接请求，则无论多线程还是多进程都会严重占据系统资源，降低系统对外界响应效率，而线程与进程本身也更容易进入假死状态。

很多程序员可能会考虑使用“线程池”或“连接池”。“线程池”旨在减少创建和销毁线程的频率，其维持一定合理数量的线程，并让空闲的线程重新承担新的执行任务。“连接池”维持连接的缓存池，尽量重用已有的连接、减少创建和关闭连接的频率。这两种技术都可以很好的降低系统开销，都被广泛应用很多大型系统，如websphere、tomcat和各种数据库等。但是，“线程池”和“连接池”技术也只是在一定程度上缓解了频繁调用IO接口带来的资源占用。而且，所谓“池”始终有其上限，当请求大大超过上限时，“池”构成的系统对外界的响应并不比没有池的时候效果好多少。所以使用“池”必须考虑其面临的响应规模，并根据响应规模调整“池”的大小。

对应上例中的所面临的可能同时出现的上千甚至上万次的客户端请求，“线程池”或“连接池”或许可以缓解部分压力，但是不能解决所有问题。总之，多线程模型可以方便高效的解决小规模的服务请求，但面对大规模的服务请求，多线程模型也会遇到瓶颈，可以用非阻塞接口来尝试解决这个问题。
    
### nonblocking IO(非阻塞IO模型)
Linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时，流程是这个样子：

![image](https://note.youdao.com/yws/api/personal/file/3A16B3B10FA34058B87483A77739672A?method=download&shareKey=4f5a628e76f4fffe4fb67878434e17ec)
 
从图中可以看出，当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。

所以，在非阻塞式IO中，用户进程其实是需要不断的主动询问kernel数据准备好了没有。非阻塞的接口相比于阻塞型接口的显著差异在于，在被调用之后立即返回。
    
使用如下的函数可以将某句柄fd设为非阻塞状态。
    
```
fcntl( fd, F_SETFL, O_NONBLOCK );
```

下面将给出只用一个线程，但能够同时从多个连接中检测数据是否送达，并且接受数据的模型。

![使用非阻塞的接收数据模型](https://note.youdao.com/yws/api/personal/file/EA9296399DDD4BA4ABCB073F945E887D?method=download&shareKey=1a8f3f5397b69b10550de1e80e7ac229)

在非阻塞状态下，recv() 接口在被调用后立即返回，返回值代表了不同的含义。如在本例中，
* recv() 返回值大于 0，表示接受数据完毕，返回值即是接受到的字节数；
* recv() 返回 0，表示连接已经正常断开；
* recv() 返回 -1，且 errno 等于 EAGAIN，表示 recv 操作还没执行完成；
* recv() 返回 -1，且 errno 不等于 EAGAIN，表示 recv 操作遇到系统错误 errno。

可以看到服务器线程可以通过循环调用recv()接口，可以在单个线程内实现对所有连接的数据接收工作。但是上述模型绝不被推荐。因为，循环调用recv()将大幅度推高CPU 占用率；此外，在这个方案中recv()更多的是起到检测“操作是否完成”的作用，实际操作系统提供了更为高效的检测“操作是否完成“作用的接口，例如select()多路复用模式，可以一次检测多个连接是否活跃。
    
### IO multiplexing(I/O多路复用模型)
IO multiplexing这个词可能有点陌生，但是如果我说select/epoll，大概就都能明白了。有些地方也称这种IO方式为事件驱动IO(event driven IO)。我们都知道，select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select/epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。它的流程如图：

![image](https://note.youdao.com/yws/api/personal/file/1E3841E5537C4F79BF7EEC1770F9D576?method=download&shareKey=ad43b4927d7167255b81903ecc29a850)

当用户进程调用了select，那么整个进程会被block，而同时，kernel会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。

这个图和blocking IO的图其实并没有太大的不同，事实上还更差一些。因为这里需要使用两个系统调用(select和recvfrom)，而blocking IO只调用了一个系统调用(recvfrom)。但是，用select的优势在于它可以同时处理多个connection。（多说一句：所以，如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。）
    
在多路复用模型中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的process其实是一直被block的。只不过process是被select这个函数block，而不是被socketIO给block。因此select()与非阻塞IO类似。

大部分Unix/Linux都支持select函数，该函数用于探测多个文件句柄的状态变化。   
    
下面给出select接口的原型：

```
FD_ZERO(int fd, fd_set* fds) 
FD_SET(int fd, fd_set* fds) 
FD_ISSET(int fd, fd_set* fds) 
FD_CLR(int fd, fd_set* fds) 

int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout)
```

#### 缺点
这种模型的特征在于每一个执行周期都会探测一次或一组事件，一个特定的事件会触发某个特定的响应。我们可以将这种模型归类为“事件驱动模型”。

相比其他模型，使用select() 的事件驱动模型只用单线程（进程）执行，占用资源少，不消耗太多 CPU，同时能够为多客户端提供服务。如果试图建立一个简单的事件驱动的服务器程序，这个模型有一定的参考价值。

但这个模型依旧有着很多问题。首先select()接口并不是实现“事件驱动”的最好选择。因为当需要探测的句柄值较大时，select()接口本身需要消耗大量时间去轮询各个句柄。很多操作系统提供了更为高效的接口，如linux提供了epoll，BSD提供了kqueue，Solaris提供了/dev/poll，…。如果需要实现更高效的服务器程序，类似epoll这样的接口更被推荐。遗憾的是不同的操作系统特供的epoll接口有很大差异，所以使用类似于epoll的接口实现具有较好跨平台能力的服务器会比较困难。

其次，该模型将事件探测和事件响应夹杂在一起，一旦事件响应的执行体庞大，则对整个模型是灾难性的。如下例，庞大的执行体1的将直接导致响应事件2的执行体迟迟得不到执行，并在很大程度上降低了事件探测的及时性。
    
    
幸运的是，有很多高效的事件驱动库可以屏蔽上述的困难，常见的事件驱动库有libevent库，还有作为libevent替代者的libev库。这些库会根据操作系统的特点选择最合适的事件探测接口，并且加入了信号(signal) 等技术以支持异步响应，这使得这些库成为构建事件驱动模型的不二选择。下章将介绍如何使用libev库替换select或epoll接口，实现高效稳定的服务器模型。

实际上，Linux内核从2.6开始，也引入了支持异步响应的IO操作，如aio_read, aio_write，这就是异步IO。
    
#### selctor模式
当io事件（通道）注册到选择器之后，选择器会为其分配一个key,相当于一个标签，选择器采用轮询的方法轮询每个io事件，当io事件就绪后，选择器根据key选择出相应通道，通知其进行数据的读取或写入数据缓冲区。

一个多路复用器Selector可以负责成千上万个注册到其上的通道，轮询哪个通道准备好数据了，通知cpu执行io的读取和写入操作。


每个通道都会对选择器进行注册不同的事件状态，以便选择器查找
* SelectionKey.OP_CONNECT
* SelectionKey.OP_ACCEPT
* SelectionKey.OP_READ
* SelectionKey.OP_WRITE

## 异步IO（Asynchronous I/O）
Linux下的asynchronous IO其实用得不多，从内核2.6版本才开始引入。无需自己进行读写，操作系统负责将数据从内核拷贝到用户空间。Java从JDK1.7开始支持。先看一下它的流程：

![image](https://note.youdao.com/yws/api/personal/file/39507553BC6C4279AE8FAA2387852181?method=download&shareKey=69b29aa9bebb3cb520158d42cd568ed2)

用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。

用异步IO实现的服务器这里就不举例了，以后有时间另开文章来讲述。异步IO是真正非阻塞的，它不会对请求进程产生任何的阻塞，因此对高并发的网络服务器实现至关重要。

## 总结
到目前为止，已经将四个IO模型都介绍完了。现在回过头来回答最初的那几个问题：blocking和non-blocking的区别在哪，synchronous IO和asynchronous IO的区别在哪。

先回答最简单的这个：blocking与non-blocking。前面的介绍中其实已经很明确的说明了这两者的区别。调用blocking IO会一直block住对应的进程直到操作完成，而non-blocking IO在kernel还在准备数据的情况下会立刻返回。

在说明synchronous IO和asynchronous IO的区别之前，需要先给出两者的定义。Stevens给出的定义（其实是POSIX的定义）是这样子的：
* A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes;
* An asynchronous I/O operation does not cause the requesting process to be blocked;

两者的区别就在于synchronous IO做”IO operation”的时候会将process阻塞。按照这个定义，之前所述的blocking IO，non-blocking IO，IO multiplexing都属于synchronous IO。有人可能会说，non-blocking IO并没有被block啊。这里有个非常“狡猾”的地方，定义中所指的”IO operation”是指真实的IO操作，就是例子中的recvfrom这个系统调用。non-blocking IO在执行recvfrom这个系统调用的时候，如果kernel的数据没有准备好，这时候不会block进程。但是当kernel中数据准备好的时候，recvfrom会将数据从kernel拷贝到用户内存中，这个时候进程是被block了，在这段时间内进程是被block的。而asynchronous IO则不一样，当进程发起IO操作之后，就直接返回再也不理睬了，直到kernel发送一个信号，告诉进程说IO完成。在这整个过程中，进程完全没有被block。

还有一种不常用的signal driven IO，即信号驱动IO。总的来说，UNP中总结的IO模型有5种之多：阻塞IO，非阻塞IO，IO复用，信号驱动IO，异步IO。前四种都属于同步IO。阻塞IO不必说了。非阻塞IO ，IO请求时加上O_NONBLOCK一类的标志位，立刻返回，IO没有就绪会返回错误，需要请求进程主动轮询不断发IO请求直到返回正确。IO复用同非阻塞IO本质一样，不过利用了新的select系统调用，由内核来负责本来是请求进程该做的轮询操作。看似比非阻塞IO还多了一个系统调用开销，不过因为可以支持多路IO，才算提高了效率。信号驱动IO，调用sigaltion系统调用，当内核中IO数据就绪时以SIGIO信号通知请求进程，请求进程再把数据从内核读入到用户空间，这一步是阻塞的。
异步IO，如定义所说，不会因为IO操作阻塞，IO操作全部完成才通知请求进程。

各个IO Model的比较如图所示：

![image](https://note.youdao.com/yws/api/personal/file/E35C6F2508264583A4F4FC03EAA9CB6F?method=download&shareKey=84c428245a6258ea6b01ca25e56034a0)

经过上面的介绍，会发现non-blocking IO和asynchronous IO的区别还是很明显的。在non-blocking IO中，虽然进程大部分时间都不会被block，但是它仍然要求进程去主动的check，并且当数据准备完成以后，也需要进程主动的再次调用recvfrom来将数据拷贝到用户内存。而asynchronous IO则完全不同。它就像是用户进程将整个IO操作交给了他人（kernel）完成，然后他人做完后发信号通知。在此期间，用户进程不需要去检查IO操作的状态，也不需要主动的去拷贝数据。


https://www.cnblogs.com/findumars/p/6361627.html