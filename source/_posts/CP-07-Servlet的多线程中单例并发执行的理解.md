---
title: CP-07-Servlet的多线程中单例并发执行的理解
date: 2018-01-19 20:01:48
tags: ConcurrentPrograming
---

### 每个请求创建一个线程
一般在一个web应用中会分为三层controller、service、dao，在不同层中编写对应的逻辑，下层通过接口向上层提供调用，程序从接收到用户请求到返回响应都运行在一个线程中。

因为每个用户请求都会创建一个新的线程，每个线程中，servlet容器都会创建一对servletRequest和servletResponse对象，两者不会被多线程共享，所以是线程安全的。

### 线程安全的定义
当多个线程访问某个类时，不管运行时环境采用何种调度方式，或者这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么就称这个类是线程安全的。（Java并发编程实战）

### Servlet的生命周期
Servlet是运行在Servlet容器中的，如：tomcat、jboss、weblogic、jetty，Servlet的生命周期也是由容器来管理的。

* 加载并实例化
* 初始化（init）
* 请求处理（service）
* 销毁（destroy）

### Servlet的线程安全
当客户端第一次请求Servlet时，容器会实例化Servlet。当又一个客户端请求Servlet时，不会再实例化Servlet，也就是多线程会采用同一个实例。Servlet容器默认采用单实例多线程处理多个请求。

servlet的方法中存在局部变量时：因为JVM是基于堆栈的虚拟机，JVM会为每个新建的线程分配一个独立的堆栈空间，方法内的局部变量存储在这个堆栈空间中，且参数传入是按值value copy的方式，所以是线程安全的。

servlet中存在实例成员变量时：由于servlet在servlet容器中是以单例存在的，所有的线程共享实例变量。多个线程对共享资源的访问就造成了线程不安全问题。

```
下面这个示例来自《Java并发编程实战》，在竞态条件下存在线程不安全。

public class UnsafeCountingFactorizer implements Servlet{
 
    private long count=0;
    
    public long getCount(){
        return count;
    }
 
    @Override
    public void service(ServletRequest req, ServletResponse resp) throws ServletException, IOException {
        BigInteger i=extractFromRequest();
        BigInteger[] factors=factor(i);
        ++count; //操作共享变量，非线程安全的
    }
}

递增操作count++并非是原子操作，它包含了三个独立的操作：读取count的值，将值加1，
然后将计算结果写入coune，这是一个“读取-修改-写入”的操作序列，并且其结果状态依赖于之前的状态。在执行时序不同的情况下，可能会产生错误。
```
