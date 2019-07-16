---
title: Spring Boot-20-条件注解-源码实现
date: 2018-12-14 10:58:31
tags: Spring Boot
---



前文多次提到自动配置原理为判断classpath下是否存在相应框架的标志性的Class，如果存在就向Spring IOC容器中注入整合的配置类，否则就skip，其实判断相应Class是否存在classpath的功能是由一系列的注解并配合类实现的，本文就这些注解实现解析。

这些注解被称为条件注解(Conditional Annotation)。比如@ConditionalOnBean、@ConditionalOnClass、@ConditionalOnExpression、@ConditionalOnMissingBean等。条件注解存在的意义在于动态识别(也可以说是代码自动化执行)。比如@ConditionalOnClass会检查类加载器中是否存在对应的类，如果有的话被注解修饰的类就有资格被Spring容器所注册，否则会被skip。

比如FreemarkerAutoConfiguration这个自动化配置类的定义如下：


```
@Configuration
@ConditionalOnClass({ freemarker.template.Configuration.class,
		FreeMarkerConfigurationFactory.class })
@EnableConfigurationProperties(FreeMarkerProperties.class)
@Import({ FreeMarkerServletWebConfiguration.class,
		FreeMarkerReactiveWebConfiguration.class, FreeMarkerNonWebConfiguration.class })
public class FreeMarkerAutoConfiguration {
```

这个自动化配置类被@ConditionalOnClass条件注解修饰，这个条件注解存在的意义在于判断类加载器中是否存在freemarker.template.Configuration和FreeMarkerConfigurationFactory这两个类，如果都存在的话会通过@Import引入参数中配置的配置类及自身配置类；否则不会加载。

## @ConditionalOnClass
以下以@ConditionalOnClass为例来分析条件注解的原理

```
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnClassCondition.class)
public @interface ConditionalOnClass {

	Class<?>[] value() default {};  // 需要匹配的类
	String[] name() default {};  // 需要匹配的类名
}
```
1. 它有2个属性，分别是类数组和字符串数组，指定在classpath下要存在的Class,作用一样，类型不一样。

2. 它被@Conditional注解修饰，参数为OnClassCondition.class，@Conditional实现如下

## @Conditional
```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

    // 参数Class要实现Condition接口
	Class<? extends Condition>[] value();
}
```

此处@Conditional的参数OnClassCondition.class类实现了Condition接口，该接口用于匹配组件是否有资格被容器注册。

以上组合起来，也就是说@Conditional注解属性中可以持有多个Condition接口的实现类，所有的Condition接口实现类需要全部匹配成功后这个@Conditional修饰的组件才有资格被注册，即@ConditionalOnClass的功能实现在OnClassCondition类中，功能是匹配@ConditionalOnClass属性中指定的类是否在classpath存在，如果存在则@ConditionalOnClass修饰的配置类才会生效，否则不会生效。

## Condition接口
该接口用于匹配组件是否有资格被容器注册
```
@FunctionalInterface // Java8的新特性
public interface Condition {

    // ConditionContext内部会存储Spring容器、应用程序环境信息、资源加载器、类加载器
	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```
Condition接口有个子接口ConfigurationCondition：


```
public interface ConfigurationCondition extends Condition {

  ConfigurationPhase getConfigurationPhase();

  public static enum ConfigurationPhase {
  	PARSE_CONFIGURATION, //解析配置阶段
  	REGISTER_BEAN //注册bean阶段
  }
  
}
```

这个子接口是一种特殊的条件接口，多了一个getConfigurationPhase方法，也就是条件注解的生效阶段。只有在ConfigurationPhase中定义的两种阶段下才会生效。

