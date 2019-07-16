---
title: Spring Boot-15-注入自定义Servlet等组件的二种方式
date: 2018-07-24 17:13:16
tags: Spring Boot
---

> 以下两种方式都是Servlet 3.0规范中新增的方式。

## 方式一：注解配置

#### 1. 用注解声明组件

* Servlet
```
@WebServlet(urlPatterns = "/druid/*",
initParams = {
		// IP白名单(没有配置或者为空，则允许所有访问)
		@WebInitParam(name="allow",value="192.168.1.72,127.0.0.1"),
		// IP黑名单 (存在共同时，deny优先于allow)
		@WebInitParam(name="deny",value="192.168.1.73"),
		// 用户名
		@WebInitParam(name="loginUsername",value="admin"),
		// 密码
        @WebInitParam(name="loginPassword",value="123456"),
        // 禁用HTML页面上的“Reset All”功能
        @WebInitParam(name="resetEnable",value="false")
})
public class DruidStatViewServlet extends StatViewServlet{
	private static final long serialVersionUID = 1L;
}

```

* Filter
```
@WebFilter(	urlPatterns = "/*", 
			filterName = "stat",
			initParams = {
					// 忽略资源
					@WebInitParam(name="exclusions",value="*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*")
			})
public class DruidStatFilter extends WebStatFilter{
}
```
* Listener

```
@WebListener("This is only a demo listener") 
public class SimpleListener implements ServletContextListener{...}

```

#### 2. Spring Boot启动类上添加@ServletComponentScan注解
@ServletComponentScan是Spring的注解。

Spring通过该注解能够扫描到我们自己编写的Servlet、Filter、Listner。

## 方式二：代码注册
通过Spring提供的如下bean注册，其内部使用了Servlet 3.0的javax.servlet.ServletContext新增的addServlet()、addFilter()、addListener()等方法。
- ServletRegistrationBean
- FilterRegistrationBean
- ServletListenerRegistrationBean
```
@Configuration
public class DruidMonitorConfig {

	@Bean
	public ServletRegistrationBean servletRegistrationBean() {
		ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean();

		// 其内部使用了Servlet 3.0的javax.servlet.ServletContext新增的addServlet()方法
		servletRegistrationBean.setServlet(new StatViewServlet());
		servletRegistrationBean.addUrlMappings("/druid/*");

		// 添加初始化参数：initParams

		// 白名单：
		servletRegistrationBean.addInitParameter("allow", "127.0.0.1");
		// IP黑名单 (存在共同时，deny优先于allow) :
		// 如果满足deny的话提示:Sorry, you are not permitted to view this page.
		servletRegistrationBean.addInitParameter("deny", "192.168.1.73");
		// 登录查看信息的账号密码.
		servletRegistrationBean.addInitParameter("loginUsername", "admin");
		servletRegistrationBean.addInitParameter("loginPassword", "123456");
		// 是否能够重置数据.
		servletRegistrationBean.addInitParameter("resetEnable", "false");

		return servletRegistrationBean;
	}

	@Bean
	public FilterRegistrationBean filterRegistrationBean() {
		FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();

		// 其内部使用了Servlet 3.0的javax.servlet.ServletContext新增的addFilter()方法
		filterRegistrationBean.setFilter(new WebStatFilter());
		// 添加过滤规则.
		filterRegistrationBean.addUrlPatterns("/*");
		
		// 添加不需要忽略的格式信息.
		filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid2/*");
		
		return filterRegistrationBean;
	}
}
```
