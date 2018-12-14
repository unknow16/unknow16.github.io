---
title: Spring Boot-21-条件注解一览
date: 2018-12-14 10:59:54
tags: Spring Boot
---

## 条件注解总结
条件注解|	对应的Condition处理类|	处理逻辑
--|--|--
@ConditionalOnBean|	OnBeanCondition|	Spring容器中是否存在对应的实例。可以通过实例的类型、类名、注解、昵称去容器中查找(可以配置从当前容器中查找或者父容器中查找或者两者一起查找)这些属性都是数组，通过”与”的关系进行查找
@ConditionalOnClass|	OnClassCondition|	类加载器中是否存在对应的类。可以通过Class指定(value属性)或者Class的全名指定(name属性)。如果是多个类或者多个类名的话，关系是”与”关系，也就是说这些类或者类名都必须同时在类加载器中存在
@ConditionalOnExpression|	OnExpressionCondition|	判断SpEL 表达式是否成立
@ConditionalOnJava|	OnJavaCondition|	指定Java版本是否符合要求。内部有2个属性value和range。value表示一个枚举的Java版本，range表示比这个老或者新于等于指定的Java版本(默认是新于等于)。内部会基于某些jdk版本特有的类去类加载器中查询，比如如果是jdk9，类加载器中需要存在java.security.cert.URICertStoreParameters；如果是jdk8，类加载器中需要存在java.util.function.Function；如果是jdk7，类加载器中需要存在java.nio.file.Files；如果是jdk6，类加载器中需要存在java.util.ServiceLoader
@ConditionalOnMissingBean|	OnBeanCondition|	Spring容器中是否缺少对应的实例。可以通过实例的类型、类名、注解、昵称去容器中查找(可以配置从当前容器中查找或者父容器中查找或者两者一起查找)这些属性都是数组，通过”与”的关系进行查找。还多了2个属性ignored(类名)和ignoredType(类名)，匹配的过程中会忽略这些bean
@ConditionalOnMissingClass|	OnClassCondition|	跟ConditionalOnClass的处理逻辑一样，只是条件相反，在类加载器中不存在对应的类
@ConditionalOnNotWebApplication|	OnWebApplicationCondition|	应用程序是否是非Web程序，没有提供属性，只是一个标识。会从判断Web程序特有的类是否存在，环境是否是Servlet环境，容器是否是Web容器等
@ConditionalOnProperty|	OnPropertyCondition|	应用环境中的屬性是否存在。提供prefix、name、havingValue以及matchIfMissing属性。prefix表示属性名的前缀，name是属性名，havingValue是具体的属性值，matchIfMissing是个boolean值，如果属性不存在，这个matchIfMissing为true的话，会继续验证下去，否则属性不存在的话直接就相当于匹配不成功
@ConditionalOnResource|	OnResourceCondition|	是否存在指定的资源文件。只有一个属性resources，是个String数组。会从类加载器中去查询对应的资源文件是否存在
@ConditionalOnSingleCandidate|	OnBeanCondition|	Spring容器中是否存在且只存在一个对应的实例。只有3个属性value、type、search。跟ConditionalOnBean中的这3种属性值意义一样
@ConditionalOnWebApplication|	OnWebApplicationCondition|	应用程序是否是Web程序，没有提供属性，只是一个标识。会从判断Web程序特有的类是否存在，环境是否是Servlet环境，容器是否是Web容器等

## 一些例子
例子|	例子意义
--|--
@ConditionalOnBean(javax.sql.DataSource.class)|	Spring容器或者所有父容器中需要存在至少一个javax.sql.DataSource类的实例
@ConditionalOnClass({ Configuration.class,FreeMarkerConfigurationFactory.class })|	类加载器中必须存在Configuration和FreeMarkerConfigurationFactory这两个类
@ConditionalOnExpression(“‘${server.host}’==’localhost’”)|	server.host配置项的值需要是localhost
ConditionalOnJava(JavaVersion.EIGHT)|	Java版本至少是8
@ConditionalOnMissingBean(value = ErrorController.class, search = SearchStrategy.CURRENT)|	Spring当前容器中不存在ErrorController类型的bean
@ConditionalOnMissingClass(“GenericObjectPool”)|	类加载器中不能存在GenericObjectPool这个类
@ConditionalOnNotWebApplication|	必须在非Web应用下才会生效
@ConditionalOnProperty(prefix = “spring.aop”, name = “auto”, havingValue = “true”, matchIfMissing = true)|	应用程序的环境中必须有spring.aop.auto这项配置，且它的值是true或者环境中不存在spring.aop.auto配置(matchIfMissing为true)
@ConditionalOnResource(resources=”mybatis.xml”)|	类加载路径中必须存在mybatis.xml文件
@ConditionalOnSingleCandidate(PlatformTransactionManager.class)|	Spring当前或父容器中必须存在PlatformTransactionManager这个类型的实例，且只有一个实例
@ConditionalOnWebApplication|	必须在Web应用下才会生效
