---
title: JVM-类加载机制
date: 2018-01-12 17:14:14
tags: JVM
---

## 简介

指将*.class字节码文件通过类加载器加载到JVM内存模型的方法区和堆中。

在堆内存中生成一个Class对象，供程序访问。

JVM规范规定类加载器预料到要使用某个类时，可以提前加载该类字节码，如果该类字节码存在错误，也只能等到使用它时，才能发现，类加载器不会报错。

## 类加载顺序
![image](https://note.youdao.com/yws/api/personal/file/6C85DC063AB74A3DB0E1110168BA692D?method=download&shareKey=6d71d5931cebfe5917424161fa82bbe5)

其中加载、验证、准备、初始化一定会发生，但解析不一定，这是为了支持Java语言的运行时绑定。且一定是顺序开始的，但不一定顺序结束，因为一个阶段的执行可能会调用或激活另一个阶段。

#### 1. 加载
查找并加载一个类的字节码进入JVM内存

1. 根据类的全限定名查找类的字节码文件二进制流
2. 将静态的字节码二进制流转换存入方法区内存中的动态数据结构
3. 创建一个Class对象实例，作为访问方法区类数据结构的入口。

#### 2. 链接
* 验证：验证类字节码文件是否符合规范，且不危害JVM运行。
* 准备：为类的静态变量分配内存，并初始化为默认值。0/null，而不是程序中指定的初始化值。
* 解析：把常量池中的符号引用替换成直接引用。直接引用就是直接指向目标的指针。

#### 3. 初始化
为类的静态变量赋予指定的值。包括静态代码块赋值和静态变量直接赋值。
```
public static int value; //默认值为0，可不初始化值。
public static final int value = 1; //必须为其初始化值，否则编译不通过。
public final int value; //系统不会赋予默认值，必须显式的赋值。

static final常量在准备阶段时，就被初始化为代码中指定的值。
```

## 类加载器

类加载器负责加载所有的类进入JVM，每个加载到JVM中的类，都有一个java.lang.Class的实例，并且保证该类只会被加载一次，已经加载过的类不会再加载。


![image](https://note.youdao.com/yws/api/personal/file/2F6F4734FCA74EE28C2FA9DD12892246?method=download&shareKey=3c4eed16235abdc5de0b479f7685b67f)


每个class都有一个reference，指向自己的ClassLoader。
array的ClassLoader就是其元素的ClassLoader，若是基本数据类型，则这个array没有ClassLoader，Java代码中，通过一个类的全限定名（包名和类名）标识一个类，JVM中一个类的唯一标识是：一个类的全限定名 + 加载该类的类加载器，如：同一个类通过不同的ClassLoader加载到JVM中，是不同的类，则会生成不同的java.lang.Class实例。

* 根类加载器

负责加载Java核心类, %JAVA_HOME%/jre/lib

* 扩展类加载

负责加载jre中ext下的扩展类jar包, %JAVA_HOME%/jre/lib

* 系统类/应用类加载器

负责加载classpath下用户自己编写的类。

如果应用程序没有自定义过自己的类加载器，一般默认就是这个类加载器。

* 除了根类加载器，其它的都是ClassLoader抽象类的子类。

## 类加载机制
* 全盘负责

一个类被某个类加载器加载后，该类所依赖的其他类也由该类加载器加载

* 父类委托

加载一个类时，先委托父类加载，父类找不到时，才在自己classpath下加载

* 缓存机制

加载一个类前，先查询是否已加载过该类，加载过直接使用，否则再加载。

#### 加载流程
1. 调用 findLoadedClass 来查看是否存在已装入的类。  
1. 如果没有，那么采用某种特殊的神奇方式来获取原始字节。（通过IO从文件系统，来自网络的字节流等）  
1. 如果已有原始字节，调用 defineClass 将它们转换成 Class 对象。  
1. 如果没有原始字节，然后调用 findSystemClass 查看是否从本地文件系统获取类。  
1. 如果 resolve 参数是 true，那么调用 resolveClass 解析 Class 对象。  
1. 如果还没有类，返回 ClassNotFoundException。  
1. 否则，将类返回给调用程序。  

#### Tomcat类加载机制
* 自定义了自己的类加载器。
* 采用代理，每个应用一个类加载器，先在应用中寻找加载类，找不到委托给父类，跟JDK默认的机制相反。
* 保证了JDK核心类的安全。

在Tomcat 里的Web应用中，WebApp的ClassLoader的工作原理有点不同，它先试图自己载入类（在ContextPath/WEB-INF/...中载入类），如果无法载入，再请求父ClassLoader完成。  由此可得：  

1. 对于WEB APP线程，它的contextClassLoader是WebAppClassLoader  
1. 对于Tomcat Server线程，它的contextClassLoader是CatalinaClassLoader  

#### 线程上下文类加载器
线程上下文类加载器（context class loader）是从 JDK 1.2 开始引入的。类 java.lang.Thread中的方法 getContextClassLoader()和 setContextClassLoader(ClassLoader cl)用来获取和设置线程的上下文类加载器。如果没有通过 setContextClassLoader(ClassLoader cl)方法进行设置的话，线程将继承其父线程的上下文类加载器。Java 应用运行的初始线程的上下文类加载器是系统类加载器。在线程中运行的代码可以通过此类加载器来加载类和资源。

实际上，在Java应用中所有程序都运行在线程里，如果在程序中没有手工设置过ClassLoader，对于一般的java类如下两种方法获得的ClassLoader通常都是同一个  
1. this.getClass.getClassLoader()；  
1. Thread.currentThread().getContextClassLoader()；  

方法1得到的Classloader是静态的，表明类的载入者是谁；方法2得到的Classloader是动态的，谁执行（某个线程），就是那个执行者的 Classloader。对于单例模式的类，静态类等，载入一次后，这个实例会被很多程序（线程）调用，对于这些类，载入的Classloader和执行 线程的Classloader通常都不同。  

## 获得ClassLoader的3种方法
1. this.getClass.getClassLoader(); // 使用当前类的ClassLoader  
1. Thread.currentThread().getContextClassLoader(); // 使用当前线程的ClassLoader  
1. ClassLoader.getSystemClassLoader(); // 使 用系统ClassLoader，即系统的入口点所使用的ClassLoader。（注意，system ClassLoader与根 ClassLoader并不一样。JVM下system ClassLoader通常为App ClassLoader）  

## 定制ClassLoader的应用  
1. 安全性。类进入JVM之前先经过ClassLoader，所以可以在这边检查是否有正确的数字签名等  
1. 加密。java字节码很容易被反编译，通过定制ClassLoader使得字节码先加密防止别人下载后反编译，这里的ClassLoader相当于一个动态的解码器  
1. 归档。可能为了节省网络资源，对自己的代码做一些特殊的归档，然后用定制的ClassLoader来解档  
1. 自展开程序。把java应用程序编译成单个可执行类文件，这个文件包含压缩的和加密的类文件数据，同时有一个固定的ClassLoader，当程序运行时它在内存中完全自行解开，无需先安装  
1. 动态生成。可以生成应用其他还未生成类的类，实时创建整个类并可在任何时刻引入JVM  


## 加载类字节码到JVM的3种方法
一个应用程序总是由n多个类组成，Java程序启动时，并不是一次把所有的类全部加载后再运行，它总是先把保证程序运行的基础类一次性加载到jvm中，其它类等到jvm用到的时候再加载，这样的好处是节省了内存的开销，因为java最早就是为嵌入式系统而设计的，内存宝贵，这是一种可以理解的机制，而用到时再加载这也是java动态性的一种体现

1. 直接new，隐式装载
```
  B b = new B(); 
```
隐式装载：程序在运行过程中当碰到通过new 等方式生成对象时，隐式调用类装载器加载对应的类到jvm中。 

2. 使用Class静态方法 Class.forName  显式装载

```
Class cls = Class.forName("com.rain.B"); 
    B b = (B)cls.newInstance();
```


3. 使用ClassLoader  显式装载
    
```
// Get ClassLoader  参考上面三种方法获取
ClassLoader cl;

// Load the class 使用第一步得到的ClassLoader来载入B
Class cls = cl.loadClass("com.rain.B"); 
```

## 文件载入方法
1. 直接IO 
```
String path = ""; // 使用绝对路径或相对路径
File f = new File(path); 
InputStream is = new FileInputStream(f); 
```
如果是配置文件，可以通过java.util.Properties.load(is)将内容读到Properties里，Properties默认认为is的编码是ISO-8859-1，如果配置文件是非英文的，可能出现乱码问题。  

2. 使用ClassLoader  


```
/** 
 * 因为有3种方法得到ClassLoader，对应有如下3种方法读取文件 
 * 使用的路径是相对于这个ClassLoader的那个点的相对路径，此处只能使用相对路径 
 */ 
String path = ""; //    相对路径 
InputStream is = null; 
//方法1 
is = this.getClass().getClassLoader().getResourceAsStream(path); 
//方法2
is = Thread.currentThread().getContextClassLoader().getResourceAsStream(path);  
//方法3
is = ClassLoader.getSystemResourceAsStream("com/rain/config/sys.properties");
```

如果是配置文件，可以通过java.util.Properties.load(is)将内容读到Properties里，这里要注意编码问题。

3. 使用ResourceBundle  


```
ResourceBundle bundle = ResourceBundle.getBoundle("com.rain.config.sys");
``` 
这种用法通常用来载入用户的配置文件，关于ResourceBunlde更详细的用法请参考其他文档  