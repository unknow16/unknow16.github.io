---
title: 跨站请求伪造配置-CSRF
date: 2018-02-27 23:05:55
tags: Spring Security
---

http://blog.csdn.net/u012373815/article/details/55047285
http://blog.csdn.net/jxchallenger/article/details/58640883

Spring Security 3默认关闭csrf，Spring Security 4默认启动了csrf。 

csrf开启时，请求会被CsrfFilter过滤器拦截，默认get请求不会被拦截，post都会被拦截，需采取token机制。

报错：HTTP Status 403 - Invalid CSRF Token 'null' was found on the request paramet


#### 解决：

* 禁用csrf配置
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
* 启用csrf时，需传递token

```
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
```
