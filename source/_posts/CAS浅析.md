---
title: CAS浅析
date: 2018-02-26 14:14:17
tags: Concurrent
---

### 锁阻塞实现同步
独占锁：是一种悲观锁，如：synchronized,会导致其他需要锁的其他线程挂起，等待持有锁的线程释放。
乐观锁：每次假设没有冲突而去完成操作，当发生冲突时，就失败重试，知道成功为止。其使用的机制就是CAS。

### CAS
CAS(compare and swap),即比较并交换，是实现并发算法时常用的一种技术，Doug lea大神在Java同步器中大量使用了该技术。

CAS思想：三个值，一个内存当前值V，一个旧的预期值A，一个需要更新的值B，当且仅当V和A是否相等，相等则将V更新为B并返回true，否则什么也不做，并返回false。

实现语言层级非阻塞算法。在硬件层面是阻塞的，再加上volatile的特性实现。

而synchronized是阻塞的算法。

atomic包中的原子类是使用CAS来实现原子性操作的，实际是利用了cpu的CMPXCHG指令实现的，而该指令是一个原子性操作。

### CAS缺点
* ABA问题
描述：线程1读取内存当前值为A，然后被挂起，线程2读取当前值A，并修改为B，再修改为A，线程2再次被调度执行，会认为A没有被修改过。

针对这种情况，Java提供了一个AtomicStampedReference类，除了判断内存当前值和预期值相等外，还要判断当前标志和预期标志版本是否相等。它可以控制变量值的版本来保证CAS的正确性。

* 只能保证一个共享变量的原子操作

当对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁。

### JDK中的CAS方案

下面以AtomicInteger的实例为例

```
public class AtomicInteger extends Number implements java.io.Serializable {
    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset; //this当前值的内存偏移量

    static {
        try {
            //通过Unsafe获取value变量的内存地址，用于set/get值
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value; //this当前值, 保证了多线程间内存可见性
    public final int get() {return value;}
}
```
Unsafe，是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地方法（native）来访问，Unsafe相当于一个后门，通过该类可以直接操作特定内存的数据。


```
//返回旧值，然后增量加值
public final int getAndAdd(int delta) {    
    return unsafe.getAndAddInt(this, valueOffset, delta);
}

//unsafe.getAndAddInt       当前线程副本值  内存地址    增量新值
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5; //内存当前值
    do {
        var5 = this.getIntVolatile(var1, var2); //获取内存当前值
        
        //循环CAS
        //比较当前线程副本值和内存当前值是否相等，相等则交换为新值，
        //否则，当前线程应重新获取内存最新值，再交换新值
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```
1. AtomicInteger里面的value原始值为3，即当前主内存中的值为3，假设有两个线程共享，线程A和B同时有相同副本3.
2. 线程A执行到getIntVolatile(var1,var2),获取value为3，然后被挂起
3. 线程B也执行到getIntVolatile(var1,var2),获取value为3，幸运的是得以继续执行，比较当前内存值和线程副本值相同，则成功修改内存值为5。
4. 此时调度线程A继续执行，此时执行getCompareAndSwapInt(),发现内存当前值和线程A持有的副本值不一样，则执行getIntVolatile(),重新获取内存当前最新值，再进行比较，一样则执行相加，否则继续重新获取内存值。
5. 因为value是被volatile修饰的，所以当一个线程修改它的值后，其他线程总是能立即看到。整个过程使用CAS保证了对value并发修改的安全。
