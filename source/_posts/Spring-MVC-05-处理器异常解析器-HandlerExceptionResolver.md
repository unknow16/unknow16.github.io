---
title: Spring MVC-05-处理器异常解析器-HandlerExceptionResolver
date: 2018-02-28 16:25:30
tags: Spring MVC
---

Springmvc 通过HandlerExceptionResolver接口 处理程序的异常，包括Handler映射，数据绑定以及目标方法执行时发生的异常。

### HandlerExceptionResolver接口

```

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Interface to be implemented by objects that can resolve exceptions thrown during
 * handler mapping or execution, in the typical case to error views. Implementors are
 * typically registered as beans in the application context.
 *
 * <p>Error views are analogous to JSP error pages but can be used with any kind of
 * exception including any checked exception, with potentially fine-grained mappings for
 * specific handlers.
 *
 * @author Juergen Hoeller
 * @since 22.11.2003
 */
public interface HandlerExceptionResolver {

	/**
	 * Try to resolve the given exception that got thrown during handler execution,
	 * returning a {@link ModelAndView} that represents a specific error page if appropriate.
	 * <p>The returned {@code ModelAndView} may be {@linkplain ModelAndView#isEmpty() empty}
	 * to indicate that the exception has been resolved successfully but that no view
	 * should be rendered, for instance by setting a status code.
	 * @param request current HTTP request
	 * @param response current HTTP response
	 * @param handler the executed handler, or {@code null} if none chosen at the
	 * time of the exception (for example, if multipart resolution failed)
	 * @param ex the exception that got thrown during handler execution
	 * @return a corresponding {@code ModelAndView} to forward to, or {@code null}
	 * for default processing
	 */
	ModelAndView resolveException(
			HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex);

}
```
* 实现类

![image](https://note.youdao.com/yws/api/personal/file/8F88FA342F244104B1F3CDD028208E7E?method=download&shareKey=914787c23d3fc7b73954778a7f689409)

### ExceptionHandlerExceptionResolver

**当请求的目标方法出现了异常时**
1. 先在当前控制器中，查找注解了@ExceptionHandler的方法，如果存在进行异常匹配，匹配成功则调用该方法处理异常，如下：

```
@Controller
public class SpringTest {

    //value可以指定多个异常类型
    @ExceptionHandler(value = {ArithmeticException.class})
    public String handleArithmeticException(Exception ex) {
        System.out.println("spring test class");
        ex.printStackTrace();
        return "error";
    }
}
```

2. 如果找不到@ExceptionHandler注解的方法，就会找注解@ControllerAdvice的类中的注解@ExceptionHandler的方法,如果存在进行异常匹配，匹配成功则调用该方法处理异常，如下：
```
@ControllerAdvice
public class HandlerExceptionController {
    
    @ExceptionHandler({ArithmeticException.class,IOException.class})
    public ModelAndView TestExceptionHandlerExceptionResolver(Exception ex){
        System.out.println("03----ControllerAdvice出异常了："+ex);
        
        ModelAndView model = new ModelAndView("error");
        model.addObject("exception", ex);
        return model;
    }
}
```

### ResponseStatusExceptionResolver

用于解析 当抛出自定义的异常（该异常用@ResponseStatus注解）时，对客户端响应。

```
//自定义异常类
//reason: 指定显示信息   
//value: http的状状态码
//不要标在方法上面，尽管可以，会造成应该正常的显示页面也产生错误页面上
@ResponseStatus(value=HttpStatus.FORBIDDEN,reason="用户名与密码不匹配")
public class NameNOTINPasswordException extends RuntimeException {
    private static final long serialVersionUID = 1L;
}
```


```
// 测试ResponseStatusExceptionResolver  注意自定义异常类NameNOTINPasswordException
@RequestMapping("TestResponseStatusExceptionResolver")
public String TestResponseStatusExceptionResolver(@RequestParam("i") int i){
    if(i==13 ){
        throw new NameNOTINPasswordException(); // 更改浏览器参数i为13
        //  throw new RuntimeErrorException(null);
    }
    System.out.println("TestResponseStatusExceptionResolver ...");
    return "success";
}
```
注意：ResponseStatusExceptionResolver只处理带有@ResponseStatus注解的异常并将其映射为状态码。

1. 第一次我们抛出RuntimeErrorException，由于没有注解@ResponseStatus，所以返回的为500 
1. 第二次我们修改我们抛出的异常为自定义的异常，并且利用注解@ResponseStatus，所以返回我们特定的状态码和信息。 

![image](http://img.blog.csdn.net/20170326184807718?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg5Nzg2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### DefaultHandlerExceptionResolver
DefaultHandlerExceptionResolver这个主要是处理Spring一些特定的异常并且把他们转化为状态码。在源码中我们可以看到。
```
/**
 * Default implementation of the {@link org.springframework.web.servlet.HandlerExceptionResolver
 * HandlerExceptionResolver} interface that resolves standard Spring exceptions and translates
 * them to corresponding HTTP status codes.
 *
 * <p>This exception resolver is enabled by default in the common Spring
 * {@link org.springframework.web.servlet.DispatcherServlet}.
 *
 * @author Arjen Poutsma
 * @author Rossen Stoyanchev
 * @author Juergen Hoeller
 * @since 3.0
 * @see org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler
 * @see #handleNoSuchRequestHandlingMethod
 * @see #handleHttpRequestMethodNotSupported //如：不支持我们提交的方法方式异常
 * @see #handleHttpMediaTypeNotSupported
 * @see #handleMissingServletRequestParameter
 * @see #handleServletRequestBindingException
 * @see #handleTypeMismatch
 * @see #handleHttpMessageNotReadable
 * @see #handleHttpMessageNotWritable
 * @see #handleMethodArgumentNotValidException
 * @see #handleMissingServletRequestParameter
 * @see #handleMissingServletRequestPartException
 * @see #handleBindException
 */
public class DefaultHandlerExceptionResolver extends AbstractHandlerExceptionResolver {
    //...
}
```
如：HttpRequestMethodNotSupported 是 不支持我们提交的方法方式异常

```
    //前台用get方式访问，会出现下列异常页面

    //后台处理  post方式
    @RequestMapping(value="/TestHttpRequestMethodNotSupportedException",method=RequestMethod.POST)
    public String TestHttpRequestMethodNotSupportedException(){
        System.out.println("TestHttpRequestMethodNotSupportedException...");
        return "success";

    } 
```

![image](http://img.blog.csdn.net/20170326190425385?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg5Nzg2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注意： 显然，从这里我们可以看出以前再出现错误页面时，都是经过框架给我们处理过的。

#### SimpleMappingExceptionResolver
主要用于当发生我们所指定的异常情况时，跳转到我们所指定的页面。现在版本不采用。

### Spring容器扫描配置
SpringMVC 层容器可以作为 Spring 容器的子容器：

即 Web 层容器可以引用业务层容器的 Bean，而业务层容器却访问不到 WEB 层容器 Bean。
```
<!-- SpringMVC Config -->
<context:component-scan base-package="com.fuyi.springmvc" use-default-filters="false">
    <!-- 只扫描 @Controller 和 @ControllerAdvice 注解-->
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    <context:include-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
</context:component-scan>

<!-- Spring Config -->
<context:component-scan base-package="com.fuyi.springmvc">
    <!-- 不扫描 @Controller 和 @ControllerAdvice 注解-->
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    <context:exclude-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
</context:component-scan>
```


参考：http://blog.csdn.net/sinat_28978689/article/details/66003845#t4