---
title: Spring MVC-06-控制器增强-@ControllerAdvice
date: 2018-02-28 17:29:54
tags: Spring MVC
---

@ControllerAdvice，是spring3.2提供的新注解，从名字上可以看出大体意思是控制器增强。

### @ControllerAdvice的实现：

```
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Component  
public @interface ControllerAdvice {  
  
}

package org.springframework.web.bind.annotation;

import java.lang.annotation.Annotation;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.core.annotation.AliasFor;
import org.springframework.stereotype.Component;

/**
 * Indicates the annotated class assists a "Controller".
 *      指定注解的类辅助一个控制器
 *
 * <p>Serves as a specialization of {@link Component @Component}, allowing for
 * implementation classes to be autodetected through classpath scanning.
 *      允许类路径下自动扫描注入
 *
 * <p>It is typically used to define {@link ExceptionHandler @ExceptionHandler},
 * {@link InitBinder @InitBinder}, and {@link ModelAttribute @ModelAttribute}
 * methods that apply to all {@link RequestMapping @RequestMapping} methods.
 * 
        把@ControllerAdvice注解内部使用@ExceptionHandler、@InitBinder、@ModelAttribute注解的方法
        应用到所有的 @RequestMapping注解的方法
 *
 * <p>One of {@link #annotations()}, {@link #basePackageClasses()},
 * {@link #basePackages()} or its alias {@link #value()}
 * may be specified to define specific subsets of Controllers
 * to assist. When multiple selectors are applied, OR logic is applied -
 * meaning selected Controllers should match at least one selector.
 *
 * <p>The default behavior (i.e. if used without any selector),
 * the {@code @ControllerAdvice} annotated class will
 * assist all known Controllers.
 *
 * <p>Note that those checks are done at runtime, so adding many attributes and using
 * multiple strategies may have negative impacts (complexity, performance).
 *
 * @author Rossen Stoyanchev
 * @author Brian Clozel
 * @author Sam Brannen
 * @since 3.2
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component //该注解说明使用<context:component-scan>扫描时也能扫描到
public @interface ControllerAdvice {...}
```

### 自定义使用

```
@ControllerAdvice  
public class ControllerAdviceTest {  
  
    @ModelAttribute  
    public User newUser() {  
        System.out.println("============应用到所有@RequestMapping注解方法，在其执行之前把返回值放入Model");  
        return new User();  
    }  
  
    @InitBinder  
    public void initBinder(WebDataBinder binder) {  
        System.out.println("============应用到所有@RequestMapping注解方法，在其执行之前初始化数据绑定器");  
    }  
  
    @ExceptionHandler(UnauthenticatedException.class)  
    @ResponseStatus(HttpStatus.UNAUTHORIZED)  
    public String processUnauthenticatedException(NativeWebRequest request, UnauthenticatedException e) {  
        System.out.println("===========应用到所有@RequestMapping注解的方法，在其抛出UnauthenticatedException异常时执行");  
        return "viewName"; //返回一个逻辑视图名  
    }  
}  
```

### 配置
* spring-mvc.xml中配置，包含扫描@ControllerAdvice注解类
```
<context:component-scan base-package="com.fuyi.mvc" use-default-filters="false">  
       <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>  
       <context:include-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice"/>  
</context:component-scan>  
```

