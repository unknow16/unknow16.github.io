---
title: 设计模式-行为型
date: 2018-02-26 23:04:47
tags: Java
---

### 策略模式

### 观察者模式

### 模版模式

---
### 责任链模式

### 状态模式

---

### 策略模式实现
* 定义策略接口

```
public interface Strategy {
   public void draw(int radius, int x, int y);
}
```
* 具体策略
```
public class RedPen implements Strategy {
   @Override
   public void draw(int radius, int x, int y) {
      System.out.println("用红色笔画图，radius:" + radius + ", x:" + x + ", y:" + y);
   }
}
public class GreenPen implements Strategy {
   @Override
   public void draw(int radius, int x, int y) {
      System.out.println("用绿色笔画图，radius:" + radius + ", x:" + x + ", y:" + y);
   }
}
public class BluePen implements Strategy {
   @Override
   public void draw(int radius, int x, int y) {
      System.out.println("用蓝色笔画图，radius:" + radius + ", x:" + x + ", y:" + y);
   }
}
```
* 使用策略的类

```
public class Context {
   private Strategy strategy;

   public Context(Strategy strategy){
      this.strategy = strategy;
   }

   public int executeDraw(int radius, int x, int y){
      return strategy.draw(radius, x, y);
   }
}
```
* 客户端调用

```
public static void main(String[] args) {
    Context context = new Context(new BluePen()); // 使用绿色笔来画
      context.executeDraw(10, 0, 0);
}
```

### 观察者模式
* 定义主题

```
public class Subject {

   private List<Observer> observers = new ArrayList<Observer>();
   private int state;

   public int getState() {
      return state;
   }

   public void setState(int state) {
      this.state = state;
      // 数据已变更，通知观察者们
      notifyAllObservers();
   }

   public void attach(Observer observer){
      observers.add(observer);        
   }

   // 通知观察者们
   public void notifyAllObservers(){
      for (Observer observer : observers) {
         observer.update();
      }
   }     
}
```
* 定义观察者接口

```
public abstract class Observer {
   protected Subject subject;
   public abstract void update();
}
```

* 具体的观察者

```
public class BinaryObserver extends Observer {

      // 在构造方法中进行订阅主题
    public BinaryObserver(Subject subject) {
        this.subject = subject;
        // 通常在构造方法中将 this 发布出去的操作一定要小心
        this.subject.attach(this);
    }

      // 该方法由主题类在数据变更的时候进行调用
    @Override
    public void update() {
        String result = Integer.toBinaryString(subject.getState());
        System.out.println("订阅的数据发生变化，新的数据处理为二进制值为：" + result);
    }
}

public class HexaObserver extends Observer {

    public HexaObserver(Subject subject) {
        this.subject = subject;
        this.subject.attach(this);
    }

    @Override
    public void update() {
          String result = Integer.toHexString(subject.getState()).toUpperCase();
        System.out.println("订阅的数据发生变化，新的数据处理为十六进制值为：" + result);
    }
}
```
* 客户端使用

```
public static void main(String[] args) {
    // 先定义一个主题
      Subject subject1 = new Subject();
      // 定义观察者
      new BinaryObserver(subject1);
      new HexaObserver(subject1);

      // 模拟数据变更，这个时候，观察者们的 update 方法将会被调用
      subject.setState(11);
}
```

### 模版模式实现
* 抽象类

```
public abstract class AbstractTemplate {
    // 这就是模板方法
      public void templateMethod(){
        init();
        apply(); // 这个是重点
        end(); // 可以作为钩子方法
    }
    protected void init() {
        System.out.println("init 抽象层已经实现，子类也可以选择覆写");
    }
      // 留给子类实现
    protected abstract void apply();
    protected void end() {
    }
}
```
* 实现

```
public class ConcreteTemplate extends AbstractTemplate {
    public void apply() {
        System.out.println("子类实现抽象方法 apply");
    }
      public void end() {
        System.out.println("我们可以把 method3 当做钩子方法来使用，需要的时候覆写就可以了");
    }
}
```
* 调用

```
public static void main(String[] args) {
    AbstractTemplate t = new ConcreteTemplate();
      // 调用模板方法
      t.templateMethod();
}
```






