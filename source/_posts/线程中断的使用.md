---
title: 线程中断的使用
date: 2018-02-26 17:57:05
tags: Concurrent
---

当系统中的线程数大于cpu核心数时，就会通过切换不同线程以达到并行的效果。

直接忽略挂起状态的线程，但会顾及阻塞的线程，当阻塞的线程等待的资源就绪后，就可以转就绪状态，等待调度获取CPU执行时间片。

### 线程挂起（suspend）
线程的主动行为，因此恢复也要主动完成。

释放CPU使用权

如： LockSupport.park(Object obj) 挂起当前线程

### 线程阻塞（pend）
线程的被动行为，是在等待事件或资源时的表现，一旦获取到了资源就自动回到就绪状态。

释放CPU使用权

### 线程中断（interrupt）
每一个线程都有一个boolean类型标志，用来表明当前线程是否请求中断，当一个线程调用interrupt() 方法时，线程的中断标志将被设置为true。


```
// Thread 类中的实例方法，持有线程实例引用即可检测线程中断状态
public boolean isInterrupted() {}

// Thread 中的静态方法，检测调用这个方法的线程是否已经中断
// 注意：这个方法返回中断状态的同时，会将此线程的中断状态重置为 false
// 所以，如果我们连续调用两次这个方法的话，第二次的返回值肯定就是 false 了
public static boolean interrupted() {}

// Thread 类中的实例方法，用于设置一个线程的中断状态为 true
public void interrupt() {}
```


所以说调用线程的interrupt()方法不会中断一个正在运行的线程，这个机制只是设置了一个线程中断标志位，如果在程序中你不检测线程中断标志位，那么即使
设置了中断标志位为true，线程也一样照常运行。

#### 如果线程处于以下三种情况，那么当线程被中断的时候，能自动感知到：

* 来自 Object 类的 wait()、wait(long)、wait(long, int)，

* 来自 Thread 类的 join()、join(long)、join(long, int)、sleep(long)、sleep(long, int)

这几个方法的相同之处是，方法上都有: throws InterruptedException

如果线程阻塞在这些方法上（我们知道，这些方法会让当前线程阻塞），这个时候如果其他线程对这个线程进行了中断，那么这个线程会从这些方法中立即返回，抛出 InterruptedException 异常，同时重置中断状态为 false。

* 实现了 InterruptibleChannel 接口的类中的一些 I/O 阻塞操作，如 DatagramChannel 中的 connect 方法和 receive 方法等
如果线程阻塞在这里，中断线程会导致这些方法抛出 ClosedByInterruptException 并重置中断状态。

* Selector 中的 select 方法，这个有机会我们在讲 NIO 的时候说
一旦中断，方法立即返回

对于以上 3 种情况是最特殊的，因为他们能自动感知到中断（这里说自动，当然也是基于底层实现），并且在做出相应的操作后都会重置中断状态为 false。

那是不是只有以上 3 种方法能自动感知到中断呢？不是的，如果线程阻塞在 LockSupport.park(Object obj) 方法，也叫挂起，这个时候的中断也会导致线程唤醒，但是唤醒后不会重置中断状态，所以唤醒后去检测中断状态将是 true。

一般来说中断线程分为三种情况：
* (一) ：中断非阻塞线程
* (二)：中断阻塞线程
* (三)：不可中断线程


#### 中断非阻塞线程
中断非阻塞线程通常有两种方式：

(1)采用线程共享变量

这种方式比较简单可行，需要注意的一点是共享变量必须设置为volatile，这样才能保证修改后其他线程立即可见。


```
public class InterruptThreadTest extends Thread{  
  
    // 设置线程共享变量  
    volatile boolean isStop = false;  
      
    public void run() {  
        while(!isStop) {  
            long beginTime = System.currentTimeMillis();  
            System.out.println(Thread.currentThread().getName() + "is running");  
            // 当前线程每隔一秒钟检测一次线程共享变量是否得到通知  
            while (System.currentTimeMillis() - beginTime < 1000) {}  
        }  
        if (isStop) {  
            System.out.println(Thread.currentThread().getName() + "is interrupted");  
        }  
    }  
      
    public static void main(String[] args) {  
        // TODO Auto-generated method stub  
        InterruptThreadTest itt = new InterruptThreadTest();  
        itt.start();  
        try {  
            Thread.sleep(5000);  
        } catch (InterruptedException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
        // 线程共享变量设置为true  
        itt.isStop = true;  
    }  
  
}  
```
(2) 采用中断机制 

