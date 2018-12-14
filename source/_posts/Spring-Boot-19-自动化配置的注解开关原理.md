---
title: Spring Boot-19-自动化配置的注解开关原理
date: 2018-12-14 10:55:47
tags: Spring Boot
---


有些自动化配置类，需要加上Enable*之类的注解开关的情况下才会生效，如@EnableCaching等，其实该类注解中都是通过@Import注解来导入相应的标注了@Configuration注解的配置类，但spring boot中的实现更灵活，通常用@Import导入的是ImportSelector的实现类，它的selectImports方法可以根据开关类注解中的属性，灵活的选择导入不同的配置类。



## 简单示例
需求：定义一个Annotation，让使用了这个Annotaion的应用程序自动化地注入一些类或者做一些底层的事情。

如下定义一个注解EnableContentService，使用了这个注解的程序会自动注入ContentService这个bean。

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(ContentConfiguration.class)
public @interface EnableContentService {}

public interface ContentService {
    void doSomething();
}

public class SimpleContentService implements ContentService {
    @Override
    public void doSomething() {
        System.out.println("do some simple things");
    }
}

@Configuration
public class ContentConfiguration {
    @Bean
    public ContentService contentService() {
        return new SimpleContentService();
    }
}
```
然后在应用程序的入口加上@EnableContentService注解。

这样的话，ContentService就被注入进来了。 SpringBoot也就是用这个完成的。只不过它用了更加高级点的ImportSelector。

## 应用ImportSelector
用了ImportSelector之后，我们可以在Annotation上添加一些属性，然后根据属性的不同加载不同的bean。

我们在@EnableContentService注解添加属性policy，同时Import一个Selector。


```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(ContentImportSelector.class)
public @interface EnableContentService {
    String policy() default "simple";
}
```

这个ContentImportSelector根据EnableContentService注解里的policy加载不同的bean。


```
public class ContentImportSelector implements ImportSelector {

    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        Class<?> annotationType = EnableContentService.class;
        AnnotationAttributes attributes = AnnotationAttributes.fromMap(importingClassMetadata.getAnnotationAttributes(annotationType.getName(), false));
        String policy = attributes.getString("policy");
        if ("core".equals(policy)) {
            return new String[] { CoreContentConfiguration.class.getName() };
        } else {
            return new String[] { SimpleContentConfiguration.class.getName() };
        }
    }
}
```
CoreContentService和CoreContentConfiguration如下：

```
public class CoreContentService implements ContentService {
    @Override
    public void doSomething() {
        System.out.println("do some import things");
    }
}

public class CoreContentConfiguration {
    @Bean
    public ContentService contentService() {
        return new CoreContentService();
    }
}
```
这样的话，如果在@EnableContentService注解的policy中使用core的话，应用程序会自动加载CoreContentService，否则会加载SimpleContentService。

## ImportSelector在SpringBoot中的使用
SpringBoot里的ImportSelector是通过SpringBoot提供的@EnableAutoConfiguration这个注解里完成的。

这个@EnableAutoConfiguration注解可以显式地调用，否则它会在@SpringBootApplication注解中隐式地被调用。

@EnableAutoConfiguration注解中使用了EnableAutoConfigurationImportSelector作为ImportSelector。下面这段代码就是EnableAutoConfigurationImportSelector中进行选择的具体代码：

```
@Override
public String[] selectImports(AnnotationMetadata metadata) {
    try {
        AnnotationAttributes attributes = getAttributes(metadata);
        // 从所有的jar包中读取META-INF/spring.factories中
        // 以EnableAutoConfiguration全类名为key的默认支持自动配置的配置类
        List<String> configurations = getCandidateConfigurations(metadata, attributes);
        configurations = removeDuplicates(configurations); // 删除重复的配置
        Set<String> exclusions = getExclusions(metadata, attributes); // 去掉需要exclude的配置
        configurations.removeAll(exclusions);
        configurations = sort(configurations); // 排序
        recordWithConditionEvaluationReport(configurations, exclusions);
        return configurations.toArray(new String[configurations.size()]);
    }
    catch (IOException ex) {
        throw new IllegalStateException(ex);
    }
}
```
上面的getCandidateConfigurations()方法会从所有的jar包中读取META-INF/spring.factories中以EnableAutoConfiguration全类名为key的默认支持自动配置的配置类，spring.factories部分内容如下：
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