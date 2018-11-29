---
title: Spring Boot-04-启动过程解析
date: 2018-05-07 20:45:10
tags: Spring Boot
---


### Spring Boot应用程序一般启动代码如下：
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```
## @SpringBootApplication
进入该注解，其中标注的三个注解正能解决我们自动配置、组件扫描功能，它们是：

```
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```
该注解除了上面三个注解外，还有四个方法如下：
- Class<?>[] exclude() default {}:
根据 class 来排除，排除特定的类加入 spring 容器，传入参数 value 类型是 class 类型。
- String[] excludeName() default {}:
根据 class name 来排除，排除特定的类加入 spring 容器，传入参数 value 类型是 class 的全类名字符串数组。
- String[] scanBasePackages() default {}:
指定扫描包，参数是包名的字符串数组。
- Class<?>[] scanBasePackageClasses() default {}:
扫描特定的包，参数类似是 Class 类型数组。

#### 1. @SpringBootConfiguration

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {

}
```
@SpringBootConfiguration继承自@Configuration，二者功能也一致，标注当前类是配置类，并会将当前类内声明的一个或多个以@Bean注解标记的方法的实例纳入到srping容器中，并且实例名就是方法名。

#### 2. @ComponentScan
可以通过该注解指定扫描某些包下包含如下注解的均自动注册为 spring beans：

@Component、@Service、 @Repository、 @Controller、@Entity 等等

#### 3. @EnableAutoConfiguration
@EnableAutoConfiguration的作用启动自动的配置，意思就是Springboot根据你添加的 jar 包来配置你项目的默认配置，比如根据spring-boot-starter-web ，来判断你的项目是否需要添加了webmvc和tomcat，就会自动的帮你配置 web 项目中所需要的默认配置。简单点说就是它会根据定义在 classpath 下的类，自动的给你生成一些 Bean，并加载到 Spring 的 Context 中。

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	Class<?>[] exclude() default {};

	String[] excludeName() default {};
}
```
可以看到 import 引入了 AutoConfigurationImportSelector 类。该类的selectImports方法调用SpringFactoriesLoader 类的 loadFactoryNamesof()方法，会从所有的jar包中读取META-INF/spring.factories中以EnableAutoConfiguration全类名为key的默认支持自动配置的配置类，spring.factories部分内容如下：


```
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration,\
```

当然了，这些AutoConfiguration类中的@Bean标注需实例化的类，不是所有都会加载的，会根据AutoConfiguration上的@ConditionalOnClass等条件判断是否加载。





## SpringApplication.run(Application.class, args);
SpringApplication的静态run方法中会先new一个SpringApplication实例，然后再调用其run(String... args)方法。

#### 1. new SpringApplication

```
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
	this.resourceLoader = resourceLoader;
	Assert.notNull(primarySources, "PrimarySources must not be null");
	this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
	
	// 判断是否是web程序
	this.webApplicationType = WebApplicationType.deduceFromClasspath();
	
	// 从spring.factories文件中找出key为ApplicationContextInitializer的类
	// 并实例化后设置到SpringApplication的initializers属性中。
	// 这个过程也就是找出所有的应用程序初始化器
	setInitializers((Collection) getSpringFactoriesInstances(
			ApplicationContextInitializer.class));
			
    // 从spring.factories文件中找出key为ApplicationListener的类
    // 并实例化后设置到SpringApplication的listeners属性中。
    // 这个过程就是找出所有的应用程序事件监听器
	setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
	this.mainApplicationClass = deduceMainApplicationClass();
}
```
该阶段主要是设置了SpringApplication的webApplicationType、initializers、listeners属性。

ApplicationContextInitializer，应用程序初始化器，做一些初始化的工作：


```
public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {
	void initialize(C applicationContext);
}
```
默认情况下，从spring.factories文件中找出的key为ApplicationContextInitializer的类有：

```
org.springframework.boot.context.config.DelegatingApplicationContextInitializer
org.springframework.boot.context.ContextIdApplicationContextInitializer
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer
org.springframework.boot.context.web.ServerPortInfoApplicationContextInitializer
org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer
```

ApplicationListener，应用程序事件(ApplicationEvent)监听器，继承了java.util.EventListener接口，注意区别后面提到的SpringApplicationRunListener：


```
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
	void onApplicationEvent(E event);
}
```

这里的应用程序事件(ApplicationEvent)继承了java.util.EventObject类，有应用程序启动事件(ApplicationStartingEvent，ApplicationStartedEvent)，失败事件(ApplicationFailedEvent)，准备事件(ApplicationPreparedEvent)等。


key为ApplicationListener的有：


```
org.springframework.boot.context.config.ConfigFileApplicationListener
org.springframework.boot.context.config.AnsiOutputApplicationListener
org.springframework.boot.logging.LoggingApplicationListener
org.springframework.boot.logging.ClasspathLoggingApplicationListener
org.springframework.boot.autoconfigure.BackgroundPreinitializer
org.springframework.boot.context.config.DelegatingApplicationListener
org.springframework.boot.builder.ParentContextCloserApplicationListener
org.springframework.boot.context.FileEncodingApplicationListener
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener
```

#### 2. getRunListeners(args)
该方法为run(String... args)中的一个方法
```
private SpringApplicationRunListeners getRunListeners(String[] args) {
	Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
	
	// 1. getSpringFactoriesInstance获取SpringApplicationRunListener实现类
	// 2. new 一个SpringApplicationRunListeners工具类
	return new SpringApplicationRunListeners(logger, 
	    getSpringFactoriesInstances(
	        SpringApplicationRunListener.class, 
	        types, this, args));
}
```
1. 先通过getSpringFactoriesInstance从spring.factories文件中找出的key为SpringApplicationRunListener接口的实现类，目前                    其实现类只有一个EventPublishingRunListener。
2. 构造一个SpringApplicationRunListeners工具类，SpringApplicationRunListeners内部持有SpringApplicationRunListener集合和1个Log日志类。用于SpringApplicationRunListener监听器的批量执行。

SpringApplicationRunListener看名字也知道用于监听SpringApplication的run方法的执行，即run方法执行过程中，会回调其相应方法，在其相应方法中会做什么呢？

SpringApplicationRunListener接口定义如下：
```
public interface SpringApplicationRunListener {

