---
title: CP-16-用AQS实现的常用并发工具类
date: 2018-10-13 20:12:36
tags: ConcurrentPrograming
---

## CountDownLatch
CountDownLatch 这个类是比较典型的 AQS 的共享模式的使用，这是一个高频使用的类。

#### 使用场景
我们看下 Doug Lea 在CountDownLatch中的 java doc 中给出的例子，这个例子非常实用，我们经常会写这个代码。

假设我们有 N ( N > 0 ) 个任务，那么我们会用 N 来初始化一个 CountDownLatch，然后将这个 latch 的引用传递到各个线程中，在每个线程完成了任务后，调用 latch.countDown() 代表完成了一个任务。

在main方法中调用 latch.await() 的方法的线程会阻塞，直到所有的任务完成，才继续往后执行。
```
class Driver2 { // ...
    void main() throws InterruptedException {
        CountDownLatch doneSignal = new CountDownLatch(N);
        Executor e = Executors.newFixedThreadPool(8);

        // 创建 N 个任务，提交给线程池来执行
        for (int i = 0; i < N; ++i) // create and start threads
            e.execute(new WorkerRunnable(doneSignal, i));

        // 等待所有的任务完成，这个方法才会返回
        doneSignal.await();           // wait for all to finish
    }
}

class WorkerRunnable implements Runnable {
    private final CountDownLatch doneSignal;
    private final int i;

    WorkerRunnable(CountDownLatch doneSignal, int i) {
        this.doneSignal = doneSignal;
        this.i = i;
    }

    public void run() {
        try {
            doWork(i);
            // 这个线程的任务完成了，调用 countDown 方法
            doneSignal.countDown();
        } catch (InterruptedException ex) {
        } // return;
    }

    void doWork() { ...}
}
```
所以说 CountDownLatch 非常实用，我们常常会将一个比较大的任务进行拆分，然后开启多个线程来执行，等所有线程都执行完了以后，再往下执行其他操作。这里例子中，只有 main 线程调用了 await 方法。

我们再来看另一个例子，这个例子很典型，用了两个 CountDownLatch：
```
class Driver { // ...
    void main() throws InterruptedException {
        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(N);

        for (int i = 0; i < N; ++i) // create and start threads
            new Thread(new Worker(startSignal, doneSignal)).start();

        // 这边插入一些代码，确保上面的每个线程先启动起来，才执行下面的代码。
        doSomethingElse();            // don't let run yet
        // 因为这里 N == 1，所以，只要调用一次，那么所有的 await 方法都可以通过
        startSignal.countDown();      // let all threads proceed
        doSomethingElse();
        // 等待所有任务结束
        doneSignal.await();           // wait for all to finish
    }
}

class Worker implements Runnable {
    private final CountDownLatch startSignal;
    private final CountDownLatch doneSignal;

    Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
        this.startSignal = startSignal;
        this.doneSignal = doneSignal;
    }

    public void run() {
        try {
            // 为了让所有线程同时开始任务，我们让所有线程先阻塞在这里
            // 等大家都准备好了，再打开这个门栓
            startSignal.await();
            doWork();
            doneSignal.countDown();
        } catch (InterruptedException ex) {
        } // return;
    }

    void doWork() { ...}
}
```
这个例子中，doneSignal 同第一个例子的使用，我们说说这里的 startSignal。N 个新开启的线程都调用了startSignal.await() 进行阻塞等待，它们阻塞在栅栏上，只有当条件满足的时候（startSignal.countDown()），它们才能同时通过这个栅栏。

#### 执行流程
1. 构造时需要传入一个不小于0的整数，和ReentrantLock类似，其有一个内部类Sync继承自AQS，该初始值设置给了AQS的state属性。

```
// CountDownLatch的构造方法
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}

// CountDownLatch的内部类Sync的唯一构造方法
Sync(int count) {
    setState(count);
}
```
2. 调用了await()方法的线程都会被挂起，并放入AQS的阻塞队列中，等待唤醒。
3. 调用了countDown()方法的线程会将AQS的state值通过CAS减1，直到state等于0时，该线程负责唤醒所有调用await()方法的线程，从挂起处继续执行。

## CyclicBarrier
字面意思是“可重复使用的栅栏”，与CountDownLatch类似，都是需要多个线程都要到达某个条件时，然后再一起继续向下执行，它是 ReentrantLock 和 Condition 的组合实现。

