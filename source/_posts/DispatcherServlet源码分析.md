---
title: DispatcherServlet源码分析
date: 2018-02-28 14:53:47
tags: Spring MVC
---

### DispatcherServlet初始化
![image](https://note.youdao.com/yws/api/personal/file/6159259AF0E14C53BE5A3F14DB564681?method=download&shareKey=d32f14ea2b0ba4c8c310323c2e5a4346)
1. HttpServletBean继承HttpServlet，因此在Web容器启动时将调用它的init方法，该初始化方法的主要作用是将Servlet初始化参数（init-param）设置到该组件上（如contextAttribute、contextClass、namespace、contextConfigLocation），通过BeanWrapper简化设值过程，方便后续使用。并提供给子类初始化扩展点，initServletBean()，该方法由FrameworkServlet覆盖。

```
public abstract class HttpServletBean extends HttpServlet implements EnvironmentAware{
    @Override
    public final void init() throws ServletException {
       //省略部分代码
       //1、如下代码的作用是将Servlet初始化参数设置到该组件上
       //如contextAttribute、contextClass、namespace、contextConfigLocation；
       try {
           PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
           BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
           ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
           bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, this.environment));
           initBeanWrapper(bw);
           bw.setPropertyValues(pvs, true);
       }
       catch (BeansException ex) {
           //…………省略其他代码
       }
       //2、提供给子类初始化的扩展点，该方法由FrameworkServlet覆盖
       initServletBean();
       if (logger.isDebugEnabled()) {
           logger.debug("Servlet '" + getServletName() + "' configured successfully");
       }
    }
    //…………省略其他代码
}
```
2. FrameworkServlet继承HttpServletBean，通过initServletBean()进行Web上下文初始化，该方法主要覆盖一下两件事情
- 初始化web上下文
- 提供给子类初始化扩展点
```
public abstract class FrameworkServlet extends HttpServletBean {
    @Override
    protected final void initServletBean() throws ServletException {
        //省略部分代码
       try {
             //1、初始化Web上下文
           this.webApplicationContext = initWebApplicationContext();
             //2、提供给子类初始化的扩展点
           initFrameworkServlet();
       }
        //省略部分代码
    }
}
 

protected WebApplicationContext initWebApplicationContext() {
        //ROOT上下文（ContextLoaderListener加载的）
       WebApplicationContext rootContext =
              WebApplicationContextUtils.getWebApplicationContext(getServletContext());
       WebApplicationContext wac = null;
       if (this.webApplicationContext != null) {
           // 1、在创建该Servlet注入的上下文
           wac = this.webApplicationContext;
           if (wac instanceof ConfigurableWebApplicationContext) {
              ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
              if (!cwac.isActive()) {
                  if (cwac.getParent() == null) {
                      cwac.setParent(rootContext);
                  }
                  configureAndRefreshWebApplicationContext(cwac);
              }
           }
       }
       if (wac == null) {
             //2、查找已经绑定的上下文
           wac = findWebApplicationContext();
       }
       if (wac == null) {
            //3、如果没有找到相应的上下文，并指定父亲为ContextLoaderListener
           wac = createWebApplicationContext(rootContext);
       }
       if (!this.refreshEventReceived) {
             //4、刷新上下文（执行一些初始化）
           onRefresh(wac);
       }
       if (this.publishContext) {
           // Publish the context as a servlet context attribute.
           String attrName = getServletContextAttributeName();
           getServletContext().setAttribute(attrName, wac);
           //省略部分代码
       }
       return wac;
    }
```
从initWebApplicationContext（）方法可以看出，基本上如果ContextLoaderListener加载了上下文，该上下文将作为DispatcherServlet的根上下文（DispatcherServlet的父容器）。

最后调用了onRefresh()方法执行容器的一些初始化，这个方法由子类实现，来进行扩展。

3. DispatcherServlet继承FrameworkServlet，并实现了onRefresh()方法提供一些前端控制器相关的配置

```
public class DispatcherServlet extends FrameworkServlet {
     //实现子类的onRefresh()方法，该方法委托为initStrategies()方法。
    @Override
    protected void onRefresh(ApplicationContext context) {
       initStrategies(context);
    }
 
    //初始化默认的Spring Web MVC框架使用的策略（如HandlerMapping）
    protected void initStrategies(ApplicationContext context) {
       initMultipartResolver(context);
       initLocaleResolver(context);
       initThemeResolver(context);
       initHandlerMappings(context);
       initHandlerAdapters(context);
       initHandlerExceptionResolvers(context);
       initRequestToViewNameTranslator(context);
       initViewResolvers(context);
       initFlashMapManager(context);
    }
}
```
从如上代码可以看出，DispatcherServlet启动时会进行我们需要的Web层Bean的配置，如HandlerMapping、HandlerAdapter等，而且如果我们没有配置，还会给我们提供默认的配置。
 
4. 初始化总结
从如上代码我们可以看出，整个DispatcherServlet初始化的过程和做了些什么事情，具体主要做了如下两件事情：
- 初始化Spring Web MVC使用的Web上下文，并且**可能**指定父容器为（ContextLoaderListener加载了根上下文）
- 初始化DispatcherServlet使用的策略，如HandlerMapping、HandlerAdapter等。

### DispatcherServlet默认配置
DispatcherServlet的默认配置在DispatcherServlet.properties（和DispatcherServlet类在一个包下）中，而且是当Spring配置文件中没有指定配置时使用的默认策略。DispatcherServlet在启动时会自动注册这些特殊的Bean，无需我们注册，如果我们注册了，默认的将不会注册。

### DispatcherServlet的请求分派处理
![image](https://note.youdao.com/yws/api/personal/file/9C0D22885C74449C8173007E795D8B46?method=download&shareKey=c8f271df6561d0ce20451d3fde9f8eba)

1. 文件上传解析，如果请求类型是mutilpart将通过MultipartpartResolver进行文件上传解析。
2. 将请求通过HandlerMapping映射成HandlerExecutionChain(包含一个Handler和多个拦截器)
3. 将HandlerExecutionChain通过HandlerAdapter适配支持多种处理器。
4. 通过适配器调用用户定义的处理器的逻辑，返回ModelAndView
5. 通过ViewResolver解析逻辑视图名到具体实现。
6. 本地化解析
7. 如果执行过程中遇到异常，则交给HandlerExceptionResolver来解析。

#### 主要接口
* HandlerMapping：处理器映射器
* HandlerAdapter：处理器适配器
* ViewResolver：视图解析器（InternalResourceViewResolver）
* 
* MultipartResolver：文件上传解析器
* HandlerExceptionResolver: 处理器异常解析器
* 
* HandlerExecutionChain：包装用户处理器Handler + 多个处理器拦截器
* HandlerInterceptor: 处理器拦截器

```
//前端控制器分派方法  
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {  
        HttpServletRequest processedRequest = request;  
        HandlerExecutionChain mappedHandler = null;  
        int interceptorIndex = -1;  
  
        try {  
            ModelAndView mv;  
            boolean errorView = false;  
  
            try {  
                   //步骤1、检查是否是请求是否是multipart（如文件上传），如果是将通过MultipartResolver解析  
                processedRequest = checkMultipart(request);  
                   //步骤2、请求到处理器（页面控制器）的映射，通过HandlerMapping进行映射  
                mappedHandler = getHandler(processedRequest, false);  
                if (mappedHandler == null || mappedHandler.getHandler() == null) {  
                    noHandlerFound(processedRequest, response);  
                    return;  
                }  
                   //步骤3、处理器适配，即将我们的处理器包装成相应的适配器（从而支持多种类型的处理器）  
                HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());  
  
                  // 304 Not Modified缓存支持  
                //此处省略具体代码  
  
                // 执行处理器相关的拦截器的预处理（HandlerInterceptor.preHandle）  
                //此处省略具体代码  
  
                // 步骤4、由适配器执行处理器（调用处理器相应功能处理方法）  
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());  
  
                // Do we need view name translation?  
                if (mv != null && !mv.hasView()) {  
                    mv.setViewName(getDefaultViewName(request));  
                }  
  
                // 执行处理器相关的拦截器的后处理（HandlerInterceptor.postHandle）  
                //此处省略具体代码  
            }  
            catch (ModelAndViewDefiningException ex) {  
                logger.debug("ModelAndViewDefiningException encountered", ex);  
                mv = ex.getModelAndView();  
            }  
            catch (Exception ex) {  
                Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);  
                mv = processHandlerException(processedRequest, response, handler, ex);  
                errorView = (mv != null);  
            }  
  
            //步骤5 步骤6、解析视图并进行视图的渲染  
            //步骤5 由ViewResolver解析View（viewResolver.resolveViewName(viewName, locale)）  
            //步骤6 视图在渲染时会把Model传入（view.render(mv.getModelInternal(), request, response);）  
            if (mv != null && !mv.wasCleared()) {  
                render(mv, processedRequest, response);  
                if (errorView) {  
                    WebUtils.clearErrorRequestAttributes(request);  
                }  
            }  
            else {  
                if (logger.isDebugEnabled()) {  
                    logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + getServletName() +  
                            "': assuming HandlerAdapter completed request handling");  
                }  
            }  
  
            // 执行处理器相关的拦截器的完成后处理（HandlerInterceptor.afterCompletion）  
            //此处省略具体代码  
  
  
        catch (Exception ex) {  
            // Trigger after-completion for thrown exception.  
            triggerAfterCompletion(mappedHandler, interceptorIndex, processedRequest, response, ex);  
            throw ex;  
        }  
        catch (Error err) {  
            ServletException ex = new NestedServletException("Handler processing failed", err);  
            // Trigger after-completion for thrown exception.  
            triggerAfterCompletion(mappedHandler, interceptorIndex, processedRequest, response, ex);  
            throw ex;  
        }  
  
        finally {  
            // Clean up any resources used by a multipart request.  
            if (processedRequest != request) {  
                cleanupMultipart(processedRequest);  
            }  
        }  
    }  
```
