---
title: Dubbo-05-SPI-@Adaptive注解
date: 2018-05-11 14:53:05
tags: Dubbo
---

## 为什么要设计Adaptive？注解在类上和注解在方法上的区别？
adaptive设计的目的是为了识别固定已知类和扩展未知类。

#### 注解在类上
代表人工实现，实现一个装饰类（设计模式中的装饰模式），它主要作用于固定已知类，
目前整个系统只有2个，AdaptiveCompiler、AdaptiveExtensionFactory。
    
1. 为什么AdaptiveCompiler这个类是固定已知的？

因为整个框架仅支持Javassist和JdkCompiler。

2. 为什么AdaptiveExtensionFactory这个类是固定已知的？

因为整个框架仅支持2个objFactory,一个是SPI,另一个是Spring。

#### 注解在方法上
代表自动生成和编译一个动态的Procotol$Adpative类，它主要是用于SPI，因为SPI的类是不固定、未知的扩展类，所以设计了动态$Adaptive类，然后可以在运行时通过从url中获取协议key获取具体实现类，该代理会调用具体实现类相应方法。

例如 Protocol的SPI类有 injvm dubbo registry filter listener等等 很多扩展未知类，通过下列代码获取代理类Protocol$Adaptive

```
Protocol protocol = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();  
```
在代理类中再通过从url中获取的协议名获取具体实现类，让代理来调用。

```
ExtensionLoader.getExtensionLoader(Protocol.class).getExtension(extName);
```

#### @Adaptive注解源码

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface Adaptive {
    
    /**
     * 从{@link URL}的Key名，对应的Value作为要Adapt成的Extension名。
     * <p>
     * 如果{@link URL}这些Key都没有Value，使用 用 缺省的扩展（在接口的{@link SPI}中设定的值）。<br>
     * 比如，<code>String[] {"key1", "key2"}</code>，表示
     * <ol>
     * <li>先在URL上找key1的Value作为要Adapt成的Extension名；
     * <li>key1没有Value，则使用key2的Value作为要Adapt成的Extension名。
     * <li>key2没有Value，使用缺省的扩展。
     * <li>如果没有设定缺省扩展，则方法调用会抛出{@link IllegalStateException}。
     * </ol>
     * <p>
     * 如果不设置则缺省使用Extension接口类名的点分隔小写字串。<br>
     * 即对于Extension接口{@code com.alibaba.dubbo.xxx.YyyInvokerWrapper}的缺省值为<code>String[] {"yyy.invoker.wrapper"}</code>
     * 
     * @see SPI#value()
     */
    String[] value() default {};
    
}
```
