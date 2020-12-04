---
title: Spring Boot-06-单元测试
date: 2018-06-28 18:42:16
tags: Spring Boot
---

## spring-test
使用spring的测试框架需要加入以下依赖包：
- JUnit 4 （官方下载：http://www.junit.org/）
- Spring Test （Spring框架中的test包）
- Spring 相关其他依赖包（不再赘述了，就是context等包）

如果使用maven，在基于spring的项目中添加如下依赖：
```
<dependency>  
            <groupId>junit</groupId>  
            <artifactId>junit</artifactId>  
            <version>4.9</version>  
            <scope>test</scope>  
</dependency>   
<dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-test</artifactId>  
            <version>4.3.18.RELEASE</version>  
            <scope>provided</scope>  
</dependency> 
```
#### 创建测试类
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:META-INF/spring/*.xml"})
@Transactional
@TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = true)
public class TestApp {

	@Autowired
	IUserInfoService userInfoService;
	
	@Test
	public void test() {
		System.out.println("ppppppppppppppppp" + userInfoService);
	}
}
```
- @RunWith：用于指定junit运行环境，是junit提供给其他框架测试环境接口扩展，为了便于使用spring的依赖注入，spring提供了org.springframework.test.context.junit4.SpringJUnit4ClassRunner作为Junit测试环境
- @ContextConfiguration：用来指定要加载的spring的配置文件，其中locations和value属性是等同的，指定配置文件位置
- @TransactionConfiguration：这里的事务关联到配置文件中的事务控制器，同时指定自动回滚。这样做操作的数据才不会污染数据库！
- @Transactional:这个非常关键，如果不加入这个注解配置，事务控制就会完全失效！ 
- 事务相关注解不是必须的，可在方法上加注解控制

## spring-boot-test
spring-boot-test通过以下两个模块支持：
- spring-boot-test ： 包含测试核心，其中包含spring-test,它是spring框架专用的测试框架
- spring-boot-test-autoconfigure ：支持自动配置测试环境

通过spring-boot-starter-test能同时引入上面两个模块，这两个模块引用了下列库：
- JUnit：The de-facto standard for unit testing Java applications.
- Spring Test & Spring Boot Test：Utilities and integration test support for Spring Boot applications.
- AssertJ：A fluent assertion library.
- Hamcrest：A library of matcher objects (also known as constraints or predicates).
- Mockito：A Java mocking framework.
- JSONassert：An assertion library for JSON.
- JsonPath：XPath for JSON.

Spring Boot 从1.4之后 提供了@SpringBootTest代替了spring-test中的@ContextConfiguration。Spring Boot 1.4之前使用是的SpringApplicationConfiguration（已过时）。

如果你只想测试SpringMVC层而不想测试service，或者只想测service层而不想测dao和controller层等等，可以采用Mock，且spring-boot-test-autoconfigure模块包含了很多注解，像切片一样为你自动配置

大多像@...Test的注解，如@WebMvcTest、@DataJpaTest、@JdbcTest、@DataMongoTest、@RestClientTest等等，它们会加载一个相应的ApplicationContext

或者可以使用@AutoConfigure…之类注解自定义需要的配置，如@AutoConfigureMockMvc、@AutoConfigureDataJpa，上面的@...Test之类注解根据相应环境对这些@AutoConfigure…之类注解进行了组合，如@WebMvcTest就包含了@AutoConfigureMockMvc和其他一些web层要用到的对象。

#### 非controller测试类
```
// Note: given方法所在类，来自AssertJ库
import static org.assertj.core.api.Assertions.*;

// Note: assertThat方法所在类，来自Mockito库
import static org.mockito.BDDMockito.*;

// @SpringBootTest类的classes属性一般可不配置，会自动寻找
// 但如果不配置报错时，配置启动类即可，推荐配置
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {Application.class})
public class TestApp {
	// 采用dubbo时，即该接口实现在另外dubbo项目时
	// 要先启动服务提供者dubbo,注册到zk,才能注入实现
	@Autowired
	IUserService userService; 
		
    // 如果配置mock对象，则会覆盖该接口的真实实现
    // 即上面配置的实现将不会生效
	@MockBean // Mock对象配置1
	IUserService userServiceMock; 

	@Test
	public void test1() {
	    // Mock对象配置2
	    // 此处可换为调用Dao层的模拟调用返回
		given(this.userServiceMock.checkUserLoanStatus("aaa")).willReturn("Hello");
		
		assertThat(userService.checkUserLoanStatus("aaa")).isEqualTo("Hello");
	}
	
}
```

#### controller测试

```
@RestController
public class TestController {
	
	@Autowired
	IUserService userService;

	@RequestMapping("/test")
	public String test() {
		return userService.checkUsr("aaa").toString();
	}
}


@RunWith(SpringRunner.class)
@SpringBootTest(classes = {Application.class},webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
//@WebMvcTest(TestController.class) //不建议用
public class TestAppMVC {

    // 通过@AutoConfigureMockMvc加载的
	@Autowired
	MockMvc  mvc; 
	
	@MockBean
	IUserService userService;
	
	// 要配置@SpringBootTest的webEnvironment属性
	// 用来执行http调用，用法和RestTemplate类似
	@Autowired
	TestRestTemplate testRestTemplate;
	
	
	@Test
	public void test1() throws Exception {
	    // 此处入参要和Controller中一样
		given(this.userService.checkUsr("aaa")).willReturn("fuyi");
		
		// 执行的请求url
		this.mvc.perform(MockMvcRequestBuilders.get("/test"))
        // 状态码200
        .andExpect(MockMvcResultMatchers.status().isOk())
        // 期待的响应值
        .andExpect(content().string("fuyi"));
	}
	
	
	@Test
	public void test2() {
		System.out.println("oooooooooooooooo " + testRestTemplate.getForObject("https://www.baidu.com", String.class));
	}
}
```
下面简单说下@WebMvcTest在源代码中的解释：
1. 结合@RunWith(SpringRunner.class)对SpringMVC进行测试，仅测试SpringMVC层。
2. 使用此注释将禁用全部的自动配置，而仅应用与MVC测试相关的配置 ，如只会应用 @Controller, @ControllerAdvice, @JsonComponent Filter注解和WebMvcConfigurer and HandlerMethodArgumentResolver 这些beans 而不应用 @Component, @Service or @Repository等。
3. 如果你还想加载所有配置，且还想使用MockMVC，则可使用@SpringBootTest和 @AutoConfigureMockMvc，而不是用@WebMvcTest注解
