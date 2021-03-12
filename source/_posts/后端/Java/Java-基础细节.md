---
title: Java-基础细节
toc: true
date: 2020-03-21 21:03:14
tags:
categories:
---



#### 抽象类相关问题
- 抽象类必须要有抽象方法吗？  不需要，抽象类不一定非要有抽象方法。
- 普通类和抽象类有哪些区别？ 1.普通类不能包含抽象方法，抽象类可以包含抽象方法。2.抽象类不能直接实例化，普通类可以直接实例化。
- 抽象类能使用 final 修饰吗？ 不能，定义抽象类就是让其他类继承的，如果定义为 final 该类就不能被继承，这样彼此就会产生矛盾，所以 final 不能修饰抽象类
- 接口和抽象类有什么区别？
实现：抽象类的子类使用 extends 来继承；接口必须使用 implements 来实现接口。

构造函数：抽象类可以有构造函数；接口不能有。

main 方法：抽象类可以有 main 方法，并且我们能运行它；接口不能有 main 方法。

实现数量：类可以实现很多个接口；但是只能继承一个抽象类。

访问修饰符：接口中的方法默认使用 public 修饰；抽象类中的方法可以是任意访问修饰符。

### 容器


#### List、Set、Map 之间的区别是什么？

- List:  存储有序、元素可重复
- Set:  存储有无序和有序的实现，元素不可重复，底层基于Map的key唯一实现
- Map: 存储kv，key唯一，有无序和有序的实现



#### HashMap 和 Hashtable 有什么区别？

- 前者线程不安全，后者线程安全，只是给方法加了synchroized关键字，效率低，应该采用ConcurrentHashMap
- 前者允许null作为键,存储在数组中的第0个索引处，后者不允许



#### ArrayList 和 Vector 的区别是什么？

- 后者是前者的线程安全版本
- 扩容，前者扩容0.5，后者扩容1倍



#### 如何决定使用 HashMap 还是 TreeMap？

1. 两者都是线程不安全的
2. 前者key无序，后者key可以排序
3. 前者通过hash表实现，查询、删除、插入都有更好性能，没有排序需求时，尽量使用前者
4. 前者可以通过初始容量和负载因子调优，后者采用红黑树实现，始终保持平衡状态不能调优

#### ArrayList 和 LinkedList 的区别是什么？

- ArrayList: 底层数组实现，适合快速随机查询
- LinkedList: 底层采用双向链表实现，适合快速插入和删除



#### 如何实现数组和 List 之间的转换？

- List转数组：toArray(arraylist.size()方法

- 数组转List：Arrays的asList(a)方法



#### 迭代器 Iterator 是什么？

它是为容器提供的公共的访问接口，它不用关心容器底层实现，用同一种方式遍历容器中的元素，



#### 怎么确保一个集合不能被修改？

采用Collections包下的unmodifiableXXX方法进行包装



## 阻塞队列

java.util.concurrent 中加入了 BlockingQueue 接口和五个阻塞队列类。它实质上就是一种带有一点扭曲的 FIFO 数据结构。不是立即从队列中添加或者删除元素，线程执行操作阻塞，直到有空间或者元素可用。

#### 常见队列

- ArrayBlockingQueue: 一个由数组支持的有界队列。
- LinkedBlockingQueue: 一个由链表实现的可选有界或无界队列
- PriorityBlockingQueue: 一个由优先级堆支持的无界优先级队列
- DelayQueue:一个由优先级堆支持的、基于时间的调度队列。
- SynchronousQueue: 一个只能包含一个元素的阻塞队列

#### 取元素方法区别

- add、remove、element: 分别是添加、移除、返回头元素，失败会抛异常
- offer、poll、peek: 分别是添加、移除并返回头元素、返回头元素，失败时分别返回false，null, null
- put、take: 添加、移除，失败时会阻塞线程



## 并发

#### atomic变量的实现原理

它是一个乐观锁的实现，底层通过volatile关键字和CAS实现修改值，获取变量地址和偏移量，通过CAS修改，如果失败则自旋再次重试。

ABA问题，采用版本号，每次修改增加版本号



#### synchronized 和 volatile 的区别是什么？

volatile保证内存可见性，JVM内存模型规范描述，线程执行时，会将主存中数据复制到线程本地内存中，操作完成之后再写回主存，volatile变量的读写都会被直接写进主存，但不能保证原子性操作。

synchronized则是会获取对象监视器锁，然后保护的代码块不能被其他线程访问，直到释放锁，同时还会创建一个内存屏障，保证线程的所有操作都会直接刷到主存中，从而也保证了内存可见性



## SpringCloud

#### 微服务优缺点

1. 每个微服务模块只做一件事，高内聚，对外隐藏实现
2. 可以根据不同模块业务特性，采用不同编程语言开发
3. 每个模块根据请求和负载需求，弹性横向扩展

- 增加部署的复杂度
- 分布式事务、数据一致性
- 调用链路复杂，出现问题不好定位
- 服务间通讯成本



#### eureka 的自我保护机制

短时间内接受不到实例的心跳时，会进入自我保护机制，保护注册的实例信息而不删除，故障恢复时自动退出保护模式。



#### SpringCloud

它是基于SpringBoot的，提供了一套开发微服务系统所需要的一系列基础组件，如下：

- 注册中心-服务发现：eureka   nacos  zookeeper consol
- 客户端负载均衡、服务降级限流、断路器、服务网关、链路追踪、配置管理、Feign调用



#### SpringCloud与Dubbo区别

1. 调用方式：基于http的RestApi   基于TCP的RPC效率更高
2. 生态完整性：dubbo只是实现了服务治理，springCloud较全





## MySQL

#### 两阶段提交

MySQL使用两阶段提交主要解决 binlog 和 InnoDB redo log 的数据一致性的问题

阶段一：先写redo log, 此时InnoDB进入prepare阶段

阶段二：如果redo log写成功，则开始写binlog，成功后则进入commit阶段，提交实际是在redo log末尾写一个记录



## 分布式常见问题

#### 幂等性

1. 查询、删除是天然的幂等的

2. 唯一索引

3. 分布式锁

4. 状态机

5. 悲观锁for update

6. 乐观锁version

   ```
   1. 通过版本号实现
   update table_xxx set name=#name#,version=version+1 where version=#version
   2. 通过条件限制 
   update table_xxx set avai_amount=avai_amount-#subAmount# where avai_amount-#subAmount# >= 0
   ```

#### 分布式事务

## 参考资料

> - []()
> - []()