```
public class InterruptThreadTest2 extends Thread{  
    public void run() {  
        // 这里调用的是非清除中断标志位的isInterrupted方法  
        while(!Thread.currentThread().isInterrupted()) {  
            long beginTime = System.currentTimeMillis();  
            System.out.println(Thread.currentThread().getName() + "is running");  
            // 当前线程每隔一秒钟检测线程中断标志位是否被置位  
            while (System.currentTimeMillis() - beginTime < 1000) {}  
        }  
        if (Thread.currentThread().isInterrupted()) {  
            System.out.println(Thread.currentThread().getName() + "is interrupted");  
        }  
    }  
      
    public static void main(String[] args) {  
        // TODO Auto-generated method stub  
        InterruptThreadTest2 itt = new InterruptThreadTest2();  
        itt.start();  
        try {  
            Thread.sleep(5000);  
        } catch (InterruptedException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
        // 设置线程的中断标志位  
        itt.interrupt();  
    }  
}  
```

#### 中断阻塞线程

当线程调用Thread.sleep()、Thread.join()、object.wait()再或者调用阻塞的i/o操作方法时，都会使得当前线程进入阻塞状态。

那么此时如果在线程处于阻塞状态是调用interrupt()方法设置线程中断标志位时会出现什么情况呢！ 

此时处于阻塞状态的线程会抛出一个异常，并且会清除线程中断标志位(设置为false)。这样一来线程就能退出阻塞状态。

当然抛出异常的方法就是造成线程处于阻塞状态的Thread.sleep()、Thread.join()、object.wait()这些方法。
代码实例如下：

```
public class InterruptThreadTest3 extends Thread{  
      
    public void run() {  
        // 这里调用的是非清除中断标志位的isInterrupted方法  
        while(!Thread.currentThread().isInterrupted()) {  
            System.out.println(Thread.currentThread().getName() + " is running");  
            try {  
                System.out.println(Thread.currentThread().getName() + " Thread.sleep begin");  
                Thread.sleep(1000);  
                System.out.println(Thread.currentThread().getName() + " Thread.sleep end");  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                //由于调用sleep()方法清除状态标志位 所以这里需要再次重置中断标志位 否则线程会继续运行下去  
                Thread.currentThread().interrupt();  
                e.printStackTrace();  
            }  
        }  
        if (Thread.currentThread().isInterrupted()) {  
            System.out.println(Thread.currentThread().getName() + "is interrupted");  
        }  
    }  
      
    public static void main(String[] args) {  
        // TODO Auto-generated method stub  
        InterruptThreadTest3 itt = new InterruptThreadTest3();  
        itt.start();  
        try {  
            Thread.sleep(5000);  
        } catch (InterruptedException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
        // 设置线程的中断标志位  
        itt.interrupt();  
    }  
}  
```
需要注意的地方就是Thread.sleep()、Thread.join()、object.wait()这些方法，会检测线程中断标志位，如果发现中断标志位为true则抛出异常并且将中断标志位设置为false。

所以while循环之后每次调用阻塞方法后都要在捕获异常之后，调用Thread.currentThread().interrupt()重置状态标志位。

#### 不可中断线程

有一种情况是线程不能被中断的，就是调用synchronized关键字和reentrantLock.lock()获取锁的过程。

但是如果调用带超时的tryLock方法reentrantLock.tryLock(longtimeout, TimeUnit unit)，那么如果线程在等待时被中断，将抛出一个InterruptedException异常，这是一个非常有用的特性，因为它允许程序打破死锁。你也可以调用reentrantLock.lockInterruptibly()方法，它就相当于一个超时设为无限的tryLock方法。


```
public class InterruptThreadTest5 {  
      
    public void deathLock(Object lock1, Object lock2) {  
        try {  
            synchronized (lock1) {  
                System.out.println(Thread.currentThread().getName()+ " is running");  
                // 让另外一个线程获得另一个锁  
                Thread.sleep(10);  
                // 造成死锁  
                synchronized (lock2) {  
                    System.out.println(Thread.currentThread().getName());  
                }  
            }  
        } catch (InterruptedException e) {  
            System.out.println(Thread.currentThread().getName()+ " is interrupted");  
            e.printStackTrace();  
        }  
    }  
      
    public static void main(String [] args) {   
          
        final InterruptThreadTest5 itt = new InterruptThreadTest5();  
        final Object lock1 = new Object();  
        final Object lock2 = new Object();  
        Thread t1 = new Thread(new Runnable(){  
            public void run() {  
                itt.deathLock(lock1, lock2);  
            }  
        },"A");   
        Thread t2 = new Thread(new Runnable(){  
            public void run() {  
                itt.deathLock(lock2, lock1);  
            }  
        },"B");   
          
        t1.start();  
        t2.start();  
        try {  
            Thread.sleep(3000);  
        } catch (InterruptedException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
        // 中断线程t1、t2  
        t1.interrupt();  
        t2.interrupt();  
    }  
}  
```

