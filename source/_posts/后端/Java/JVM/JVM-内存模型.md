---
title: JVM-内存模型
date: 2018-01-12 17:12:35
tags: JVM
---

参考 

https://www.cnblogs.com/lishun1005/p/6019678.html

http://www.importnew.com/27454.html

### Java程序运行流程
![image](https://note.youdao.com/yws/api/personal/file/D96907FE11F14427BA6681B964FBCEFC?method=download&shareKey=e8d53230aec525655eb52c4ff4a02ecc)

### JVM内存模型
![image](https://note.youdao.com/yws/api/personal/file/CDA244EA5C364D1A9E8419A65293C2B9?method=download&shareKey=8a06f12346301e984cb8871b33dcf10f)

### 栈帧图
![image](https://note.youdao.com/yws/api/personal/file/7E061D6E7808411CA88435FDBDA98746?method=download&shareKey=6becf3898e6cd1201f8553b7bd4de97d)

### 方法区内存（多线程共享）/非堆内存/永久代
* 类字节码：运行Java程序时，会加载类的class字节码文件到内存中，会存储在这个区。
* 常量，如：字符串常量
* 类的静态成员变量
* 编译器编译后的类的方法的定义代码

JDK8,改名叫 Native Memory, 即Metaspace空间

### 堆内存（多线程共享）
* 对象的实例

JVM内存中最大的一块。

进行垃圾收集时，根据垃圾收集器的实现不同，又会细分为，新生代区、老生代区、永久代区。

## 每个线程都有下列内存区域
### 程序计数器内存
* 保存着当前线程下一条需要执行指令的地址

多个线程是并发执行的，但在单个线程内是顺序流执行，程序计数器就是来记录程序顺序执行的。

程序计数器是唯一个在JVM规范中没有规定内存溢出的。

### 本地方法栈内存
JVM中用来调用本地原生Native方法使用的栈内存区域。

HotSpot虚拟机之间将该内存区域和Java栈合并。

### Java栈内存/Java虚拟机栈内存
JVM中用来调用Java方法（即字节码方法）使用的栈内存区域。

* 栈帧

Java程序从Main方法开始单线程中顺序执行，同时伴随着方法的调用，执行到每个方法内时，它们都有自己的环境，这样的每个方法环境称为栈帧。

该Java虚拟机栈内存用来以栈的数据结构存储栈帧，每调用一个方法会发生一次栈帧的入栈，方法执行完毕，该栈帧出栈。

每个栈帧中包含方法局部变量表，操作数栈，动态链接，返回地址

编译时，方法内的局部变量及操作都已确定，所以当进入调用到该方法时，栈帧中的局部变量表的大小，操作数栈的大小都能确定，进而栈帧的大小也能确定。

方法的调用链可能会很深，但总是只有栈顶的栈帧处于执行状态，称为当前栈帧，该栈帧关联的方法称为当前方法，执行引擎执行的字节码指令只对当前栈帧执行操作。

* 局部变量表

基本类型直接存值，引用类型存引用

局部变量表的最小存储单位是一个Slot，一个Slot能存储32位内的数据类型

每个Slot能存储boolean、byte、short、int、float、char、reference、returnAddress类型的数据。

long、double需要两个Slot来存储，高位在前。

* 操作数栈

操作数栈的深度在编译时，被计算出来，写在字节码文件中。

操作数栈中的元素可以是Java任意类型，包括long，double，但每个该类型占2个栈容量。

* 动态链接
* 返回地址


