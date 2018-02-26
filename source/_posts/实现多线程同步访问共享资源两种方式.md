---
title: 实现多线程同步访问共享资源两种方式
date: 2018-02-26 17:43:32
tags: Concurrent
---

### 实现多线程同步访问共享资源两种方式
* synchronzied关键字+Object的wait/notify/notifyAll
* Lock体系+Condition条件谓词

### Synchronized
多个线程同步的依次访问它修饰的方法

对象锁或类锁由JVM自动释放。


### 对象锁
一个类中存在普通成员变量，多个线程中同时运行同一个类实例的实例方法，会共享访问普通成员变量，产生竞争，存在线程安全问题。需用实例对象作为锁同步控制。

如果多线程访问多个实例的实例方法，每个线程中的每个实例都有自己的成员变量，不存在多个线程共享情况，是线程安全的。

### 类锁
一个类中存在静态成员变量，多个线程同时运行一个或多个类实例的实例方法，都会共享访问静态成员变量，所以需要用类字节码作为类锁同步控制对共享静态变量的访问。


---

### Lock体系
Lock需程序员在finnally中释放
* void lock()

获取一个锁，如果该锁已被其他线程获取，则被挂起

* boolean tryLock()

尝试获取一个锁，获取到返回ture, 否则false

* boolean tryLock(long timeout, TimeUnit unit)

尝试获取锁，获取成功返回true, 否则尝试timeout给定时间，这段时间内能获取到则返回true，超时后返回false

* void lockInterruptibly()

如果获取锁定立即返回，否则当前线程处于休眠状态，直到获取到锁，休眠期间能被别的线程中断。

```
public class ReentrantLockTest3 {

	private ReentrantLock myLock = new ReentrantLock();
	
	private Condition condition = myLock.newCondition();
	
	private List<Integer> listBuffer = new ArrayList<Integer>();
	
	private volatile boolean runFlag = true;
	
	/**
	 * 生产者 生产数据
	 */
	public void produce() {
		int i = 0;
		while(runFlag) {
			myLock.lock();
			try {
				// 生产者检查容器中是否有数据，如果容器中有数据则生产者等待
				// 如果容器中没有数据则生产数据放入容器中并通知消费者
				if (listBuffer.size() > 0) {	
					try {
						// 调用await()方法，生产者线程阻塞并释放锁，之后进入该条件的等待集中
						// 直到消费者调用signalAll()方法之后，生产者线程解除阻塞并重新竞争锁
						// 生产者线程获得锁之后，重新开始从被阻塞的地方继续执行程序
						condition.await();
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				} else {
					System.out.println(Thread.currentThread().getName() + " add Integer");
					listBuffer.add(i++);
					// 生产者线程调用signalAll()方法，通知消费者线程容器中有数据
					condition.signalAll();
				}
			} finally {
				myLock.unlock();
			}			
		}
	}
	
	/**
	 * 消费者 读取数据
	 */
	public void consume() {
		while(runFlag) {
			myLock.lock();
			try {
				// 消费者检查容器中是否有数据，如果没有数据消费者等待
				// 如果容器中有数据则读取数据，读完之后通知生产者
				if (listBuffer.size() == 0) {
					try {
						// 同生产者线程一样，消费者线程调用await()方法阻塞并进入该条件等待集中
						condition.await();
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				} else {
					System.out.println(Thread.currentThread().getName() + " get Integer");
					long beginTime = 0;
					System.out.println(listBuffer.remove(0));
					beginTime = System.currentTimeMillis();
					while(System.currentTimeMillis() - beginTime < 100) {}
					// 消费者线程调用signalAll()方法，通知生产者生产数据
					condition.signalAll();
				}
			} finally {
				myLock.unlock();
			}		
		}		
	}
	
	
	
	public boolean isRunFlag() {
		return runFlag;
	}

	public void setRunFlag(boolean runFlag) {
		this.runFlag = runFlag;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		final ReentrantLockTest3 test = new ReentrantLockTest3();
		
		Thread produce = new Thread(new Runnable() {
			public void run() {
				test.produce();
			}
		},"A");
		
		Thread consume = new Thread(new Runnable() {
			public void run() {
				test.consume();
			}
		},"B");
		
		produce.start();
		consume.start();
		
		try {
			Thread.sleep(5000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		test.setRunFlag(false);
	}

}
```