Condition接口有个实现抽象类SpringBootCondition，SpringBoot中所有条件注解对应的条件类都继承这个抽象类。它实现了matches方法：
```
@Override
public final boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
  // 得到类名或者方法名(条件注解可以作用的类或者方法上)
  String classOrMethodName = getClassOrMethodName(metadata); 
  
  try {
    // 抽象方法，具体子类实现。ConditionOutcome记录了匹配结果boolean和log信息
  	ConditionOutcome outcome = getMatchOutcome(context, metadata); 
  	
  	// log记录一下匹配信息
  	logOutcome(classOrMethodName, outcome); 
  	
  	// 报告记录一下匹配信息
  	recordEvaluation(context, classOrMethodName, outcome); 
  	
  	// 返回是否匹配
  	return outcome.isMatch(); 
  }
  catch (NoClassDefFoundError ex) {
  	throw new IllegalStateException(
  			"Could not evaluate condition on " + classOrMethodName + " due to "
  					+ ex.getMessage() + " not "
  					+ "found. Make sure your own configuration does not rely on "
  					+ "that class. This can also happen if you are "
  					+ "@ComponentScanning a springframework package (e.g. if you "
  					+ "put a @ComponentScan in the default package by mistake)",
  			ex);
  }
  catch (RuntimeException ex) {
  	throw new IllegalStateException(
  			"Error processing condition on " + getName(metadata), ex);
  }
}
```

## 基于Class的条件注解
SpringBoot提供了两个基于Class的条件注解：@ConditionalOnClass(类加载器中存在指明的类)或者@ConditionalOnMissingClass(类加载器中不存在指明的类)。

@ConditionalOnClass或者@ConditionalOnMissingClass注解对应的条件类是OnClassCondition，定义如下：
```
@Order(Ordered.HIGHEST_PRECEDENCE) // 优先级、最高级别
class OnClassCondition extends SpringBootCondition {

  @Override
  public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {

    // 记录匹配信息
  	StringBuffer matchMessage = new StringBuffer(); 

    // 得到@ConditionalOnClass注解的属性
  	MultiValueMap<String, Object> onClasses = getAttributes(metadata, ConditionalOnClass.class); 
  	
  	// 如果属性存在
  	if (onClasses != null) { 
  	
  	    // 得到在类加载器中不存在的类
  		List<String> missing = getMatchingClasses(onClasses, MatchType.MISSING, context); 
  		
  		// 如果存在类加载器中不存在对应的类，返回一个匹配失败的ConditionalOutcome
  		if (!missing.isEmpty()) { 
  			return ConditionOutcome.noMatch("required @ConditionalOnClass classes not found: "
  							+ StringUtils.collectionToCommaDelimitedString(missing));
  		}
        
        // 如果类加载器中存在对应的类的话，匹配信息进行记录
  		matchMessage.append("@ConditionalOnClass classes found: "
  				+ StringUtils.collectionToCommaDelimitedString(getMatchingClasses(onClasses, MatchType.PRESENT, context)));
  	}
    
    // 对@ConditionalOnMissingClass注解做相同的逻辑处理
    // 说明: @ConditionalOnClass和@ConditionalOnMissingClass可以一起使用
  	MultiValueMap<String, Object> onMissingClasses = getAttributes(metadata, ConditionalOnMissingClass.class);
  	
  	if (onMissingClasses != null) {
  		List<String> present = getMatchingClasses(onMissingClasses, MatchType.PRESENT, context);
  		if (!present.isEmpty()) {
  			return ConditionOutcome.noMatch("required @ConditionalOnMissing classes found: "
  							+ StringUtils.collectionToCommaDelimitedString(present));
  		}
  		matchMessage.append(matchMessage.length() == 0 ? "" : " ");
  		matchMessage.append("@ConditionalOnMissing classes not found: "
  				+ StringUtils.collectionToCommaDelimitedString(getMatchingClasses(onMissingClasses, MatchType.MISSING, context)));
  	}
        
    // 返回全部匹配成功的ConditionalOutcome
  	return ConditionOutcome.match(matchMessage.toString());
	}

// 枚举：匹配类型。用于查询类名在对应的类加载器中是否存在。
  private enum MatchType { 

  	PRESENT { // 匹配成功
  		@Override
  		public boolean matches(String className, ConditionContext context) {
  			return ClassUtils.isPresent(className, context.getClassLoader());
  		}
  	},

  	MISSING { // 匹配不成功
  		@Override
  		public boolean matches(String className, ConditionContext context) {
  			return !ClassUtils.isPresent(className, context.getClassLoader());
  		}
  	};

  	public abstract boolean matches(String className, ConditionContext context);

  }

}
```
比如FreemarkerAutoConfiguration中的@ConditionalOnClass注解中有value属性是freemarker.template.Configuration.class和FreeMarkerConfigurationFactory.class。在OnClassCondition执行过程中得到的最终ConditionalOutcome中的log message如下：


