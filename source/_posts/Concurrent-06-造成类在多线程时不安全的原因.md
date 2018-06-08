---
title: 造成类在多线程时不安全的原因
date: 2018-02-26 16:39:44
tags: Concurrent
---

### 线程安全的类： 不存在竞态条件（类中不存在被修改的成员变量），或存在时进行了同步控制。

### 多线程不安全的原因-竞态条件/临界区
同一个程序运行在多个线程中本身不会有线程安全问题，问题在于多个线程访问共享资源时存在，如：类成员变量（普通或静态变量），系统共享资源（文件，数据库）等。

同时只有多个线程同时对这些资源进行了写的操作时才会发生线程安全问题，不对资源的进行修改时就不会存在问题。


```
// 线程不安全的计数器
public class Counter {
    // 多个线程时存在的共享成员变量，产生竞态条件
    protected long count = 0;

    public void add(long value){
        //此处包含三个操作：1.获取count当前值 2.加上value值 3.将结果值赋给count
        this.count = this.count + value;  
    }
}
```
如：线程A和B同时执行同一个Counter实例的add（）方法，我们不知道操作系统何时在线程之间切换，JVM并不是将该方法当作单条指令执行的，而是按下列顺序执行的：

```
1. 从内存中获取this.count的值放到寄存器中
2. 将寄存器值加value
3. 将寄存器中的结果值写回内存
```

但多个线程执行时，会共享同一个实例的count成员变量，被调度执行时，可能会按照下列顺序执行：

```
A: 读取内存中this.count的值0到寄存器,被挂起
B: 读取内存中this.count的值0到寄存器
B: 将寄存器值加value=2
B: 将寄存器结果写回内存,此时内存中this.count值为2，执行结束
A: 再此调度A继续执行，将寄存器值加value=3
A: 将寄存器中结果写回内存，覆盖原来的结果值为3，执行结束
```

两个线程分别对count加2和3，两个线程执行结束后，应该为5，实际为3。在两个线程交叉执行时，读到的初始值都为0，分别写回2或3，后者覆盖前者，如果不对这样的多线程访问进行同步控制，就会造成这种线程不安全的结果。

* 竞态条件&临界区

当多个线程访问同一个资源时，对先后顺序敏感，就存在竞态条件。导致竞态条件发生的代码区称为临界区。

上例中add()方法是一个临界区，它会产生竞态条件。在临界区中使用适当的同步就可以避免竞态条件。

### 类中能造成线程不安全的共享资源
当多个线程访问共享资源变量时，并且进行了写操作，会引发竞态条件。同时读不会产生竞态条件。

* 方法中的局部基本类型变量

多线程中同时运行类的方法时（包括静态方法和成员方法），方法中局部变量会在每个线程的堆栈空间中存在副本，对它的修改不会影响其他线程，所以不存在线程安全问题，而成员变量根据不同情况会产生线程安全问题。

```
public void someMethod(){
  long threadSafeInt = 0;
  threadSafeInt++;
}
```

* 方法中的局部对象引用变量

对象引用存在每个线程的线程栈中，但new出来的对象实例在共享堆中，如果在某个方法中创建的局部对象不逃逸出该方法，则该类就是线程安全的。哪怕将该对象作为参数传递给其他方法，只要其他线程获取不到，就还是线程安全的。

逃逸：即该对象不会被其他方法获得，也不会被非局部变量引用。
```
public void someMethod(){
  LocalObject localObject = new LocalObject();
  localObject.callMethod();
  method2(localObject);
}

public void method2(LocalObject localObject){
  localObject.setValue("value");
}
```

样例中LocalObject对象没有被方法返回，也没有被传递给someMethod()方法外的对象。每个执行someMethod()的线程都会创建自己的LocalObject对象，并赋值给localObject引用。因此，这里的LocalObject是线程安全的。事实上，整个someMethod()都是线程安全的。即使将LocalObject作为参数传给同一个类的其它方法或其它类的方法时，它仍然是线程安全的。当然，如果LocalObject通过某些方法被传给了别的线程，那它就不再是线程安全的了。

* 对象成员变量

对象成员存储在堆上。如果两个线程同时更新同一个对象的同一个成员，那这个代码就不是线程安全的。

```
public class NotThreadSafe{
    StringBuilder builder = new StringBuilder();
    public add(String text){
        this.builder.append(text);
    }  
}
```
如果两个线程同时调用同一个NotThreadSafe实例上的add()方法，就会有竞态条件问题。

```
注意两个MyRunnable共享了同一个NotThreadSafe对象。
因此，当它们调用add()方法时会造成竞态条件。

NotThreadSafe sharedInstance = new NotThreadSafe();
new Thread(new MyRunnable(sharedInstance)).start();
new Thread(new MyRunnable(sharedInstance)).start();

public class MyRunnable implements Runnable{
  NotThreadSafe instance = null;
  
  public MyRunnable(NotThreadSafe instance){
    this.instance = instance;
  }
  
  public void run(){
    this.instance.add("some text");
  }
}
```


```
当然，如果这两个线程在不同的NotThreadSafe实例上调用call()方法，
就不会导致竞态条件。下面是稍微修改后的例子：

new Thread(new MyRunnable(new NotThreadSafe())).start();
new Thread(new MyRunnable(new NotThreadSafe())).start();
```
现在两个线程都有自己单独的NotThreadSafe对象，调用add()方法时就会互不干扰，再也不会有竞态条件问题了。所以非线程安全的对象仍可以通过某种方式来消除竞态条件。

* 线程控制逃逸规则

#### 线程安全的类：不包含竞态条件，即多线程时不存在共享资源变量或存在共享资源变量时进行了适当的同步控制

### 不可变对象保证线程安全
我们可以通过创建不可变的共享对象来保证对象在线程间共享时不会被修改，从而实现线程安全。如下示例：

```
请注意add()方法以加法操作的结果作为一个新的ImmutableValue类实例返回
而不是直接对它自己的value变量进行操作。

public class ImmutableValue{
    private int value = 0;
    
    public ImmutableValue(int value){
        this.value = value;
    }
    public int getValue(){
        return this.value;
    }
    
    public ImmutableValue add(int valueToAdd){
        return new ImmutableValue(this.value + valueToAdd);
    }
}
```
请注意ImmutableValue类的成员变量value是通过构造函数赋值的，并且在类中没有set方法。这意味着一旦ImmutableValue实例被创建，value变量就不能再被修改，这就是不可变性。但你可以通过getValue()方法读取这个变量的值。


* 即使一个对象是线程安全的不可变对象，但在另一个包含这个对象的一个引用的类中，该类可能不是线程安全的。

