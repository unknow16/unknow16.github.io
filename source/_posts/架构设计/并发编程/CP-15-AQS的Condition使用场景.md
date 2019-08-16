---
title: CP-15-AQS的Condition使用场景
date: 2018-10-13 20:12:24
tags: ConcurrentPrograming
---

我们先来看看 Condition 的使用场景，Condition 经常可以用在生产者-消费者的场景中，请看 Doug Lea 给出的这个例子：
```
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class BoundedBuffer {
    final Lock lock = new ReentrantLock();
    // condition 依赖于 lock 来产生
    final Condition notFull = lock.newCondition();
    final Condition notEmpty = lock.newCondition();

    final Object[] items = new Object[100];
    int putptr, takeptr, count;

    // 生产
    public void put(Object x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length)
                notFull.await();  // 队列已满，等待，直到 not full 才能继续生产
            items[putptr] = x;
            if (++putptr == items.length) putptr = 0; //满后索引归0，循环从0放入，此时0处已被消费了
            ++count;
            notEmpty.signal(); // 生产成功，队列已经 not empty 了，发个通知出去
        } finally {
            lock.unlock();
        }
    }

    // 消费
    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0)
                notEmpty.await(); // 队列为空，等待，直到队列 not empty，才能继续消费
            Object x = items[takeptr];
            if (++takeptr == items.length) takeptr = 0; //消费完后索引归0，循环从0取
            --count;
            notFull.signal(); // 被我消费掉一个，队列 not full 了，发个通知出去
            return x;
        } finally {
            lock.unlock();
        }
    }
}
```
ArrayBlockingQueue 采用这种方式实现了生产者-消费者，所以请只把这个例子当做学习例子，实际生产中可以直接使用 ArrayBlockingQueue。

## 构造Condition
首先，明确一点，Condition 是依赖于 ReentrantLock 的，不管是调用 await 进入等待还是 signal 唤醒，都必须获取到锁才能进行操作。

可以通过多次调用ReentrantLock的newCondition()方法获取多个ConditionObject对象。

```
// ReentrantLock中
public Condition newCondition() {
    return sync.newCondition();
}
    
// ReentrantLock的内部抽象类Sync中
// ConditionObject是AQS的内部类，实现了Condition接口
final ConditionObject newCondition() {
    return new ConditionObject();
}
```

## ConditionObject结构
我们首先来看下我们关注的 Condition 的实现类 AbstractQueuedSynchronizer 类中的 ConditionObject。

```
public class ConditionObject implements Condition, java.io.Serializable {
        private static final long serialVersionUID = 1173984872572414699L;
        // 条件队列的第一个节点
          // 不要管这里的关键字 transient，是不参与序列化的意思
        private transient Node firstWaiter;
        // 条件队列的最后一个节点
        private transient Node lastWaiter;
        ......
```

之前介绍过，在AQS中有一个保存等待获取锁的线程的阻塞队列，这里引入另一个叫条件队列的东西，该条件队列是一个单向链表，每一个Condition都会有自己的一个条件队列。示意图如下：

![image](https://note.youdao.com/yws/api/personal/file/FEF7FAB1303842A9BD96D4BEF4D363CB?method=download&shareKey=762519de857b8731945055d9d89d4435)

## Condition执行流程
以生产者-消费者模式为例来说明，此处指的是最简单的流程，不考虑中断、signalAll、带超时参数等情况。

#### notFull条件：当生产者速度大于消费者速度时
1. 生产者首先获取关联的锁，然后会判断任务队列是否满，如果满了会调用await()方法等待不满条件满足，此时会挂起当前生产者线程，并将其加入和notFull的条件队列，等待唤醒，然后释放锁，注意此处是完全释放锁，因为锁是可重入的。
2. 唤醒操作通常由消费者线程来操作，唤醒前，也要先获取到锁，当消费者消费了一个任务时，会调用notFull的signal()，将和notFull的条件队列中的firstWaiter节点移到AQS的阻塞队列的队尾，等待获取锁，当获取锁成功await()方法返回，继续往下执行，将新生产的任务放入任务队列。

#### notEmpty条件：当消费者速度大于生产速度时
1. 消费者首先获取关联的锁，然后会判断任务队列是否有任务，如果没有会调用notEmpty的await()方法等待该条件满足，此时会挂起当前线程，并将其加入notEmpty的条件队列，等待唤醒，然后释放锁，注意此处是完全释放锁，因为锁是可重入的。
2. 此时唤醒该消费者由生产者来操作，前提也是要先获取关联的锁，当生产者生产一个任务放入任务队列后，此时会调用notEmpty的signal()方法，将notEmpty的条件队列的firstWaiter节点移到AQS的阻塞队列的队尾，然后该节点线程等待获取锁，当获取锁成功消费者的notEmpty的await()方法返回，继续往下执行，消费任务。