```
@ConditionalOnClass classes found: freemarker.template.Configuration,org.springframework.ui.freemarker.FreeMarkerConfigurationFactory
```

## 基于Bean的条件注解
@ConditionalOnBean(Spring容器中存在指明的bean)、@ConditionalOnMissingBean(Spring容器中不存在指明的bean)以及ConditionalOnSingleCandidate(Spring容器中存在且只存在一个指明的bean)都是基于Bean的条件注解，它们对应的条件类是ConditionOnBean。

@ConditionalOnBean注解定义如下：
```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional({OnBeanCondition.class})
public @interface ConditionalOnBean {

    // 匹配的bean类型
    Class<?>[] value() default {};

    // 匹配的bean类型的类名
    String[] type() default {};

    // 匹配的bean注解
    Class<? extends Annotation>[] annotation() default {};

    // 匹配的bean的名字
    String[] name() default {};

    // 搜索策略
    // CURRENT(只在当前容器中找)
    // PARENTS(只在所有的父容器中找；但是不包括当前容器)
    // ALL(CURRENT和PARENTS的组合)
    SearchStrategy search() default SearchStrategy.ALL;

    Class<?>[] parameterizedContainer() default {};
}
```
OnBeanCondition条件类的匹配代码如下：
```
@Override
public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
  
  // 记录匹配信息
  StringBuffer matchMessage = new StringBuffer(); 
  
  // 针对@ConditionalOnBean注解
  if (metadata.isAnnotated(ConditionalOnBean.class.getName())) {
  
    // 构造一个BeanSearchSpec，会从@ConditionalOnBean注解中获取属性，然后设置到BeanSearchSpec中
    BeanSearchSpec spec = new BeanSearchSpec(context, metadata, ConditionalOnBean.class); 
    
    // 从BeanFactory中根据策略找出所有匹配的bean
    List<String> matching = getMatchingBeans(context, spec); 
    
    // 如果没有匹配的bean，返回一个没有匹配成功的ConditionalOutcome
    if (matching.isEmpty()) { 
      return ConditionOutcome.noMatch("@ConditionalOnBean " + spec + " found no beans");
    }
    
    // 如果找到匹配的bean，匹配信息进行记录
    matchMessage.append("@ConditionalOnBean " + spec + " found the following " + matching);
  }
  
  // 相同的逻辑，针对@ConditionalOnSingleCandidate注解
  if (metadata.isAnnotated(ConditionalOnSingleCandidate.class.getName())) { 
    BeanSearchSpec spec = new SingleCandidateBeanSearchSpec(context, metadata, ConditionalOnSingleCandidate.class);
    
    List<String> matching = getMatchingBeans(context, spec);
    if (matching.isEmpty()) {
      return ConditionOutcome.noMatch("@ConditionalOnSingleCandidate " + spec + " found no beans");
    }
    
    // 多了一层判断，判断是否只有一个bean
    else if (!hasSingleAutowireCandidate(context.getBeanFactory(), matching)) { 
      return ConditionOutcome.noMatch("@ConditionalOnSingleCandidate " + spec
          + " found no primary candidate amongst the" + " following " + matching);
    }
    
    matchMessage.append("@ConditionalOnSingleCandidate " + spec + " found "
        + "a primary candidate amongst the following " + matching);
  }
  
  // 相同的逻辑，针对@ConditionalOnMissingBean注解
  if (metadata.isAnnotated(ConditionalOnMissingBean.class.getName())) { 
    BeanSearchSpec spec = new BeanSearchSpec(context, metadata, ConditionalOnMissingBean.class);
    
    List<String> matching = getMatchingBeans(context, spec);
    if (!matching.isEmpty()) {
      return ConditionOutcome.noMatch("@ConditionalOnMissingBean " + spec
          + " found the following " + matching);
    }
    
    matchMessage.append(matchMessage.length() == 0 ? "" : " ");
    matchMessage.append("@ConditionalOnMissingBean " + spec + " found no beans");
  }
  
  // 返回匹配成功的ConditonalOutcome
  return ConditionOutcome.match(matchMessage.toString()); 
}
```
SpringBoot还提供了其他比如ConditionalOnJava、ConditionalOnNotWebApplication、ConditionalOnWebApplication、ConditionalOnResource、ConditionalOnProperty、ConditionalOnExpression等条件注解，有兴趣的读者可以自行查看它们的底层处理逻辑。

