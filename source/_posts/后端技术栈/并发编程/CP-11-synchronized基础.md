---
title: CP-11-synchronized基础
date: 2018-10-13 20:11:29
tags: ConcurrentPrograming
---

#### 目录
1. 对象锁/类锁
1. 锁重入
1. 自动释放锁
1. 锁对象的改变问题

## 对象锁/类锁
对象锁：一个类中存在普通成员变量，多个线程中同时运行同一个类实例的实例方法，会共享访问普通成员变量，产生竞争，存在线程安全问题。需用实例对象作为锁同步控制。如果多线程访问多个实例的实例方法，每个线程中的每个实例都有自己的成员变量，不存在多个线程共享情况，是线程安全的。

类锁：一个类中存在静态成员变量，多个线程同时运行一个或多个类实例的实例方法，都会共享访问静态成员变量，所以需要用类字节码作为类锁同步控制对共享静态变量的访问。

https://www.cnblogs.com/paddix/p/5367116.html

```
public class ObjectLock {

	public void method1(){
		synchronized (this) {	//对象锁
			try {
				System.out.println("do method1..");
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public void method2(){		//类锁
		synchronized (ObjectLock.class) {
			try {
				System.out.println("do method2..");
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	private Object lock = new Object();
	public void method3(){		//任何对象锁
		synchronized (lock) {
			try {
				System.out.println("do method3..");
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	
	public static void main(String[] args) {
		
		final ObjectLock objLock = new ObjectLock();
		
		// 以objLock为锁，多个实例即多个锁
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				objLock.method1();
			}
		});
		
		// 以ObjectLock.class为锁，仅一个
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				objLock.method2();
			}
		});
		
		// 以new Object的lock为锁，仅一个
		Thread t3 = new Thread(new Runnable() {
			@Override
			public void run() {
				objLock.method3();
			}
		});
		
		// 以new Object的lock为锁，仅一个，会等待t3释放才能获取
		Thread t4 = new Thread(new Runnable() {
			@Override
			public void run() {
				objLock.method3();
			}
		});
		
		t1.start();
		t2.start();
		t3.start();
		t4.start();
		
		
	}
	
}
```


## 锁重入
也就是当一个线程得到了一个对象的锁后，再次请求此对象时是可以再次得到该对象的锁

```
// 调用方法重入
public class SyncDubbo1 {

	public synchronized void method1(){
		System.out.println("method1..");
		method2();
	}
	public synchronized void method2(){
		System.out.println("method2..");
		method3();
	}
	public synchronized void method3(){
		System.out.println("method3..");
	}
	
	public static void main(String[] args) {
		final SyncDubbo1 sd = new SyncDubbo1();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				sd.method1();
			}
		});
		t1.start();
	}
}

// 继承重入
public class SyncDubbo2 {

	static class Main {
		public int i = 10;
		public synchronized void operationSup(){
			try {
				i--;
				System.out.println("Main print i = " + i);
				Thread.sleep(100);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	static class Sub extends Main {
		public synchronized void operationSub(){
			try {
				while(i > 0) {
					i--;
					System.out.println("Sub print i = " + i);
					Thread.sleep(100);		
					this.operationSup();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				Sub sub = new Sub();
				sub.operationSub();
			}
		});
		
		t1.start();
	}
}
```

## 自动释放锁
1. 正常执行完synchronized块会自动释放锁
2. 执行时出现异常，也会自动释放锁

对于执行出现异常时，导致释放锁的情况，如果不及时处理，很可能对你的应用程序业务逻辑产生严重的错误。

比如你现在执行一个队列任务，很多对象都在等待第一个对象正确执行完毕再去释放锁，但是第一个对象由于异常的出现，导致业务逻辑没有正确执行完毕，就释放了锁，那可想而知后续的对象执行的都是错误的逻辑，这点需要考虑。

正确做法：
1. catch异常InterruptedException，该异常会打断此后的执行逻辑
2. 在catch中throw new RuntimeException(e);
```
public class SyncException {

	private int i = 0;	
	public synchronized void operation(){
		while(true){
			try {
				i++;
				Thread.sleep(100);
				System.out.println(Thread.currentThread().getName() + " , i = " + i);
				if(i == 10){
					Integer.parseInt("a");
					//throw new RuntimeException();
				}
			} catch (Exception e) { //Exception:后面逻辑会继续执行   InterruptedException 会打断此后的逻辑
				e.printStackTrace(); //根据任务之间是否有独立
				
				//throw new RuntimeException(e);
			}
		}
	}
	
	public static void main(String[] args) {
		
		final SyncException se = new SyncException();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				se.operation();
			}
		},"t1");
		t1.start();
	}
	
	
}

```

## 锁对象的改变问题
1. 字符串作为锁，在synchronized块中修改其值后，其他线程就能进来了
```
public class ChangeLock {

	private String lock = "lock";
	
	private void method(){
		synchronized (lock) {
			try {
				System.out.println("当前线程 : "  + Thread.currentThread().getName() + "开始");
				lock = "change lock"; //修改锁后，其他线程就能进来了
				Thread.sleep(2000);
				System.out.println("当前线程 : "  + Thread.currentThread().getName() + "结束");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
	
		final ChangeLock changeLock = new ChangeLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				changeLock.method();
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				changeLock.method();
			}
		},"t2");
		t1.start();
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t2.start();
	}
	
}
```
2. 同一对象属性的修改不会影响锁的情况
```
public class ModifyLock {
	
	private String name ;
	private int age ;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	
	public synchronized void changeAttributte(String name, int age) {
		try {
			System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 开始");
			this.setName(name);
			this.setAge(age);
			
			System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 修改对象内容为： " 
					+ this.getName() + ", " + this.getAge());
			
			Thread.sleep(2000);
			System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 结束");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		final ModifyLock modifyLock = new ModifyLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				modifyLock.changeAttributte("张三", 20);
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				modifyLock.changeAttributte("李四", 21);
			}
		},"t2");
		
		t1.start();
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t2.start();
	}
	
}
```


3. synchronized代码块对字符串的锁，注意String常量池的缓存功能，相当于类锁，如下示例，虽然是两个类实例调用，但字符串作为锁，相当于类锁，导致t2线程不能执行

```
public class StringLock {

	public void method() {
		//new String("字符串常量")
		synchronized ("字符串常量") {
			try {
				while(true){
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + "开始");
					Thread.sleep(1000);		
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + "结束");
					
					
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
		final StringLock stringLock = new StringLock();
		final StringLock stringLock1 = new StringLock();
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				stringLock.method();
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				stringLock1.method();
			}
		},"t2");
		
		t1.start();
		t2.start();
	}
}

```
