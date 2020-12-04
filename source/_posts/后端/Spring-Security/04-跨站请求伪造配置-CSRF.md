---
title: 04-跨站请求伪造配置-CSRF
toc: true
date: 2018-02-27 23:05:55
tags: Spring Security
---



## 什么是CSRF?

科普一下，什么是csrf,这是一个web应用安全的问题，CSRF（Cross-site request forgery跨站请求伪造，也被称为“One Click Attack” 或者Session Riding，攻击方通过伪造用户请求访问受信任站点。

我们知道，客户端与服务端在基于http协议在交互的数据的时候，由于http协议本身是无状态协议，后来引进了cookie的 方式进行记录服务端和客户端的之间交互的状态和标记。cookie里面一般会放置服务端生成的session id（会话ID）用来识别客户端访问服务端过 程中的客户端的身份标记。

在跨域 (科普一下：同一个ip、同一个网络协议、同一个端口，三者都满足就是同一个域，否则就有跨域问题) 的情况下， session id可能会被恶意第三方劫持，此时劫持这个session id的第三方会根据这个session id向服务器发起请求，此时服务器收到这个请求会 认为这是合法的请求，并返回根据请求完成相应的服务端更新。

如果这个http请求是get方式发起的请求，意味着它只是访问服务器 的资源，仅仅只是查询，没有更新服务器的资源，所以对于这类请求，spring security的防御策略是允许的，

如果这个请求是通过post请求发起的， 那么spring security是默认拦截这类请求的，因为这类请求是带有更新服务器资源的危险操作，如果恶意第三方可以通过劫持session id来更新 服务器资源，那会造成服务器数据被非法的篡改，所以这类请求是会被Spring security拦截的，在默认的情况下，spring security是启用csrf 拦截功能的，这会造成，在跨域的情况下，post方式提交的请求都会被拦截无法被处理（包括合理的post请求），前端发起的post请求后端无法正常 处理，虽然保证了跨域的安全性，但影响了正常的使用，如果关闭csrf防护功能，虽然可以正常处理post请求，但是无法防范通过劫持session id的非法的post请求，所以spring security为了正确的区别合法的post请求，采用了token的机制。

在跨域的场景下，客户端访问服务端会首先发起get请求，这个get请求在到达服务端的时候，服务端的Spring security会有一个过滤 器 CsrfFilter去检查这个请求，如果这个request请求的http header里面的X-CSRF-COOKIE的token值为空的时候，服务端就好自动生成一个 token值放进这个X-CSRF-COOKIE值里面，客户端在get请求的header里面获取到这个值，如果客户端有表单提交的post请求，则要求客户端要 携带这个token值给服务端，在post请求的header里面设置_csrf属性的token值，提交的方式可以是ajax也可以是放在form里面设置hidden 属性的标签里面提交给服务端，服务端就会根据post请求里面携带的token值进行校验，如果跟服务端发送给合法客户端的token值是一样的，那么 这个post请求就可以受理和处理，如果不一样或者为空，就会被拦截。由于恶意第三方可以劫持session id，而很难获取token值，所以起到了 安全的防护作用。



## Spring Security的CSRF支持

Spring Security 3默认关闭csrf，Spring Security 4默认启动了csrf。 

csrf开启时，请求会被CsrfFilter过滤器拦截，默认get请求不会被拦截，post都会被拦截，需采取token机制。

- 报错：HTTP Status 403 - Invalid CSRF Token 'null' was found on the request paramet

- 解决方案有以下两种：

1. 禁用csrf配置

```
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()               //禁用csrf保护
                .authorizeRequests()
                    .antMatchers("/", "/index").permitAll()
                    .anyRequest().authenticated()
                    .and()
                .formLogin()
                .loginPage("/login").permitAll()
                .and()
                .logout().permitAll();
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .inMemoryAuthentication()
                .withUser("user").password("password").roles("USER");
    }
}
```
2. 启用csrf时，需传递token

```
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
```
## 参考资料

> http://blog.csdn.net/jxchallenger/article/details/58640883
> http://blog.csdn.net/u012373815/article/details/55047285
> https://www.ibm.com/developerworks/cn/web/1102_niugang_csrf/
> https://www.jianshu.com/p/7f33f9c7997b