##  条件注解的激活机制 
分析完了条件注解的执行逻辑之后，接下来的问题就是SpringBoot是如何让这些条件注解生效的？

SpringBoot使用ConditionEvaluator这个内部类完成条件注解的解析和判断。

在Spring容器的refresh过程中，只有跟解析或者注册bean有关系的类都会使用ConditionEvaluator完成条件注解的判断，这个过程中一些类不满足条件的话就会被skip。这些类比如有AnnotatedBeanDefinitionReader、ConfigurationClassBeanDefinitionReader、ConfigurationClassParse、ClassPathScanningCandidateComponentProvider等。

比如AnnotatedBeanDefinitionReader的构造函数会初始化内部属性conditionEvaluator：
```
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
	Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
	Assert.notNull(environment, "Environment must not be null");
	this.registry = registry;
	
	 // 构造ConditionEvaluator用于处理条件注解
	this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
	AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

AnnotatedBeanDefinitionReader的doRegisterBean方法中，对每个配置类进行解析的时候都会使用ConditionEvaluator
```
AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(annotatedClass);
if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
	return;
}
```

ConditionEvaluator的shouldSkip()方法实现
```
public boolean shouldSkip(@Nullable AnnotatedTypeMetadata metadata, @Nullable ConfigurationPhase phase) {
		// 如果这个类没有被@Conditional注解所修饰，返回false不会skip
		if (metadata == null || !metadata.isAnnotated(Conditional.class.getName())) {
			return false;
		}

        // 如果参数中沒有设置条件注解的生效阶段
		if (phase == null) {
		
		    // 是配置类的话直接使用PARSE_CONFIGURATION阶段，递归调用
			if (metadata instanceof AnnotationMetadata &&
					ConfigurationClassUtils.isConfigurationCandidate((AnnotationMetadata) metadata)) {
				return shouldSkip(metadata, ConfigurationPhase.PARSE_CONFIGURATION);
			}
			
			// 否则使用REGISTER_BEAN阶段，递归调用
			return shouldSkip(metadata, ConfigurationPhase.REGISTER_BEAN);
		}

		List<Condition> conditions = new ArrayList<>();
		
		 // 获取配置类的条件注解上的条件数据，并添加到集合conditions中
		for (String[] conditionClasses : getConditionClasses(metadata)) {
			for (String conditionClass : conditionClasses) {
				Condition condition = getCondition(conditionClass, this.context.getClassLoader());
				conditions.add(condition);
			}
		}

        // 对条件集合做个排序
		AnnotationAwareOrderComparator.sort(conditions);

        // 遍历条件集合
		for (Condition condition : conditions) {
			ConfigurationPhase requiredPhase = null;
			
			// 如果是ConfigurationCondition，获取配置阶段
			if (condition instanceof ConfigurationCondition) {
				requiredPhase = ((ConfigurationCondition) condition).getConfigurationPhase();
			}
			
			// 没有配置阶段或配置阶段一致，调用condition.matches()进行匹配，如果不匹配，则返回true,会skip
			if ((requiredPhase == null || requiredPhase == phase) && !condition.matches(this.context, metadata)) {
				return true;
			}
		}

        // 默认不skip
		return false;
	}
```
