---
title: CP-18-ThreadPoolExecutor和Executors
date: 2018-10-13 20:13:17
tags: ConcurrentPrograming
---

## Executor继承图
![image](https://note.youdao.com/yws/api/personal/file/DFA26036607540F2933783E85B75945D?method=download&shareKey=2c473235cf560b2d380fd2ec54a69ad0)

- Executor接口很简单只有一个execute方法
- ExecutorService是Executor的子接口，主要增加了可以获取返回值的submit()方法
- AbstractExecutorService是一个抽象类
- ThreadPoolExecutor实现这个抽象类，是线程池的真正实现
- JDK 1.5新增

## ThreadPoolExecutor
#### 新建线程流程
* 在使用有界队列时，若有新的任务需要执行，如果线程池实际线程数小于corePoolSize，则优先创建线程，若大于corePoolSize，则会将任务加入队列，若队列已满，则在总线程数不大于maximumPoolSize的前提下，创建新的线程，若线程数大于maximumPoolSize，则执行拒绝策略或自定义处理方式。

* 使用无界队列时，当有新任务到来时，系统的线程数小于corePoolSize时，则新建线程执行任务，达到corePoolSize后，若后续仍有新的任务加入，而有没有空闲的线程资源，则任务直接进入队列等待，若任务创建和处理的速度差异很大，无界队列会保持快速增长，直到耗尽系统内存，因为采用无界队列，所以maxPoolSize无作用。

#### 构造方法

```
ThreadPoolExecutor(int corePoolSize,
                        int maximumPoolSize,
                        long keepAliveTime,
                        TimeUnit unit,
                        BlockingQueue<Runnable> workQueue) 
                        
ThreadPoolExecutor(int corePoolSize,
                        int maximumPoolSize,
                        long keepAliveTime,
                        TimeUnit unit,
                        BlockingQueue<Runnable> workQueue,
                        ThreadFactory threadFactory)

ThreadPoolExecutor(int corePoolSize,
                        int maximumPoolSize,
                        long keepAliveTime,
                        TimeUnit unit,
                        BlockingQueue<Runnable> workQueue,
                        RejectedExecutionHandler handler)           

ThreadPoolExecutor(int corePoolSize,
                        int maximumPoolSize,
                        long keepAliveTime,
                        TimeUnit unit,
                        BlockingQueue<Runnable> workQueue,
                        ThreadFactory threadFactory,
                        RejectedExecutionHandler handler)                        
```
#### 构造方法参数说明
* corePoolSize：线程池的核心线程数。

刚开始提交任务，每个任务创建一个线程，当任务数大于核心线程数，则放入阻塞队列。

* maximumPoolSize：线程池中允许的最大线程数。

如果阻塞队列满了，则继续创建新的线程，但要线程总数小于该值，如果再继续提交任务，则执行拒绝策略。

* keepAliveTime：线程空闲时的存活时间

该值在当前线程数大于corePoolSize时才有用

* unit：keepAliveTime的单位，如TimeUnit.SECONDS秒

* workQueue：当提交的任务数大于corePoolSize时，存放任务的阻塞队列

ArrayBlockingQueue: 基于数组的有界阻塞队列

LinkedBlockingQueue: 基于链表的有界阻塞队列

PriorityBlockQueue: 基于优先级的无界阻塞队列

SynchronousQueue: 不存储元素的阻塞队列

* ThreadFactory

线程工厂，提供创建新线程的功能，可以对线程的一些属性进行定制，是一个接口，默认的实现类是DefaultThreadFactory

* RejectedExecutionHandler

当新提交任务数达到最大线程数时，如果继续提交任务，将对该任务执行拒绝策略

默认线程池提供了4种拒绝策略，见下文，默认策略是直接抛出异常

```
ThreadPoolExecutor pool = new ThreadPoolExecutor(
			1, 				//coreSize, 调用prestartAllCoreThreads()方法，提前启动所有核心线程
			2, 				//MaxSize，当队列满时，新建线程数不能超过该数
			60, 			//空闲60秒的超出核心线程数的线程将被销毁
			TimeUnit.SECONDS,  //单位秒
			new ArrayBlockingQueue<Runnable>(3)			//指定一种队列 （有界队列）
			//new LinkedBlockingQueue<Runnable>()
			, new MyRejected()
			//, new DiscardOldestPolicy()
		);
```


## Executors工具类

Executors扮演线程工厂的角色，通过其可创建特定功能的线程池

#### newFixedThreadPool(int nThreads)

初始化一个指定线程数的线程池，其中corePoolSize == maximumPoolSize，使用LinkedBlockingQuene作为阻塞队列，不过当线程池没有可执行任务时，也不会释放线程。

```
public static ExecutorService new FixedThreadPool(int nThreads){
    return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable());
}
```
#### newCachedThreadPool()

初始化一个可以缓存线程的线程池，默认缓存60s, 线程池的线程数可达到Integer.MAX_VALUE，即2147483647，内部使用SynchronousQueue作为阻塞队列，所以使用该线程池要控制并发的任务数，否则创建大量线程会导致严重性能问题

开始提交任务时，因核心线程数为0，又因采用SynchronousQueue队列，所以将直接创建线程，任务执行完成后，线程空闲60秒后将被销毁回收。

```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```


#### newSingleThreadExecutor()

初始化的线程池中只有一个线程，如果该线程异常结束，会重新创建一个新的线程继续执行任务，唯一的线程可以保证所提交任务的顺序执行，内部使用LinkedBlockingQueue作为阻塞队列。

```
    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
```

#### newScheduledThreadPool()

初始化的线程池可以在指定的时间内周期性的执行所提交的任务，在实际的业务场景中可以使用该线程池定期的同步数据。

除了该类其他的线程池都是基于ThreadPoolExecutor类实现的。
	

```
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

ScheduledThreadPoolExecutor类继承ThreadPoolExecutor,下为其构造方法

public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```




## 拒绝策略
* AbortPolicy:直接抛出异常组织系统正常工作，默认策略
* CallerRunsPolicy:只要线程池未关闭，该策略直接在调用者线程中，运行当前被丢弃的任务
* DiscardOldestPolicy:丢弃最老的一个请求，尝试再次提交当前任务
* DiscardPolicy:丢弃无法处理的任务，不给任何处理
如果需要自定义拒绝策略可以实现RejectedExecutionHandler接口 
* 自定义实现RejectedExecutionHandler接口处理，如记录日志或持久化存储不能处理的任务

```
public interface RejectedExecutionHandler {
  void rejectedExecution(Runnable var1, ThreadPoolExecutor var2);
}
```

## 任务提交
Executors框架提供了两种提交任务的方式，可以根据不同业务需求选择不同方式

* Executor中唯一的execute(Runnable comond)方法

```
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```

该方式提交的任务不能获取返回值，因此无法判断任务是否执行成功

* ExecutorService中提供的submit

```
<T> Future<T> submit(Callable<T> task);
```
该方式可以通过Future的get()获取任务执行完成的返回值,该方法会阻塞知道任务返回结果

## ThreadFactory
可以对线程属性进行一些定制
```
// 接口
public interface ThreadFactory {
  Thread newThread(Runnable r);
}

// 默认实现类
static class DefaultThreadFactory implements ThreadFactory {
  private static final AtomicInteger poolNumber = new AtomicInteger(1);
  private final ThreadGroup group;
  private final AtomicInteger threadNumber = new AtomicInteger(1);
  private final String namePrefix;

  DefaultThreadFactory() {
      SecurityManager var1 = System.getSecurityManager();
      this.group = var1 != null?var1.getThreadGroup():Thread.currentThread().getThreadGroup();
      this.namePrefix = "pool-" + poolNumber.getAndIncrement() + "-thread-";
  }

  public Thread newThread(Runnable var1) {
      Thread var2 = new Thread(this.group, var1, this.namePrefix + this.threadNumber.getAndIncrement(), 0L);
      if(var2.isDaemon()) {
          var2.setDaemon(false);
      }

      if(var2.getPriority() != 5) {
          var2.setPriority(5);
      }

      return var2;
  }
}
```



