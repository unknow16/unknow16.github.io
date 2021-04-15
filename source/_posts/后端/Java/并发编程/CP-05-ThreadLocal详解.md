---
title: CP-05-ThreadLocal详解
date: 2018-01-17 20:01:19
tags: ConcurrentPrograming
---


1. Thread中保存了一个ThreadLocal中的一个静态内部类ThreadLocalMap的实例对象threadLocals，默认为null。
2. 在一个线程中，new出第一个ThreadLocal时，并调用它的set方法会给当前线程中的threadLocals实例对象new出，key是当前ThreadLocal对象引用，值就是set方法的参数
3. new出第二个ThreadLocal对象时，重用当前线程的threadLocals实例对象，set时，key是此时的第二个ThreadLocal对象引用，值是set方法的值

### 理解
首先明确一个概念，ThreadLocal不是一个用来控制并发访问某个共享对象的，而是为每个线程分配一个只属于该线程的变量，顾名思义thread local variable(线程局部变量)。它的作用是会为每个使用该变量的线程提供一个该变量的副本，在每个线程中都可以独立的操作该变量，而不会影响其他线程，实现线程间的数据隔离。从线程的角度看，每个线程都完全拥有该变量。

这种变量在线程的生命周期内起作用，因为是线程的局部变量，让同一个线程内多个函数或者组件之间采用公共局部变量传递数据，减少复杂度。

> 举个例子，我出门需要先坐公交再做地铁，这里的坐公交和坐地铁就好比是同一个线程内的两个函数，我就是一个线程，我要完成这两个函数都需要同一个东西：公交卡（北京公交和地铁都使用公交卡），那么我为了不向这两个函数都传递公交卡这个变量（相当于不是一直带着公交卡上路），我可以这么做：将公交卡事先交给一个机构，当我需要刷卡的时候再向这个机构要公交卡（当然每次拿的都是同一张公交卡）。这样就能达到只要是我(同一个线程)需要公交卡，何时何地都能向这个机构要的目的。
有人要说了：你可以将公交卡设置为全局变量啊，这样不是也能何时何地都能取公交卡吗？但是如果有很多个人（很多个线程）呢？大家可不能都使用同一张公交卡吧(我们假设公交卡是实名认证的)，这样不就乱套了嘛。现在明白了吧？这就是ThreadLocal设计的初衷：提供线程内部的局部变量，在本线程内随时随地可取，隔离其他线程。

### 使用场景
* 数据库连接池实现

```
JDBC连接数据库：

Class.forName("com.mysql.jdbc.Driver");
java.sql.Connection conn = DriverManager.getConnection(jdbcUrl);
```
Connection的实例不是线程安全的，一个connection对应的是一个事务。DriverManager.getConnection(url)一次只能获取一个连接，另外每一个connection都需要浪费资源，所以诞生了数据库连接池。

> 数据库连接池实现原理：pool.getConnection()都是从ThreadLocal中获取的，如果存在连接就使用，保证线程中service的多个dao操作使用一个connection,以保证事务。如果不存在，就创建一个新的连接，放入到ThreadLocal中，在get()给到线程。将connection放到ThreadLocal,保证了每个线程从连接池中获取的connection都是自已的。

* 作为线程中局部公共变量，为不同函数的数据传递，避免形参传递数据的复杂性

* 提升性能和安全

如用SimpleDateFormat进行日期格式化。

一方面创建这个对象比较费时，应尽量避免进行多次创建，另一方面这个对象本身是线程不安全的，也不能缓存一个共享的实例。因此我们可以通过ThreadLocal为每个线程缓存一个SimpleDataFormat实例，提高性能。

* 简单使用例子：
```
public class TestThreadLocal {

    //创建线程局部变量
    private static final ThreadLocal<Integer> value = new ThreadLocal<Integer>() {
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };

    public static void main(String[] args) {
        //开启新线程，在每个线程中有独立的value对象
        for (int i = 0; i < 5; i++) {
            new Thread(new MyThread(i)).start();
        }
    }

    static class MyThread implements Runnable {
        private int index;

        public MyThread(int index) {
            this.index = index;
        }

        //在线程中操作value对象，不会影响其他线程value
        public void run() {
            System.out.println("线程" + index + "的初始value:" + value.get());
            for (int i = 0; i < 10; i++) {
                value.set(value.get() + i);
            }
            System.out.println("线程" + index + "的累加value:" + value.get());
        }
    }
}


执行结果为：线程0的初始value:0
            线程3的初始value:0
            线程2的初始value:0
            线程2的累加value:45
            线程1的初始value:0
            线程3的累加value:45
            线程0的累加value:45
            线程1的累加value:45
            线程4的初始value:0
            线程4的累加value:45
```


### 内存泄露
如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。

如果使用ThreadLocal的set方法之后，没有显示的调用remove方法，就有可能发生内存泄露，所以养成良好的编程习惯十分重要，使用完ThreadLocal之后，记得在创建ThreadLocal的线程中调用remove方法。

```
//一般在主线程中创建,或赋初始化值，使用时直接get()
ThreadLocal<String> tl = new ThreadLocal();

try {
    开启新线程，获取该副本，进行业务操作
    tl.set("fuyi");
    String val = tl.get();
    
    // 其它业务逻辑
} finally {
    // 用完之后，移除，以让GC回收
    tl.remove();
}

```


### 解析

每个Thread维护一个ThreadLocalMap映射表（变量名为threadLocals），这个映射表的key是ThreadLocal实例本身，value是真正需要存储的Object。

* set(T value)方法
```
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }


ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

//Thread类里默认threadLocals为null
class Thread implements Runnable{
    ThreadLocal.ThreadLocalMap threadLocals = null;
}

static class ThreadLocalMap {

        static class Entry extends WeakReference<ThreadLocal<?>> {

            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
  }


void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```
Thread.currentThread得到当前线程，如果当前线程存在threadLocals这个变量不为空，那么根据当前的ThreadLocal实例作为key寻找在map中位置，然后用新的value值来替换旧值。

在ThreadLocal这个类中比较引人注目的应该是ThreadLocal->ThreadLocalMap->Entry这个类。这个类继承自WeakReference。

* get()方法

```
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t); //返回t.threadLocals变量
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

首先我们通过Thread.currentThread得到当前线程，然后获取当前线程的threadLocals变量，这个变量就是ThreadLocalMap类型的,如果这个变量map不为空，再获取ThreadLocalMap.Entry e,如果e不为空，则获取value值返回，否则在Map中初始化Entry，并返回初始值null。如果map为空，则创建并初始化map，并返回初始值null。



