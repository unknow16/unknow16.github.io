---
title: Spring MVC-04-拦截器处理器-HandlerInterceptor
date: 2018-02-28 15:40:56
tags: Spring MVC
---

### 常见应用场景
1. 日志记录：记录请求信息的日志，以便进行信息监控、信息统计、计算PV（Page View）等。
2. 权限检查：如登录检测，进入处理器检测检测是否登录，如果没有直接返回到登录页面；
3. 性能监控：有时候系统在某段时间莫名其妙的慢，可以通过拦截器在进入处理器之前记录开始时间，在处理完后记录结束时间，从而得到该请求的处理时间（如果有反向代理，如apache可以自动记录）；
4. 通用行为：读取cookie得到用户信息并将用户对象放入请求，从而方便后续流程使用，还有如提取Locale、Theme信息等，只要是多个处理器都需要的即可使用拦截器实现。
5. OpenSessionInView：如Hibernate，在进入处理器打开Session，在完成后关闭Session。

* 本质也是AOP（面向切面编程），也就是说符合横切关注点的所有功能都可以放入拦截器实现。

### 拦截器接口

```
package org.springframework.web.servlet;  
public interface HandlerInterceptor {  
    //预处理回调方法，调用Controller业务方法前，如进行登录检查
    //返回ture,执行下一个拦截器
    //返回false,流程中断，不会继续调用其他拦截器或处理器，需用response产生响应。
    boolean preHandle(  
            HttpServletRequest request, HttpServletResponse response,   
            Object handler)   //Controller的实现类
            throws Exception;  
    
    //调用Controller之后的处理回调方法，在ViewResolver解析渲染视图之前
    //可对模型数据或视图进行处理
    void postHandle(  
            HttpServletRequest request, HttpServletResponse response,   
            Object handler,  //Controller的实现类
            ModelAndView modelAndView)   //模型数据和视图
            throws Exception;  
    
    //本次请求处理完成后回调方法，即视图也渲染完毕
    //如可以统计本次调用处理时间或清理一些资源
    //仅调用preHandle中返回ture的该方法
    void afterCompletion(  
            HttpServletRequest request, HttpServletResponse response,   
            Object handler, Exception ex)  
            throws Exception;  
} 
```

### 拦截器适配器
有时候我们可能只需要实现三个回调方法中的某一个，如果实现HandlerInterceptor接口的话，三个方法必须实现，不管你需不需要，此时spring提供了一个HandlerInterceptorAdapter适配器（一种适配器设计模式的实现），允许我们只实现需要的回调方法。

