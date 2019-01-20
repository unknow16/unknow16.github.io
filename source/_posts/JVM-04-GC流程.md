---
title: JVM-04-GC流程
date: 2018-01-19 23:44:21
tags: JVM
---

Java内存模型中提到有一块堆内存， 我们知道它是被所有线程共享的一块内存区域，所有对象实例和数组都在堆上进行内存分配。为了高效地进行垃圾回收，JVM又把堆内存存，细分为新生代、老生代、永久代。

## 新生代
新生代又被细分为Eden和Survivor Space，后者又分为S0和S1。Eden和Survivor的默认比例是8:1。如新生代为10M 时，Eden分配8M，S0和S1各分配1M。

## Minor GC
大多数情况下，对象在Eden中分配，当其大小不够时，会发生Minor GC。

Minor GC发生时，会把Eden中存活的对象移到S0中，并清空Eden，当Minor GC再次发生时，会把Eden和S0中存活的对象移到S1中。

存活的对象会在S0和S1之间移动，当存活对象从Eden移到Survivor或在Survivor之间移动时，存活对象的GC年龄会进行累加，当GC年龄达到默认15时，该对象会被移动到老生代，

## 老生代
老生代用于存放经过几次Minor GC仍然存活的对象，当老生代内存不足时，会发生Major GC/Full GC，速度一般比Minor GC慢10倍以上。

## 永久代
永久代是Java1.8之前的HotSpot实现中的称呼。

类的元数据如方法数据、方法信息（字节码、栈、变量大小）、运行时常量等都被保存在永久代中，32位JVM默认64M，64位默认85M，一旦类的元数据超过了永久代大小，就会报OOM异常。

## Metaspace/堆外内存/元空间
Java1.8中将之前永久代从Java堆内存中移除，不占用JVM内存，并把类元数据直接保存在本地内存区域，其可用最大内存为操作系统的物理内存，从而避免了OOM。

如果不设置JVM将会根据一定的策略自动增加本地元内存空间。 如果你设置的元内存空间过小，你的应用程序可能得到以下错误： java.lang.OutOfMemoryError: Metadata space

## JVM内存调整参数
-Xmx=2048M # 最大内存

-Xmn  # 新生代内存大小

-XX:SurvivorRatio  # 指定Eden和Survivor的比例

-XX:+PrintGCDetails   # 打印内存回收日志

-XX:MaxTenuringThreshold #  新生代存活对象的年龄阀值，达到会被移到老生代

-XX:MaxPermSize # Java1.8之前永久代的大小

-XX:MaxMetaspaceSize=128m # 设置Java1.8的堆外内存