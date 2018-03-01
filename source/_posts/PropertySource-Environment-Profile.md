---
title: PropertySource-Environment-Profile
date: 2018-03-01 09:26:32
tags: Spring
---

#### Spring3.1新属性管理API：PropertySource、Environment、Profile

- PropertySource: 属性源，key-value键值对抽象，比如用于配置数据
- PropertyResolver: 属性解析器，解析出key相应的value值
- Environment: 环境，本身是一个PropertyResolver, 但提供了Profile特性，即可以根据环境得到相应的数据，即激活不同的Profile，得到不同的属性配置，用于多环境配置（开发环境，测试环境，生产环境等的DataSource配置）
- Profile: 剖面，只有激活的剖面的组件或配置，才会注入到spring容器中，类似maven的profile


### PropertySource抽象基类中常用方法
```
1. public String getName()  //属性源的名字  
2. public T getSource()        //属性源（比如来自Map，那就是一个Map对象）  
3. public boolean containsProperty(String name)  //是否包含某个属性  
4. public abstract Object getProperty(String name)   //得到属性名对应的属性值 
```
#### 常用实现
- MapPropertySource: 属性来自map集合
- PropertiesPropertySource: 属性来自Property对象
- ResourcePropertySource: 属性来自properties文件，继承自上者
- ServletContextPropertySource: 属性来自servletContext上下文初始化参数
。

#### 测试
```
1. public void test() throws IOException {  
2.     Map<String, Object> map = new HashMap<>();  
3.     map.put("encoding", "gbk");  
4.     PropertySource propertySource1 = new MapPropertySource("map", map);  
5.     System.out.println(propertySource1.getProperty("encoding"));  
6.   
7.     ResourcePropertySource propertySource2 = new ResourcePropertySource("resource", "classpath:resources.properties"); //name, location  
8.     System.out.println(propertySource2.getProperty("encoding"));  
9. }  
```
- CompositePropertySource提供了组合PropertySource的功能，查找顺序就是注册顺序
```
1. @Test  
2. public void test2() throws IOException {  
3.     //省略propertySource1/propertySource2  
4.     CompositePropertySource compositePropertySource = new CompositePropertySource("composite");  
5.     compositePropertySource.addPropertySource(propertySource1);  
6.     compositePropertySource.addPropertySource(propertySource2);  
7.     System.out.println(compositePropertySource.getProperty("encoding"));  
8. } 
```
另外还有一个PropertySources，从名字可以看出其包含多个PropertySource：

```
1. public interface PropertySources extends Iterable<PropertySource<?>> {  
2.     boolean contains(String name); //是否包含某个name的PropertySource  
3.     PropertySource<?> get(String name); //根据name找到PropertySource  
4. }  
```

### PropertyResolver接口
```
1. public interface PropertyResolver {  
2.   
3.     //是否包含某个属性  
4.     boolean containsProperty(String key);  
5.    
6.         //获取属性值 如果找不到返回null   
7.     String getProperty(String key);  
8.        
9.         //获取属性值，如果找不到返回默认值        
10.     String getProperty(String key, String defaultValue);  
11.     
12.         //获取指定类型的属性值，找不到返回null  
13.     <T> T getProperty(String key, Class<T> targetType);  
14.   
15.         //获取指定类型的属性值，找不到返回默认值  
16.     <T> T getProperty(String key, Class<T> targetType, T defaultValue);  
17.   
18.          //获取属性值为某个Class类型，找不到返回null，如果类型不兼容将抛出ConversionException  
19.     <T> Class<T> getPropertyAsClass(String key, Class<T> targetType);  
20.   
21.         //获取属性值，找不到抛出异常IllegalStateException  
22.     String getRequiredProperty(String key) throws IllegalStateException;  
23.   
24.         //获取指定类型的属性值，找不到抛出异常IllegalStateException         
25.     <T> T getRequiredProperty(String key, Class<T> targetType) throws IllegalStateException;  
26.   
27.         //替换文本中的占位符（${key}）到属性值，找不到不解析  
28.     String resolvePlaceholders(String text);  
29.   
30.         //替换文本中的占位符（${key}）到属性值，找不到抛出异常IllegalArgumentException  
31.     String resolveRequiredPlaceholders(String text) throws IllegalArgumentException;  
32.   
33. } 
```

### Environment接口，继承自PropertyResolver
- JDK环境，servlet环境，spring环境，每个环境都有自己的配置数据
- 如 System.getProperties()、System.getenv()等可以拿到JDK环境数据；	
- ServletContext.getInitParameter()可以拿到Servlet环境配置数据等等；
- 也就是说Spring抽象了一个Environment来表示环境配置。
```
1. public interface Environment extends PropertyResolver {//继承PropertyResolver  
2.   
3.         //得到当前明确激活的剖面  
4.     String[] getActiveProfiles();  
5.   
6.         //得到默认激活的剖面，而不是明确设置激活的  
7.     String[] getDefaultProfiles();  
8.    
9.         //是否接受某些剖面  
10.     boolean acceptsProfiles(String... profiles);  
11.   
12. }  
```
从API上可以看出，除了可以解析相应的属性信息外，还提供了剖面相关的API，目的是： 可以根据剖面有选择的进行注册组件/配置。比如对于不同的环境注册不同的组件/配置（正式机、测试机、开发机等的数据源配置）。它的主要几个实现如下所示：
 
- MockEnvironment：模拟的环境，用于测试时使用；
- StandardEnvironment：标准环境，普通Java应用时使用，会自动注册System.getProperties() 和 System.getenv()到环境；
- StandardServletEnvironment：标准Servlet环境，其继承了StandardEnvironment，Web应用时使用，除了StandardEnvironment外，会自动注册ServletConfig（DispatcherServlet）、ServletContext及JNDI实例到环境；
- 除了这些，我们也可以根据需求定义自己的Environment。示例如下：
```
1. @Test  
2. public void test() {  
3.     //会自动注册 System.getProperties() 和 System.getenv()  
4.     Environment environment = new StandardEnvironment();  
5.     System.out.println(environment.getProperty("file.encoding"));  
6. }  
```
- 其默认有两个属性：systemProperties（System.getProperties()）和systemEnvironment（System.getenv()）。
- 使用StandardServletEnvironment加载时，默认除了StandardEnvironment的两个属性外，还有另外三个属性：servletContextInitParams（ServletContext）、servletConfigInitParams（ServletConfig）、jndiProperties（JNDI）。

### Profile
profile，剖面，大体意思是：我们程序可能从某几个剖面来执行应用，比如正式机环境、测试机环境、开发机环境等，每个剖面的配置可能不一样（比如开发机可能使用本地的数据库测试，正式机使用正式机的数据库测试）等；因此呢，就需要根据不同的环境选择不同的配置；如果用过maven，maven中就有profile的概念。

profile有两种：
- 默认的：通过“spring.profiles.default”属性获取，如果没有配置默认值是“default”
- 明确激活的：通过“spring.profiles.active”获取
- 查找顺序是：先进性明确激活的匹配，如果没有指定明确激活的（即集合为空）就找默认的；配置属性值从Environment读取。

### @PropertySource()
Spring 3.1提供的Java Config方式的注解，其属性会自动注册到相应的Environment；如：
```
1. @Configuration  
2. @PropertySource(value = "classpath:resources.properties", ignoreResourceNotFound = false)  
3. public class AppConfig {  
4. }  
```
张开涛老师：http://jinnianshilongnian.iteye.com/blog/2000183
