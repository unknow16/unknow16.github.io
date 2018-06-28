---
title: Spring Boot-08-自动配置实现浅析
date: 2018-06-28 18:43:03
tags: Spring Boot
---

Spring Boot的自动配置实现由启动类上@SpringBootApplication注解中@EnableAutoConfiguration注解中的@Import(EnableAutoConfigurationImportSelector.class)注解导入的EnableAutoConfigurationImportSelector实现的。

1.5版本以前使用EnableAutoConfigurationImportSelector类，1.5以后这个类标识过时了，但在1.5之后代码中还是用该类，且该类中没有了核心实现，而在其父类AutoConfigurationImportSelector中。


```
// 我采用的版本：1.5.14.RELEASE

@SuppressWarnings("deprecation")
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	Class<?>[] exclude() default {};

	String[] excludeName() default {};
}

@Deprecated
public class EnableAutoConfigurationImportSelector
		extends AutoConfigurationImportSelector {

	@Override
	protected boolean isEnabled(AnnotationMetadata metadata) {
		if (getClass().equals(EnableAutoConfigurationImportSelector.class)) {
			return getEnvironment().getProperty(
					EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class,
					true);
		}
		return true;
	}

}
```
AutoConfigurationImportSelector会使用SpringFactoriesLoader.loadFactoryNames方法来扫描具有MEAT-INF/spring.factories文件的jar包。

```
// AutoConfigurationImportSelector类中的方法
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
		AnnotationAttributes attributes) {
	
	// 第一参数：EnableAutoConfiguration.class类字节码，如下
	// 第二参数：类加载器
	// 返回所有自动配置类全类名，具体实现看下文
	List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
			getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
	Assert.notEmpty(configurations,
			"No auto configuration classes found in META-INF/spring.factories. If you "
					+ "are using a custom packaging, make sure that file is correct.");
	return configurations;
}

protected Class<?> getSpringFactoriesLoaderFactoryClass() {
	return EnableAutoConfiguration.class;
}

```

SpringFactoriesLoader类的静态方法loadFactoryNames()实现

```
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";

public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
		
		// 返回org.springframework.boot.autoconfigure.EnableAutoConfiguration字符串
		String factoryClassName = factoryClass.getName();
		try {
		    // 通过ClassLoader获取META-INF/spring.factories资源文件
			Enumeration<URL> urls = (classLoader != null ? classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			List<String> result = new ArrayList<String>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				
				// 把URL转成Properties
				Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
				// 获取属性文件中key为EnableAutoConfiguration全类名的值，该值是逗号分隔的类全名字符串，它们是spring整合的自动配置类
				String factoryClassNames = properties.getProperty(factoryClassName);
				// 封装所有自动配置类全类名到List,并返回
				result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
			}
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() +
					"] factories from location [" + FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```

spring-boot-autoconfigure-1.5.14.RELEASE.jar中META-INF/spring.factories文件部分内容如下：
```
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnClassCondition

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
```
其实就是一个属性文件，左侧通常为一个接口或者是一个注解类，右侧为接口的实现，或者是和左值相关的注解。

Auto Configure注释下都是自动配置类，SpringBoot会根据这些自动配置类去自动配置环境，如对redis和mongo的自动配置类。

