---
title: Java基础细节
toc: true
date: 2020-03-21 21:03:14
tags:
categories:
---

## 基础

#### char的包装类Character

#### short a=1;a=a+1和a+=1的区别及+=的类型转换

```
short a=1;
//a= (a+1);//类型不匹配：不能从 int 转换为 short
a=(short) (a+1);
a+=1;
```

- 对于short a=1; a=a+1; 由于a+1运算时会自动提升表达式的类型，也即int类型，再将结果赋值给short类型的a时，类型会不匹配；

- 对于short a=1; a+=1; java编译器会对+=进行特殊处理，进行了类型转换，通过反编译.class源码可以看到a+=1被编译为：a=(short) (a+1)



#### == 和 equals 的区别是什么？

- 对于基本类型来说，没有equals方法，只能用==， 比较的是值
- 对于引用类型来说，==比较的是引用地址， equals默认情况下，比较的也是引用地址。但是我们可以重写equals方法，来达到比较值的目的。 其中有一个特殊的类 String， 在jdk源码里面已经重写了equals方法，实现了equals方法比较值。

Object类上的equals源码

```
 public boolean equals(Object obj) {
        return (this == obj);
  }
```

- 重写equals方法，一定要重写hashcode方法。 如果重写了equals，没有重写hashcode,会导致使用hash算法的集合出问题，比如HashMap 导致值找不到。
- 如果两个变量 equals返回true, 那么两个变量的hashcode一定相同。  两个变量的hashcode相同，equals方法不一定返回true.



#### hashCode方法作用

这个方法返回对象的散列码，返回值是int类型的散列码。

对象的散列码是为了更好的支持基于哈希机制的Java集合类，例如 Hashtable, HashMap, HashSet 等。



#### final 在 java 中有什么作用？

final作为Java中的关键字可以用于三个地方：用于修饰类、类属性和类方法。

- 修饰类：表示该类不能被继承；

- 修饰方法：表示方法不能被重写；

- 修饰变量：表示变量只能一次赋值以后值不能被修改，但是这里的"不能够被改变"对于不同的数据类型是有不同的含义的。当final修饰的是一个基本数据类型数据时, 这个数据的值在初始化后将不能被改变。当final修饰的是一个引用类型数据时, 也就是修饰一个对象时, 引用在初始化后将永远指向一个内存地址, 不可修改. 但是该内存地址中保存的对象信息, 是可以进行修改的。



#### Math.round(-1.5) 等于多少？

round:返回四舍五入，负.5小数返回较大整数，如-1.5返回-1。
ceil:返回小数所在两整数间的较大值，如-1.5返回-1。
tail:返回小数所在两整数间的较小值，如-1.5返回-2。





#### String字面量和new创建时一样吗？

- 前者字面量赋值时，JVM首先是在字符串常量池中找"i” 字符串，如果没有创建字符串常量，然后放到常量池中，若已存在，则直接引用
- 后者通过new，不检查常量池，直接要堆上创建一个新String对象



####  如何将字符串反转？

1. 使用 StringBuilder 或 StringBuffer 的 reverse 方法
2. 利用 String 的 toCharArray 方法先将字符串转化为 char 类型数组，然后将各个字符进行重新拼接
3. 利用 String 的 CharAt 方法取出字符串中的各个字符



#### 为什么要使用克隆？

想对一个对象进行处理，又想保存原对象的数据时，可以用克隆

#### 如何实现对象克隆？

Object 的 clone() 方法是浅拷贝，即如果类中属性有自定义引用类型，只拷贝引用，不拷贝引用指向的对象。

要实现深克隆，有两种方法：

1. 实现 Cloneable 接口，如果包含引用类型，其也要实现Cloneable接口，分别重写 clone() 方法，写代码实现克隆
2. 实现Serializable 接口，写方法序列化this，再读出来



#### 动态代理的应用

当想对某个类的方法之前或之后，做一些额外的处理，如日志、事务，就可以创建一个代理，它不仅包含被代理对象的功能，还可以做额外的处理，动态代理指的代理是在运行时动态生成的。

jdk动态代理： 基于接口 InvocationHandler     Proxy.newInstance()

cglib: 基于继承，通过字节码技术在运行时构造被代理的子类，子类中可以调父类方法，也可添加自己处理

## 容器



#### 如何解决hash冲突

hash冲突指的就是key的hashcode相同

1. 链地址法：它的基本思想是将冲突的实体链成一个单向链表，再将其头节点放入通过hashcode定位的哈希表的相应位置，适用于经常添加和删除的场景
2. 开放定址法：将key通过hash函数运算后，将值再次hash探测空地址，该hash函数中会有一个变量变化，根据该变量不同，又分为线性探测、二次方探测、随机探测
3. 再哈希法：准备多个hash函数，冲突时依次hash直到不冲突，元素不会聚集，但会增加计算时间
4. 将哈希表分为基础表和益处表，凡是发生冲突的放入溢出表



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
