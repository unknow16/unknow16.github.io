---
title: Spring的事件驱动机制
date: 2018-03-01 09:27:52
tags: Spring
---

- 观察者模式：行为型设计模式，观察者列表由被观察者维护，为了在合适时机通知观察者。
- 发布订阅模式：需调度中心，先订阅，再发布。
- 事件驱动模型：以事件为中心交互，监听者监听事件，相应事件触发后会回调监听者


### JDK原生提供
* JDK1.0提供

目标（被观察者）：java.util.Observable，提供了目标需要的关键抽象：addObserver/deleteObserver/notifyObservers()等，具体请参考javadoc。

观察者：java.util.Observer，提供了观察者需要的主要抽象：
update(Observable o, Object arg)

* JDK1.1提供

事件：java.util.EventObject类

事件监听器：java.util.EventListener空接口

### Spring提供的事件驱动
#### 事件：ApplicationEvent抽象类
继承自JDK的EventObject，通过source可得到数据源，如AWT事件体系都是继承自它。

spring提供的ApplicationEvent的实现：
![image](https://note.youdao.com/yws/api/personal/file/5F3C76927C134E26A232287C07069F6A?method=download&shareKey=064c7284cd53b7f3ae581147708a91f0)

#### 事件发布者：ApplicationEventPublisher及ApplicationEventMulticaster接口
![image](https://note.youdao.com/yws/api/personal/file/BB48444311AA474B96BEE46D24B1EF89?method=download&shareKey=9b78344f28556bc94359b71850f7d4b0)
ApplicationContext接口继承了ApplicationEventPublisher，并在AbstractApplicationContext实现了具体代码，实际执行是委托给ApplicationEventMulticaster（可以认为是多播）：

我们常用的ApplicationContext都继承自AbstractApplicationContext，如ClassPathXmlApplicationContext、XmlWebApplicationContext等。所以自动拥有这个功能。

ApplicationContext自动到本地容器里找一个ApplicationEventMulticaster的实现，如果没有自己new一个SimpleApplicationEventMulticaster。其中SimpleApplicationEventMulticaster发布事件的代码如下：
```
1. public void multicastEvent(final ApplicationEvent event) {  
2.     for (final ApplicationListener listener : getApplicationListeners(event)) {  
3.         Executor executor = getTaskExecutor();  
4.         if (executor != null) {  
5.             executor.execute(new Runnable() {  
6.                 public void run() {  
7.                     listener.onApplicationEvent(event);  
8.                 }  
9.             });  
10.         }  
11.         else {  
12.             listener.onApplicationEvent(event);  
13.         }  
14.     }  
15. }  
```
大家可以看到如果给它一个executor（java.util.concurrent.Executor），它就可以异步支持发布事件了。否则就是同步发送。
 
所以我们发送事件只需要通过ApplicationContext.publishEvent即可，没必要再创建自己的实现了。除非有必要。

#### 监听器：ApplicationListener接口
继承自JDK的EventListener空接口，AWT体系也继承该类
```
1. public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {  
2.     void onApplicationEvent(E event);  
3. }
```
其只提供了onApplicationEvent方法，我们需要在该方法实现内部判断事件类型来处理，也没有提供按顺序触发监听器的语义，所以Spring提供了另一个接口，SmartApplicationListener：
```
1. public interface SmartApplicationListener extends ApplicationListener<ApplicationEvent>, Ordered {  
2.         //如果实现支持该事件类型 那么返回true  
3.     boolean supportsEventType(Class<? extends ApplicationEvent> eventType);  
4.     
5.         //如果实现支持“目标”类型，那么返回true  
6.     boolean supportsSourceType(Class<?> sourceType);  
7.          
8.         //顺序，即监听器执行的顺序，值越小优先级越高  
9.         int getOrder();  
10. }  
```
定义一个有序监听器：
```
1. @Component  
2. public class WangwuListener implements SmartApplicationListener {  
3.   
4.     @Override  
5.     public boolean supportsEventType(final Class<? extends ApplicationEvent> eventType) {  
6.         return eventType == ContentEvent.class;  
7.     }  
8.     @Override  
9.     public boolean supportsSourceType(final Class<?> sourceType) {  
10.         return sourceType == String.class;  
11.     }  
12.     @Override  
13.     public void onApplicationEvent(final ApplicationEvent event) {  
14.         System.out.println("王五在孙六之前收到新的内容：" + event.getSource());  
15.     }  
16.     @Override  
17.     public int getOrder() {  
18.         return 1;  
19.     }  
20. }  
```
张开涛老师：http://jinnianshilongnian.iteye.com/blog/1902886
