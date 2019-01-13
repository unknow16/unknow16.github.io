---
title: CP-14-通过ReentrantLock来窥探AQS实现
date: 2018-10-13 20:12:05
tags: ConcurrentPrograming
---

#### 目录
1. JUC-Lock类继承图
1. AQS结构
1. ReentrantLock结构
1. 加锁过程
1. 解锁过程
1. 公平锁和非公平锁的区别
2. ReentrantLock总结

---

### JUC-Lock类继承图
![h](https://note.youdao.com/yws/api/personal/file/61A90B9C2E9845F4BF24EAF50DB904C8?method=download&shareKey=abccc1804975d19454d423d3f215da1b)

毕竟Lock体系有这么多工具类可用，在此我们以ReentrantLock为例来说明AQS原理。

### AQS结构
先看下AbstractQueuedSynchronizer的一些重要属性：
```
// 头结点，你直接把它当做 当前持有锁的线程 可能是最好理解的
private transient volatile Node head;
// 阻塞的尾节点，每个新的节点进来，都插入到最后，也就形成了一个隐视的链表
private transient volatile Node tail;
// 这个是最重要的，不过也是最简单的，代表当前锁的状态，0代表没有被占用，大于0代表有线程持有当前锁
// 之所以说大于0，而不是等于1，是因为锁可以重入嘛，每次重入都加上1
private volatile int state;
// 代表当前持有独占锁的线程，举个最重要的使用例子，因为锁可以重入
// reentrantLock.lock()可以嵌套调用多次，所以每次用这个来判断当前线程是否已经拥有了锁
// if (currentThread == getExclusiveOwnerThread()) {state++}
private transient Thread exclusiveOwnerThread; //继承自AbstractOwnableSynchronizer
```

见名知义，其会包含一个Queue，该队列用双向链表实现，其每个Node是对等待获取锁的线程的封装，结构属性如下：

```
static final class Node {
    /** Marker to indicate a node is waiting in shared mode */
    // 标识节点当前在共享模式下
    static final Node SHARED = new Node();
    /** Marker to indicate a node is waiting in exclusive mode */
    // 标识节点当前在独占模式下
    static final Node EXCLUSIVE = null;

    // ======== 下面的几个int常量是给waitStatus用的 ===========
    /** waitStatus value to indicate thread has cancelled */
    // 代码此线程取消了争抢这个锁
    static final int CANCELLED =  1;
    /** waitStatus value to indicate successor's thread needs unparking */
    // 官方的描述是，其表示当前node的后继节点对应的线程需要被唤醒
    static final int SIGNAL    = -1;
    /** waitStatus value to indicate thread is waiting on condition */
    // 本文不分析condition，所以略过吧，下一篇文章会介绍这个
    static final int CONDITION = -2;
    /**
     * waitStatus value to indicate the next acquireShared should
     * unconditionally propagate
     */
    // 同样的不分析，略过吧
    static final int PROPAGATE = -3;
    // =====================================================

    // 取值为上面的1、-1、-2、-3，或者0(以后会讲到)
    // 这么理解，暂时只需要知道如果这个值 大于0 代表此线程取消了等待，
    // 也许就是说半天抢不到锁，不抢了，ReentrantLock是可以指定timeouot的。。。
    volatile int waitStatus;
    // 前驱节点的引用
    volatile Node prev;
    // 后继节点的引用
    volatile Node next;
    // 这个就是线程本尊
    volatile Thread thread;

}
```
该队列不包含头节点，示图如下：
![image](https://note.youdao.com/yws/api/personal/file/A0BD1999F23B404EADB644784DAB92D9?method=download&shareKey=ac98af2dc492e81c9714c251d0a577f4)

## ReentrantLock结构
从上面的类继承图可知，ReentrantLock类内部结构

1. 抽象静态内部类Sync继承自AbstractQueuedSynchronizer
2. Sync有两个final内部实现类FairSync和NonfairSync

所以 ReentrantLock 在内部用了内部类 Sync 来管理锁，所以真正的获取锁和释放锁是由 Sync 的实现类来控制的。

首先，我们先看下 ReentrantLock 的使用方式。
```
// 我用个web开发中的service概念吧
public class OrderService {
    // 使用static，这样每个线程拿到的是同一把锁，当然，spring mvc中service默认就是单例，别纠结这个
    private static ReentrantLock reentrantLock = new ReentrantLock(true);

    public void createOrder() {
        // 比如我们同一时间，只允许一个线程创建订单
        reentrantLock.lock();
        // 通常，lock 之后紧跟着 try 语句
        try {
            // 这块代码同一时间只能有一个线程进来(获取到锁的线程)，
            // 其他的线程在lock()方法上阻塞，等待获取到锁，再进来
            // 执行代码...
            // 执行代码...
            // 执行代码...
        } finally {
            // 释放锁
            reentrantLock.unlock();
        }
    }
}

// ReentrantLock构造方法，默认为非公平重入锁，传true则是公平锁
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

## 加锁过程
ReentrantLock构造时，会构造一个FairSync或NonfairSync，此处以FairSync为例，ReentrantLock实例的lock方法内部就是调用FairSync的lock方法，其获取锁的过程简单如下：

1. 首先尝试直接获取锁，如果获取成功，则整个加锁过程就结束了，获取成功的条件有2个，一个是上面提到的AQS中的state属性，判断该属性值等于0，表示没有线程持有该锁，则通过CAS设置其值为1，设置成功则表示该锁获取成功，但是此时用的是公平锁，所以会在CAS抢锁前判断在阻塞队列中是否有线程在等待获取锁，有的话乖乖去队尾排队，稍后细说与非公平锁的区别，另一个成功条件是，state大于0，表示有线程持有锁，但锁是可重入的，此时判断AQS的属性exclusiveOwnerThread是否为当前线程，如果是，则重入获取锁成功，state+1。如果不满足这两个条件，则需要把当前线程挂起，放到阻塞队列中。
2. 尝试获取锁失败后，则会把当前线程包装成一个上面提到的Node，同时采用自旋的方式CAS设置到等待队列的队尾。
3. 当前线程入队后，在挂起之前会调判断该元素是否为head，是则再次尝试获取锁，如果满足1中所说的两个条件则获取成功，否则执行挂起逻辑。
4. 在说明挂起逻辑前，这里需要知道这点：进入阻塞队列排队的线程会被挂起，而唤醒的操作是由前驱节点完成的。再次解释一下上面提到的Node的一个属性waitStatus，值为1代表该节点的线程取消了抢锁，值为-1代表后驱节点需要被唤醒。
5. 挂起逻辑：先判断当前节点的前驱节点的waitStatus是否为-1，如果是则直接挂起该节点的线程，如果值大于0，则往前循环遍历，直到找到一个waitStatus为-1的前驱节点，和该前驱节点进行双向关联，并挂起当前节点的线程。简单说，就是为了找个好爹，因为你还得依赖它来唤醒呢，如果前驱节点取消了排队，找前驱节点的前驱节点做爹，往前循环总能找到一个好爹的。
6. 至此，该节点线程被挂起，等待前驱节点唤醒。

## 解锁过程
实际就是将AQS的state-1，并唤醒后驱节点线程的过程。

1. 首先获取AQS的state的值减1并重新设置值，因为此时持有锁，所以该操作不用CAS设置。
2. 然后会判断state是否为0，如果是则唤醒后继节点，否则表示该锁多次重入，不进行唤醒操作。
3. 在唤醒时，有可能后继节点取消了等待（waitStatus==1），此时会从队尾往前找，找到waitStatus<=0的所有节点中排在最前面的节点线程进行唤醒。
4. 唤醒后继节点线程后，其会尝试获取锁，此时如果加锁时采用了NonfairSync非公平锁，刚好新来一个线程来抢锁，通过CAS设置了state和exclusiveOwnerThread，则表示获取锁成功，然后前者需要继续等待被唤醒，所以不公平，但是效率要高些。

## 公平锁和非公平锁的区别
ReentrantLock 默认采用非公平锁，除非你在构造方法中传入参数 true 。


```
public ReentrantLock() {
    sync = new NonfairSync();
}
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```
1. 在调用lock后，非公平锁会比公平锁多一步，直接CAS去设置AQS的state，设置成功，则获取锁成功，整个加锁过程就结束了，此处的CAS是在上面加锁过程1中之前进行的，不会判断state是否为0，而是直接CAS(0, 1),即老值是0则设置为1。
2. 然后在上面加锁过程1中，非公平锁会少一步判断，即等待队列中是否有线程在等待获取锁，再次CAS抢锁。
3. 如果这两次CAS都抢锁失败，则非公平锁和公平锁一样要进入阻塞队列等待唤醒。

相对来说，非公平锁有更好的性能，因为它的吞吐量比较大，但是非公平锁让每个线程获取锁的时间更加不确定，可能导致在阻塞队列中的线程长期处于饥饿状态。


## ReentrantLock总结
在并发环境下，加锁和解锁需要以下三个部件的协调：

1. 锁状态。我们要知道锁是不是被别的线程占有了，这个就是 state 的作用，它为 0 的时候代表没有线程占有锁，可以去争抢这个锁，用 CAS 将 state 设为 1，如果 CAS 成功，说明抢到了锁，这样其他线程就抢不到了，如果锁重入的话，state进行+1 就可以，解锁就是减 1，直到 state 又变为 0，代表释放锁，所以 lock() 和 unlock() 必须要配对啊。然后唤醒等待队列中的第一个线程，让其来占有锁。
1. 线程的阻塞和解除阻塞。AQS 中采用了 LockSupport.park(thread) 来挂起线程，用 unpark 来唤醒线程。
1. 阻塞队列。因为争抢锁的线程可能很多，但是只能有一个线程拿到锁，其他的线程都必须等待，这个时候就需要一个 queue 来管理这些线程，AQS 用的是一个 FIFO 的队列，就是一个链表，每个 node 都持有后继节点的引用。