	void starting();

	void environmentPrepared(ConfigurableEnvironment environment);

	void contextPrepared(ConfigurableApplicationContext context);

	void contextLoaded(ConfigurableApplicationContext context);

	void started(ConfigurableApplicationContext context);

	void running(ConfigurableApplicationContext context);

	void failed(ConfigurableApplicationContext context, Throwable exception);
}
```

其唯一实现类EventPublishingRunListener包含一个SimpleApplicationEventMulticaster，它是ApplicationEventMulticaster接口的实现类，每次回调EventPublishingRunListener的各个方法时，EventPublishingRunListener先构造一个相应的ApplicationEvent的子类，如在回调starting()方法时，会构造ApplicationStartingEvent，然后会通过SimpleApplicationEventMulticaster的multicastEvent()，参数为ApplicationStartingEvent，将该事件广播出去，代码层面即是调用所有ApplicationListener实现类的onApplicationEvent(E event)方法，这些ApplicationListener是在new SpringApplication()时加载实例化的。

从SpringApplicationRunListeners到ApplicationListener的事件传递图如下：

![image](https://note.youdao.com/yws/api/personal/file/ECF865CC9B614568BC02DF7A2E4E23B1?method=download&shareKey=568e4aa117548ee73bc0ae0d790e9bf9)

#### 3. run(String... args)
如下注释1处: 就是上面提到的getRunListeners(args)方法

如下注释2处: 遍历调用SpringApplicationRunListener的starting()，其实也就是调用EventPublishingRunListener的starting()方法
```
public ConfigurableApplicationContext run(String... args) {
	StopWatch stopWatch = new StopWatch();
	stopWatch.start();
	ConfigurableApplicationContext context = null;
	Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
	configureHeadlessProperty();
	
	// 1. 获取SpringApplicationRunListeners，内部只有一个EventPublishingRunListener
	SpringApplicationRunListeners listeners = getRunListeners(args);
	
	// 2. 遍历调用SpringApplicationRunListener的starting，其实也就是调用EventPublishingRunListener的starting()方法
	listeners.starting();
	try {
		ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
		
		// 3. 准备环境，其中会调用SpringApplicationRunListeners的environmentPrepared方法
		ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
		configureIgnoreBeanInfo(environment);
		Banner printedBanner = printBanner(environment);
		
		// 4. 创建ApplicationContext
		context = createApplicationContext();
		exceptionReporters = getSpringFactoriesInstances(
				SpringBootExceptionReporter.class,
				new Class[] { ConfigurableApplicationContext.class }, context);
		
		// 5. 准备上下文，其中调用SpringApplicationRunListeners的contextPrepared，contextLoaded方法
		prepareContext(context, environment, listeners, applicationArguments, printedBanner);
		
		// 6. 刷新上下文
		refreshContext(context);
		
		// 7. 刷新后执行
		afterRefresh(context, applicationArguments);
		stopWatch.stop();
		if (this.logStartupInfo) {
			new StartupInfoLogger(this.mainApplicationClass)
					.logStarted(getApplicationLog(), stopWatch);
		}
		
		// 8. 调用SpringApplicationRunListeners的started方法
		listeners.started(context);
		
		// 9. 其中会遍历调用spring容器中所有ApplicationRunner、CommandLineRunner实例的run方法
		callRunners(context, applicationArguments);
	}
	catch (Throwable ex) {
	
	    // 11. 调用调用SpringApplicationRunListeners的failed方法
		handleRunFailure(context, ex, exceptionReporters, listeners);
		throw new IllegalStateException(ex);
	}

	try {
	    // 10. 调用SpringApplicationRunListeners的running方法
		listeners.running(context);
	}
	catch (Throwable ex) {
	
	    // 11. 调用调用SpringApplicationRunListeners的failed方法
		handleRunFailure(context, ex, exceptionReporters, null);
		throw new IllegalStateException(ex);
	}
	return context;
}
```

[Format-SpringBoot源码分析之SpringBoot的启动过程](http://fangjian0423.github.io/2017/04/30/springboot-startup-analysis/)