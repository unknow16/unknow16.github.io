---
title: Dubbo-03-SPI-原理机制
date: 2018-05-11 14:52:34
tags: Dubbo
---

### JDK的SPI
[【传送门】](https://www.cnkirito.moe/spi/)


### Dubbo的SPI
* Dubbo的SPI的思想跟JDK的SPI是一样，但提供了更丰富的功能，如延时根据运行时协议加载具体实现类、AOP等。

* Dubbo中的SPI使用
```
Protocol protocol = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();  
```
这种代码用来获取一个接口的代理类，如以上代码返回Protocol接口的一个代理类实现，dubbo框架中有很多个Protocol接口的实现类，在程序运行时，如何确定使用哪个实现类呢？或者想扩展协议的，怎么加载我们自己的协议实现呢？

#### 1. 加载实现类

```
private static final String SERVICES_DIRECTORY = "META-INF/services/";  
private static final String DUBBO_DIRECTORY = "META-INF/dubbo/";   
private static final String DUBBO_INTERNAL_DIRECTORY = DUBBO_DIRECTORY + "internal/";  
```
ExtensionLoader类依次加载以上目录中以接口全类名作为文件名的文件中key=value形式的实现类到Map中。如下为Protocol接口的META-INF/dubbo/internal/com.alibaba.dubbo.rpc.Protocol文件内容：
```
registry=com.alibaba.dubbo.registry.integration.RegistryProtocol
filter=com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper
listener=com.alibaba.dubbo.rpc.protocol.ProtocolListenerWrapper
mock=com.alibaba.dubbo.rpc.support.MockProtocol
injvm=com.alibaba.dubbo.rpc.protocol.injvm.InjvmProtocol
dubbo=com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol
rmi=com.alibaba.dubbo.rpc.protocol.rmi.RmiProtocol
hessian=com.alibaba.dubbo.rpc.protocol.hessian.HessianProtocol
com.alibaba.dubbo.rpc.protocol.http.HttpProtocol
com.alibaba.dubbo.rpc.protocol.webservice.WebServiceProtocol
thrift=com.alibaba.dubbo.rpc.protocol.thrift.ThriftProtocol
memcached=memcom.alibaba.dubbo.rpc.protocol.memcached.MemcachedProtocol
redis=com.alibaba.dubbo.rpc.protocol.redis.RedisProtocol
```
然后，根据用户在运行时使用的协议作为key而调用相应的实现类。

#### 2. 生成的代理类一般如下：
* URL中携带了大部分的用户的参数配置，包括使用的协议

```
// 动态生成的协议代理类  
public class Protocol$Adpative implements com.alibaba.dubbo.rpc.Protocol {  
    
    public void destroy() {
        throw new UnsupportedOperationException("method public abstract void com.alibaba.dubbo.rpc.Protocol.destroy() of interface com.alibaba.dubbo.rpc.Protocol is not adaptive method!");  
    }  
    
    public int getDefaultPort() {
        throw new UnsupportedOperationException("method public abstract int com.alibaba.dubbo.rpc.Protocol.getDefaultPort() of interface com.alibaba.dubbo.rpc.Protocol is not adaptive method!");  
    }  
    
    public com.alibaba.dubbo.rpc.Exporter export(com.alibaba.dubbo.rpc.Invoker arg0) throws com.alibaba.dubbo.rpc.RpcException {  
        if (arg0 == null) throw new IllegalArgumentException("com.alibaba.dubbo.rpc.Invoker argument == null");  

        if (arg0.getUrl() == null) throw new IllegalArgumentException("com.alibaba.dubbo.rpc.Invoker argument getUrl() == null");  

        com.alibaba.dubbo.common.URL url = arg0.getUrl();  

        // 默认选择dubbo协议，否则根据url中带的协议属性来选择对应的协议处理对象，这样可以动态选择不同的协议  
        // 此处dubbo是@SPI注解中设置的，存在ExtensionLoader的cachedDefaultName成员变量中
        String extName = ( url.getProtocol() == null ? "dubbo" : url.getProtocol() ); <4>

        if(extName == null) throw new IllegalStateException("Fail to get extension(com.alibaba.dubbo.rpc.Protocol) name from url(" + url.toString() + ") use keys([protocol])");  

        // 根据拿到的协议key从缓存的map中取具体协议实现类对象  
        com.alibaba.dubbo.rpc.Protocol extension = (com.alibaba.dubbo.rpc.Protocol)ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.rpc.Protocol.class).getExtension(extName);  

        // 当前代理调用具体协议实现类对象的方法
        return extension.export(arg0);  
    }  
    
    public com.alibaba.dubbo.rpc.Invoker refer(java.lang.Class arg0, com.alibaba.dubbo.common.URL arg1) throws com.alibaba.dubbo.rpc.RpcException {  

        if (arg1 == null) throw new IllegalArgumentException("url == null");  

        com.alibaba.dubbo.common.URL url = arg1;  

        String extName = ( url.getProtocol() == null ? "dubbo" : url.getProtocol() );  

        if(extName == null) throw new IllegalStateException("Fail to get extension(com.alibaba.dubbo.rpc.Protocol) name from url(" + url.toString() + ") use keys([protocol])");  

        //根据拿到的协议key从缓存的map中取协议对象  
        com.alibaba.dubbo.rpc.Protocol extension = (com.alibaba.dubbo.rpc.Protocol)ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.rpc.Protocol.class).getExtension(extName);  
 
        // 当前代理调用具体协议实现类对象的方法
        return extension.refer(arg0, arg1);  
    }  
}  
```

Protocol extension = ExtensionLoader.getExtensionLoader(Protocol.class).getExtension(extName);  

* 代理中主要通过url中获取协议，再通过该代码获取具体协议实现，再调用具体实现相应方法。
* 如果url中获取不到协议，则默认使用dubbo协议，即使用DubboProtocol实现类。

#### 3. 配置默认实现类

当从url中获取不到协议时，默认采用什么实现类呢？

* 如下为Protocol接口的源码：
```
@SPI("dubbo") <1>
public interface Protocol {
    
    /**
     * 获取缺省端口，当用户没有配置端口时使用。
     * 
     * @return 缺省端口
     */
    int getDefaultPort();

    /**
     * 暴露远程服务：<br>
     * 1. 协议在接收请求时，应记录请求来源方地址信息：RpcContext.getContext().setRemoteAddress();<br>
     * 2. export()必须是幂等的，也就是暴露同一个URL的Invoker两次，和暴露一次没有区别。<br>
     * 3. export()传入的Invoker由框架实现并传入，协议不需要关心。<br>
     * 
     * @param <T> 服务的类型
     * @param invoker 服务的执行体
     * @return exporter 暴露服务的引用，用于取消暴露
     * @throws RpcException 当暴露服务出错时抛出，比如端口已占用
     */
    @Adaptive <2>
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;

    /**
     * 引用远程服务：<br>
     * 1. 当用户调用refer()所返回的Invoker对象的invoke()方法时，协议需相应执行同URL远端export()传入的Invoker对象的invoke()方法。<br>
     * 2. refer()返回的Invoker由协议实现，协议通常需要在此Invoker中发送远程请求。<br>
     * 3. 当url中有设置check=false时，连接失败不能抛出异常，并内部自动恢复。<br>
     * 
     * @param <T> 服务的类型
     * @param type 服务的类型
     * @param url 远程服务的URL地址
     * @return invoker 服务的本地代理
     * @throws RpcException 当连接服务提供方失败时抛出
     */
    @Adaptive <2>
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;

    /**
     * 释放协议：<br>
     * 1. 取消该协议所有已经暴露和引用的服务。<br>
     * 2. 释放协议所占用的所有资源，比如连接和端口。<br>
     * 3. 协议在释放后，依然能暴露和引用新的服务。<br>
     */
    void destroy();

}

```

* <1>处@SPI中指定了如果从url中获取不到协议key时，默认采用此处配置的key去找实现类。如上Protocol接口的代理类中<4>处"dubbo"字符串。
* <2>处标记@Adaptive注解的方法才会被代理类实现并代理，否则只是一个抛异常的空实现，如上Protocol接口的代理类,标该注解的会具体实现，没该注解的抛异常。

#### 4.补充细节
* 要求接口方法中必须要含有@Adaptive注解的方法，如果一个也没有@Adaptive注解的方法的方法，则不会生成代理类，抛出异常。
* 对于接口方法中不含@Adaptive注解的方法，全部是不可调用的，如上述destroy()方法实现中抛异常。
* 含有@Adaptive注解的方法必须含有URL类型的参数，或者能够获取到URL类型参数，如上述refer和export方法
* 从URL中根据什么参数key来获取用户传来的实现类标识字符串呢？以Protocol为例，参数key就是"protocol",默认是接口简单名称首字母小写，或者可以在@Adaptive注解中给出，注意@Adaptive注解的value为一个字符串数组，如下Transporter例子：
```
@SPI("netty") // 在配置文件以netty为key对应的实现类为默认实现类
public interface Transporter {

    // 服务提供端绑定监听,注解常量：server,transporter
    @Adaptive({Constants.SERVER_KEY, Constants.TRANSPORTER_KEY})
    Server bind(URL url, ChannelHandler handler) throws RemotingException;

    // 消费者端发起连接,注解常量：client,transporter
    @Adaptive({Constants.CLIENT_KEY, Constants.TRANSPORTER_KEY})
    Client connect(URL url, ChannelHandler handler) throws RemotingException;

}

// connect方法的扩展代理实现如下
public com.alibaba.dubbo.remoting.Client connect(com.alibaba.dubbo.common.URL arg0,com.alibaba.dubbo.remoting.ChannelHandler arg1) throws com.alibaba.dubbo.remoting.RemotingException{
    if (arg0 == null)  { 
        throw new IllegalArgumentException("url == null"); 
    }
    com.alibaba.dubbo.common.URL url = arg0;
    // 先以@Adaptive中的client为key从url中获取参数值，即配置文件中实现类对应的键，
    // 获取不到，再以第二个transporter为key从url中获取参数值，再获取不到则用@SPI中配置的默认的参数值。
    // 此处代码为最终生成的扩展代理类，以上逻辑在ExtensionLoader类中构造扩展代理类Code的方法createAdaptiveExtensionClassCode中
    String extName = url.getParameter("client", url.getParameter("transporter", "netty"));
    if(extName == null) {
        throw new IllegalStateException("Fail to get extension(com.alibaba.dubbo.remoting.Transporter) name from url(" + url.toString() + ") use keys([client, transporter])"); 
    }
    com.alibaba.dubbo.remoting.Transporter extension = (com.alibaba.dubbo.remoting.Transporter)com.alibaba.dubbo.common.ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.remoting.Transporter.class).getExtension(extName);
    return extension.connect(arg0, arg1);
}
```
* 从url中获取key=value参数值方法

```
/**
 * 获取协议
 */
 url.getProtocol()

/**
 * 获取method方法的以key为键的参数值
 */
url.getMethodParameter(String method, String key);

/**
 * 获取url中公共参数以key为键的参数值
 */
url.getParameter(String key);
```
* 消费者的dubbo-xml中配置参数

下列1，2，3为公共配置参数，4为方法中配置参数
```
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="echoService" interface="com.netease.dubbo.service.EchoService" check="false">

        <dubbo:parameter key="coreSize" value="10"/>
        <dubbo:parameter key="maximumSize" value="20"/>
        <dubbo:parameter key="keepAliveTimeMinutes" value="1"/>

        <!-- 1.等待超时时间，默认1秒 -->
        <dubbo:parameter key="timeoutInMilliseconds" value="1000"/>
        
        <!-- 2.1 断路器打开的错误条件百分比，默认50% -->
        <dubbo:parameter key="errorThresholdPercentage" value="50"/>
		
        <!-- 2.2 在滚动时间窗内，断路器熔断的最小请求数，默认20 -->
        <dubbo:parameter key="requestVolumeThreshold" value="20"/>
        
        <!-- 3. 断路器打开后的休眠时间窗，默认5秒 -->
        <dubbo:parameter key="sleepWindowInMilliseconds" value="5000"/>
        
        <!-- 4. 方法参数 -->
		<dubbo:method name="echoWithTimeOut">
			<!-- key固定， value为配置该方法的降级实现类在META-INF/dubbo/com.netease.hystrix.dubbo.rpc.filter.Fallback文件中的key -->
		    <dubbo:parameter key="fallback" value="fallbackImpl"/>
		</dubbo:method>
    </dubbo:reference>
```