```
public abstract class HandlerInterceptorAdapter implements HandlerInterceptor {  
     //省略代码 此处所以三个回调方法都是空实现，preHandle返回true。  
} 
```
#### 正常流程
![image](https://note.youdao.com/yws/api/personal/file/798F0B8978174D299FE14AD493B1136E?method=download&shareKey=09f5bc05f14c7a5c2aedb2ca48e62e03)

#### 中断流程
![image](https://note.youdao.com/yws/api/personal/file/930A1756182246249E9BA6033CBA56FB?method=download&shareKey=b0aabc72c13a1d2470e7d34db41b9765)
中断流程中，比如是HandlerInterceptor2中断的流程（preHandle返回false），此处仅调用它之前拦截器的preHandle返回true的afterCompletion方法。

* 接下来看一下DispatcherServlet内部到底是如何工作的吧：
```
//doDispatch方法  简化代码
//1、处理器拦截器的预处理（正序执行）  
HandlerInterceptor[] interceptors = mappedHandler.getInterceptors();  
if (interceptors != null) {  
    for (int i = 0; i < interceptors.length; i++) {  
    HandlerInterceptor interceptor = interceptors[i];  
        if (!interceptor.preHandle(processedRequest, response, mappedHandler.getHandler())) {  
            //1.1、失败时触发afterCompletion的调用  
            triggerAfterCompletion(mappedHandler, interceptorIndex, processedRequest, response, null);  
            return;  
        }  
        interceptorIndex = i;//1.2、记录当前预处理成功的索引  
}  
}  
//2、处理器适配器调用我们的处理器  
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());  
//当我们返回null或没有返回逻辑视图名时的默认视图名翻译（详解4.15.5 RequestToViewNameTranslator）  
if (mv != null && !mv.hasView()) {  
    mv.setViewName(getDefaultViewName(request));  
}  
//3、处理器拦截器的后处理（逆序）  
if (interceptors != null) {  
for (int i = interceptors.length - 1; i >= 0; i--) {  
      HandlerInterceptor interceptor = interceptors[i];  
      interceptor.postHandle(processedRequest, response, mappedHandler.getHandler(), mv);  
}  
}  
//4、视图的渲染  
if (mv != null && !mv.wasCleared()) {  
render(mv, processedRequest, response);  
    if (errorView) {  
        WebUtils.clearErrorRequestAttributes(request);  
}  
//5、触发整个请求处理完毕回调方法afterCompletion  
triggerAfterCompletion(mappedHandler, interceptorIndex, processedRequest, response, null);  

// triggerAfterCompletion方法  
private void triggerAfterCompletion(HandlerExecutionChain mappedHandler, int interceptorIndex,  
            HttpServletRequest request, HttpServletResponse response, Exception ex) throws Exception {  
        // 5、触发整个请求处理完毕回调方法afterCompletion （逆序从1.2中的预处理成功的索引处的拦截器执行）  
        if (mappedHandler != null) {  
            HandlerInterceptor[] interceptors = mappedHandler.getInterceptors();  
            if (interceptors != null) {  
                //interceptorIndex为在preHandle时，记录返回ture的最后索引
                for (int i = interceptorIndex; i >= 0; i--) {  
                    HandlerInterceptor interceptor = interceptors[i];  
                    try {  
                        interceptor.afterCompletion(request, response, mappedHandler.getHandler(), ex);  
                    }  
                    catch (Throwable ex2) {  
                        logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);  
                    }  
                }  
            }  
        }  
    }  
```

### 添加配置
1. 实现HandlerInterceptor接口或继承HandlerInterceptorAdapter 
2. 如下在xml中配置
* 拦截器的执行顺序就是此处添加拦截器的顺序；
```
<mvc:interceptors>  
   <!--  使用bean定义一个Interceptor，直接定义在mvc:interceptors根下面的Interceptor将拦截所有的请求   -->
    <!-- <bean class="com.fuyi.web.interceptor.Login"/> 该bean会拦截所有请求 -->   
    <mvc:interceptor>  
        <!-- 进行拦截：/**表示拦截所有controller -->
        <mvc:mapping path="/**" />
    　　 <!-- 不进行拦截 -->
        <mvc:exclude-mapping path="/index.html"/>
       
        <bean class="com.fuyi.web.interceptor.Login"/>  
    </mvc:interceptor>  
</mvc:interceptors>  
```

### 性能监控应用
如记录一下请求的处理时间，得到一些慢请求（如处理时间超过500毫秒），从而进行性能改进，一般的反向代理服务器如apache都具有这个功能，但此处我们演示一下使用拦截器怎么实现。
- 实现分析：

1. 在进入处理器之前记录开始时间，即在拦截器的preHandle记录开始时间；

2. 在结束请求处理之后记录结束时间，即在拦截器的afterCompletion记录结束实现，并用结束时间-开始时间得到这次请求的处理时间。

- 问题：
我们的拦截器是单例，因此不管用户请求多少次都只有一个拦截器实现，即线程不安全，那我们应该怎么记录时间呢？
- 解决方案：是使用ThreadLocal，它是线程绑定的变量，提供线程局部变量（一个线程一个ThreadLocal，A线程的ThreadLocal只能看到A线程的ThreadLocal，不能看到B线程的ThreadLocal）。

代码实现：

```
package cn.javass.chapter5.web.interceptor;  
public class StopWatchHandlerInterceptor extends HandlerInterceptorAdapter {  
    private NamedThreadLocal<Long>  startTimeThreadLocal = new NamedThreadLocal<Long>("StopWatch-StartTime");  
    
    @Override  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {  
        long beginTime = System.currentTimeMillis();//1、开始时间  
        startTimeThreadLocal.set(beginTime);//线程绑定变量（该数据只有当前请求的线程可见）  
        return true;//继续流程  
    }  
      
    @Override  
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {  
        long endTime = System.currentTimeMillis();//2、结束时间  
        long beginTime = startTimeThreadLocal.get();//得到线程绑定的局部变量（开始时间）  
        long consumeTime = endTime - beginTime;//3、消耗的时间  
        if(consumeTime > 500) {//此处认为处理时间超过500毫秒的请求为慢请求  
            //TODO 记录到日志文件  
            System.out.println(  
            String.format("%s consume %d millis", request.getRequestURI(), consumeTime));  
        }          
    }  
} 
```
NamedThreadLocal：Spring提供的一个命名的ThreadLocal实现。

在测试时需要把stopWatchHandlerInterceptor放在拦截器链的第一个，这样得到的时间才是比较准确的。

### 登录检测应用
在访问某些资源时（如订单页面），需要用户登录后才能查看，因此需要进行登录检测。

- 流程：
1. 访问需要登录的资源时，由拦截器重定向到登录页面；
2. 如果访问的是登录页面，拦截器不应该拦截；
3. 用户登录成功后，往cookie/session添加登录成功的标识（如用户编号）；
4. 下次请求时，拦截器通过判断cookie/session中是否有该标识来决定继续流程还是到登录页面；
5. 在此拦截器还应该允许游客访问的资源。

拦截器代码如下所示：

```
@Override  
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {  
    //1、请求到登录页面 放行  
    if(request.getServletPath().startsWith(loginUrl)) {  
        return true;  
    }  
          
    //2、TODO 比如退出、首页等页面无需登录，即此处要放行 允许游客的请求  
          
    //3、如果用户已经登录 放行    
    if(request.getSession().getAttribute("username") != null) {  
        //更好的实现方式的使用cookie  
        return true;  
    }  
          
    //4、非法请求 即这些请求需要登录后才能访问  
    //重定向到登录页面  
    response.sendRedirect(request.getContextPath() + loginUrl);  
    return false;  
}  
```
提示：推荐能使用servlet规范中的过滤器Filter实现的功能就用Filter实现，因为HandlerInteceptor只有在Spring Web MVC环境下才能使用，因此Filter是最通用的、最先应该使用的。如登录这种拦截器最好使用Filter来实现。