#### 构造方法
```
public CyclicBarrier(int parties, Runnable barrierAction) {
}
 
public CyclicBarrier(int parties) {
}
```
1. 参数parties指让多少个线程或者任务等待至barrier状态；
2. 参数barrierAction为当这些线程都达到barrier状态时会执行的内容。

#### await()方法
CyclicBarrier中最重要的方法就是await方法，它有2个重载版本
```
public int await() throws InterruptedException, BrokenBarrierException { };
public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };
```

第一个版本比较常用，用来挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；

第二个版本是让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务。

#### 使用场景
如下有若干个线程都要进行写数据操作，并且只有所有线程都完成写数据操作之后，这些线程才能继续做后面的事情，此时就可以利用CyclicBarrier了
```
public class Test {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N,new Runnable() {
            @Override
            public void run() {
                System.out.println("当前线程"+Thread.currentThread().getName());   
            }
        });
         
        for(int i=0;i<N;i++)
            new Writer(barrier).start();
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }
 
        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println("所有线程写入完毕，继续处理其他任务...");
        }
    }
}

// 执行结果
线程Thread-0正在写入数据...
线程Thread-3正在写入数据...
线程Thread-2正在写入数据...
线程Thread-1正在写入数据...
线程Thread-2写入数据完毕，等待其他线程写入完毕
线程Thread-0写入数据完毕，等待其他线程写入完毕
线程Thread-3写入数据完毕，等待其他线程写入完毕
线程Thread-1写入数据完毕，等待其他线程写入完毕
当前线程Thread-3
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
```

![image](https://note.youdao.com/yws/api/personal/file/275EB05193D44EE589AD7409354DF42A?method=download&shareKey=8a542bc80bc329989c2cf0a4f3154da7)

#### 执行流程
首先CycleBarrier中的几个基本属性

```
public class CyclicBarrier {
    // 我们说了，CyclicBarrier 是可以重复使用的，我们把每次从开始使用到穿过栅栏当做"一代"
    private static class Generation {
        boolean broken = false;
    }

    /** The lock for guarding barrier entry */
    private final ReentrantLock lock = new ReentrantLock();
    // CyclicBarrier 是基于 Condition 的
    // Condition 是“条件”的意思，CyclicBarrier 的等待线程通过 barrier 的“条件”是大家都到了栅栏上
    private final Condition trip = lock.newCondition();

    // 参与的线程数
    private final int parties;

    // 如果设置了这个，代表越过栅栏之前，要执行相应的操作
    private final Runnable barrierCommand;

    // 当前所处的“代”
    private Generation generation = new Generation();

    // 还没有到栅栏的线程数，这个值初始为 parties，然后递减
    // 还没有到栅栏的线程数 = parties - 已经到栅栏的数量
    private int count;

    public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }

    public CyclicBarrier(int parties) {
        this(parties, null);
    }
```


1. 每个参与线程调用await()方法时，分两种情况，非最后一个到栅栏的线程和最后一个到达的。
2. 非最后一个到达的线程调用await()时，先把parties值减1，然后获取到CycleBarrier中的lock锁，然后再调用trip条件的await(),如果调用了CycleBarrier的超时版本await()，此处相应调用trip的超时版本await()方法，然后会将该线程放入trip的条件队列挂起，等待唤醒。此处超时后，如果最后一个线程都没到来，会打破栅栏，让所有等待线程继续执行。
3. 当最后一个线程到达时，会判断构造CycleBarrier时是否传了barrierAction，传了先执行它，然后执行trip的singalAll()方法并初始化下一代（parties复原值），将trip条件队列中所有等待线程全部转移到AQS的阻塞队列中，然后依次获取锁释放锁，继续执行。

## Semaphore
它类似一个资源池（读者可以类比线程池），每个线程需要调用 acquire() 方法获取资源，然后才能执行，执行完后，需要 release 资源，让给其他的线程用。

大概大家也可以猜到，Semaphore 其实也是 AQS 中共享锁的使用，因为每个线程共享一个池嘛。

#### 构造方法
```
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```
这里和 ReentrantLock 类似，用了公平策略和非公平策略。

此处构造Semaphore时，会将permits设置给AQS的state属性。

#### 运行流程
1. 先判断AQS的state属性减去当前线程要获取资源的个数是否小于0，小于0就会将该线程挂起放入AQS的阻塞队列，等待释放资源后被唤醒
2. 当有线程释放资源后，会CAS给state加上释放个数，然后唤醒阻塞队列的第一个节点线程。

#### 公平和非公平区别
唯一区别就是获取资源时，判读阻塞队列中是否等待线程，公平策略会乖乖去排队，非公平策略则会直接CAS获取。