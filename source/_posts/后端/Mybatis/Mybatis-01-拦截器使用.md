---
title: Mybatis拦截器使用
date: 2018-02-26 23:41:06
tags: Mybatis
---

http://blog.csdn.net/moshenglv/article/details/52699976

### Interceptor接口

```
package org.apache.ibatis.plugin;
 
import java.util.Properties;
 
public interface Interceptor {
 
 //拦截时，执行的方法
  Object intercept(Invocation invocation) throws Throwable;
 
 //用于封装目标对象
 //1. 返回目标对象本身，不会调用intercept()方法
 //2. 返回目标对象的代理对象，此时会调用该拦截器的intercept()方法
  Object plugin(Object target);
 
 //获取配置配置拦截器时，传入的参数
  void setProperties(Properties properties);
}
```

### 编写拦截器
1. 实现接口

```
/**
@Intercepts表明当前类是一个拦截器
@Signature执行了要拦截的类+方法+参数
    type: 取值只能是Executor、StatementHandler、ParameterHandler、ResultSetHandler其中之一。


*/
@Intercepts({ @Signature(method = "query", type = Executor.class, args = { MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class }),
				@Signature(method="prepare", type=StatementHandler.class, args={Connection.class, Integer.class})
			})
public class MyInterceptor implements Interceptor {

	public Object intercept(Invocation invocation) throws Throwable {
		Object result = invocation.proceed();
		System.out.println("intercept = " + result);
		return result;
	}

    /**
    1. Plugin中根据传入的this即当前拦截器，获取类注解中指定的拦截的类+方法+参数。
    2. 当拦截器链执行到需拦截的方法时，会返回一个JDK代理对象，否则返回想要拦截的对象本身，即目标对象target
    3. 返回目标对象时，不会执行intercept()方法，会继续原来逻辑
    4. 返回目标对象的代理对象时，当目标对象执行注解中配置的需拦截的方法时，
        会执行创建代理对象时传入的实现了InvocationHandler接口的实现类的invoke()方法，
        而该Plugin已经实现了InvocationHandler接口，即执行Plugin中的invoke()方法。
        而该invoke()方法中又调用了1步中传入的this拦截器中的intercept()方法，同时传入封装了目标对象，目标方法，方法参数的Invocation对象。
        此时我们可以在拦截器的intercept中编写拦截逻辑。
    5. 需放行的话，可调用Invocation的procced(),其中是让method执行，即method.invoke(target, args);
    */
	public Object plugin(Object target) {
		return Plugin.wrap(target, this);
	}

    //接收sqlMapConfig中配置的属性
	public void setProperties(Properties properties) {
		String v1 = properties.getProperty("key1");
		String v2 = properties.getProperty("key2");
		System.out.println("v1 = " + v1 + ", v2 = " + v2);
	}

}
```

2. sqlMapConfig中配置

```
	<plugins>
		<plugin interceptor="com.fuyi.mybatis_demo.interceptor.MyInterceptor">
			<property name="key1" value="vvvvv1"/>
			<property name="key2" value="vvvvv2"/>
		</plugin>
	</plugins>
```

