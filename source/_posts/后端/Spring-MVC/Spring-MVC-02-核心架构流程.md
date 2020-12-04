---
title: Spring MVC-02-核心架构流程
date: 2018-02-28 14:53:26
tags: Spring MVC
---


![image](https://note.youdao.com/yws/api/personal/file/9C0D22885C74449C8173007E795D8B46?method=download&shareKey=c8f271df6561d0ce20451d3fde9f8eba)

### 核心架构的具体流程步骤如下：

1. 首先用户发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；

2. DispatcherServlet——>HandlerMapping， HandlerMapping将会把请求映射为HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象、多个HandlerInterceptor拦截器）对象，通过这种策略模式，很容易添加新的映射策略；

3. DispatcherServlet——>HandlerAdapter，HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；

4. HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个ModelAndView对象（包含模型数据、逻辑视图名）；

5. ModelAndView的逻辑视图名——> ViewResolver， ViewResolver将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；

6. View——>渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，因此很容易支持其他视图技术；

7. 返回控制权给DispatcherServlet，由DispatcherServlet返回响应给用户，到此一个流程结束。

---
此处我们只是讲了核心流程，没有考虑拦截器、本地解析、文件上传解析等，后边再细述。
* 在此我们可以看出具体的核心开发步骤：
1. DispatcherServlet在web.xml中的部署描述，从而拦截请求到Spring Web MVC
1. HandlerMapping的配置，从而将请求映射到处理器
1. HandlerAdapter的配置，从而支持多种类型的处理器
1. ViewResolver的配置，从而将逻辑视图名解析为具体视图技术
1. 处理器（页面控制器）的配置，从而进行功能处理


### 在web.xml中的配置
```
    <!-- springmvc的前端控制器 -->
    <servlet>
        <servlet-name>fuyi</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- 可配置多个，逗号分割 -->
            <param-value>classpath:spring-servlet-config.xml</param-value>
        </init-param>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>fuyi</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    
    <!-- 加载spring容器 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:spring/applicationContext*.xml</param-value>
	</context-param>
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
```

### 与Spring集成的上下文关系
![image](https://note.youdao.com/yws/api/personal/file/A0B0242FDADC41A89885F11F4F5C6E4A?method=download&shareKey=6cceec019c277f9f8bd18253c3b9529a)

- ContextLoaderListener初始化的上下文加载的Bean是对于整个应用程序共享的，不管是使用什么表现层技术，一般如DAO层、Service层Bean；
 
- DispatcherServlet初始化的上下文加载的Bean是只对Spring Web MVC有效的Bean，如Controller、HandlerMapping、HandlerAdapter等等，该初始化上下文应该只加载Web相关组件。