---
title: Spring-18-内置的BeanPostProcessor总结
date: 2018-12-14 15:59:05
tags: Spring
---

Spring内置了一些很有用的BeanPostProcessor接口实现类。比如有AutowiredAnnotationBeanPostProcessor、RequiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor、EventListenerMethodProcessor等。这些Processor会处理各自的场景。

正是有了这些processor，把bean的构造过程中的一部分功能分配给了这些processor处理，减轻了BeanFactory的负担。

而且添加一些新的功能也很方便。比如Spring Scheduling模块，只需要添加个@EnableScheduling注解，然后加个@Scheduled注解修饰的方法即可，这个Processor内部会自行处理。

#### 目录
- ApplicationContextAwareProcessor
- CommonAnnotationBeanPostProcessor
- AutowiredAnnotationBeanPostProcessor
- RequiredAnnotationBeanPostProcessor
- BeanValidationPostProcessor
- AbstractAutoProxyCreator系列类
- MethodValidationPostProcessor
- ScheduledAnnotationBeanPostProcessor
- AsyncAnnotationBeanPostProcessor
- ServletContextAwareProcessor

## ApplicationContextAwareProcessor
ApplicationContextAwareProcessor实现BeanPostProcessor接口。

Spring容器的refresh方法内部调用prepareBeanFactory方法，prepareBeanFactory方法会添加ApplicationContextAwareProcessor到BeanFactory中。这个Processor的作用在于为实现*Aware接口的bean调用该Aware接口定义的方法，并传入对应的参数。比如实现EnvironmentAware接口的bean在该Processor内部会调用EnvironmentAware接口的setEnvironment方法，并把Spring容器内部的ConfigurableEnvironment传递进去。

具体的代码：


```
// AbstractApplicationContext.class
beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));


// ApplicationContextAwareProcessor.class
private void invokeAwareInterfaces(Object bean) {
  if (bean instanceof Aware) {
    if (bean instanceof EnvironmentAware) {
      ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
    }
    if (bean instanceof EmbeddedValueResolverAware) {
      ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(
          new EmbeddedValueResolver(this.applicationContext.getBeanFactory()));
    }
    if (bean instanceof ResourceLoaderAware) {
      ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
    }
    if (bean instanceof ApplicationEventPublisherAware) {
      ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
    }
    if (bean instanceof MessageSourceAware) {
      ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
    }
    if (bean instanceof ApplicationContextAware) {
      ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
    }
  }
}
```

## CommonAnnotationBeanPostProcessor
SpringApplication#run()方法中的prepareContext() # load() # createBeanDefinitionLoader() #  new BeanDefinitionLoader() # new AnnotatedBeanDefinitionReader() 中 有如下代码：
```
AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
```
其中会向BeanDefinitionRegistry注册如下BeanPostProcessor:
1. ConfigurationClassPostProcessor
1. AutowiredAnnotationBeanPostProcessor
1. CommonAnnotationBeanPostProcessor
1. EventListenerMethodProcessor
1. DefaultEventListenerFactory

其中第3个就是本节要说明的，主要处理@Resource、@PostConstruct和@PreDestroy注解的实现。

在postProcessPropertyValues过程中，该processor会找出bean中被@Resource注解修饰的属性(Field)和方法(Method)，找出以后注入到bean中。


```
// CommonAnnotationBeanPostProcessor.class
@Override
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
    
    // 找出bean中被@Resource注解修饰的属性(Field)和方法(Method)
	InjectionMetadata metadata = findResourceMetadata(beanName, bean.getClass(), pvs);
	try {
	    
	    // 注入到bean中
		metadata.inject(bean, beanName, pvs);
	}
	catch (Throwable ex) {
		throw new BeanCreationException(beanName, "Injection of resource dependencies failed", ex);
	}
	return pvs;
}

// since spring 5.1过时
@Deprecated
@Override
public PropertyValues postProcessPropertyValues(
		PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) {

	return postProcessProperties(pvs, bean, beanName);
}
```
CommonAnnotationBeanPostProcessor的父类InitDestroyAnnotationBeanPostProcessor类的postProcessBeforeInitialization过程中会执行被@PostConstruct注解修饰的方法
```
// BeanPostProcessor的postProcessBeforeInitialization方法在bean的构造方法执行完，
// 且该注入的依赖都注入完成,是一个完整的bean
// 在其中执行@PostConstruct修饰的方法
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {

    // 找出被@PostConstruct或@PreDestroy注解修饰的方法
	LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
	try {
	    // 其中只执行@PostConstruct修饰的方法
		metadata.invokeInitMethods(bean, beanName);
	}
	catch (InvocationTargetException ex) {
		throw new BeanCreationException(beanName, "Invocation of init method failed", ex.getTargetException());
	}
	catch (Throwable ex) {
		throw new BeanCreationException(beanName, "Failed to invoke init method", ex);
	}
	return bean;
}

```

InitDestroyAnnotationBeanPostProcessor实现了DestructionAwareBeanPostProcessor接口，该接口中的postProcessBeforeDestruction方法在bean被销毁之前调用，@PreDestroy注解修饰的方法就是在此时调用。
```
@Override
public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
	
	// 找出被@PostConstruct或@PreDestroy注解修饰的方法
	LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
	try {
	    
	    // 但只会执行被@PreDestroy注解修饰的方法
		metadata.invokeDestroyMethods(bean, beanName);
	}
	catch (InvocationTargetException ex) {
		String msg = "Destroy method on bean with name '" + beanName + "' threw an exception";
		if (logger.isDebugEnabled()) {
			logger.warn(msg, ex.getTargetException());
		}
		else {
			logger.warn(msg + ": " + ex.getTargetException());
		}
	}
	catch (Throwable ex) {
		logger.warn("Failed to invoke destroy method on bean with name '" + beanName + "'", ex);
	}
}
```

## AutowiredAnnotationBeanPostProcessor
跟CommonAnnotationBeanPostProcessor一样，在AnnotationConfigUtils类的registerAnnotationConfigProcessors方法被注册到Spring容器中。

主要处理@Autowired、@Value、@Lookup和@Inject注解的实现，处理逻辑跟CommonAnnotationBeanPostProcessor类似。
```
// 在bean构造方法执行完后，其属性被设置到目标实例之前调用，可以修改属性的设置
@Override
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
	// 找出被@Autowired、@Value以及@Inject注解修饰的属性和方法
	InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
	try {
	
	    // 注入到bean中
		metadata.inject(bean, beanName, pvs);
	}
	catch (BeanCreationException ex) {
		throw ex;
	}
	catch (Throwable ex) {
		throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);
	}
	return pvs;
}

// since 5.1 过时接口方法
@Deprecated
@Override
public PropertyValues postProcessPropertyValues(
		PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) {

	return postProcessProperties(pvs, bean, beanName);
}
```
由于@Autowired注解可以在构造器中使用，所以AutowiredAnnotationBeanPostProcessor实现了determineCandidateConstructors方法：

**对于必须的属性设置，应该用@Autowired在构造方法中注入。**
```
@Override
public Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, final String beanName) throws BeansException {
  ...
  for (Constructor<?> candidate : rawCandidates) { // 遍历所有的构造器
    // 找出被@Autowired注解修饰的构造器
    AnnotationAttributes ann = findAutowiredAnnotation(candidate);
    if (ann != null) {
      ...
      candidates.add(candidate);
    }
    else if (candidate.getParameterTypes().length == 0) {
      defaultConstructor = candidate;
    }
  }
  if (!candidates.isEmpty()) { // 有找到的话使用这些构造器
    ...
    candidateConstructors = candidates.toArray(new Constructor<?>[candidates.size()]);
  }
  else { // 否则使用默认的构造器
    candidateConstructors = new Constructor<?>[0];
  }
  ...
}
```

## RequiredAnnotationBeanPostProcessor
跟CommonAnnotationBeanPostProcessor一样，在AnnotationConfigUtils类的registerAnnotationConfigProcessors方法被注册到Spring容器中。

主要处理@Required注解的实现，@Required注解只能修饰方法

该类自从5.1开始过时，Spring推荐，对于必须要的属性设置，建议用@Autowired在其构造方法上实现注入。

## BeanValidationPostProcessor
默认不添加，需要手动添加。主要提供对JSR-303验证的支持，内部有个boolean类型的属性afterInitialization，默认是false。如果是false，在postProcessBeforeInitialization过程中对bean进行验证，否则在postProcessAfterInitialization过程对bean进行验证。


```
// 手动注册BeanValidationPostProcessor
@Bean
public BeanPostProcessor beanValidationPostProcessor() {
    return new BeanValidationPostProcessor();
}
```

定义一个使用JSR-303的bean：


```
@Component
public class BeanForBeanValidation {
    @NotNull
    private String id;
    @Min(value = 10)
    private int age;
}
```

最后实例化BeanForBeanValidation的时候，BeanValidationPostProcessor起作用，在postProcessBeforeInitialization过程中发现validate不通过，抛出异常：
```
Caused by: org.springframework.beans.factory.BeanInitializationException: Bean state is invalid: age - 最小不能小于10; id - 不能为null
```

## AbstractAutoProxyCreator系列类
这是一个抽象类，实现了SmartInstantiationAwareBeanPostProcessor接口。主要用于aop在Spring中的应用。默认情况下，AbstractAutoProxyCreator相关的BeanPostProcessor是不会注册到Spring容器中的。

只有在SpringBoot中加入aop-starter之后，会触发AopAutoConfiguration自动化配置，然后将AnnotationAwareAspectJAutoProxyCreator注册到Spring容器中。

AopAutoConfiguration自动化配置类如下：
```
@Configuration
@ConditionalOnClass({ EnableAspectJAutoProxy.class, Aspect.class, Advice.class,
		AnnotatedElement.class })
@ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
public class AopAutoConfiguration {

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = false)
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false", matchIfMissing = false)
	public static class JdkDynamicAutoProxyConfiguration {

	}

	@Configuration
	@EnableAspectJAutoProxy(proxyTargetClass = true)
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true", matchIfMissing = true)
	public static class CglibAutoProxyConfiguration {

	}
}
```
默认会生效@EnableAspectJAutoProxy(proxyTargetClass = true)，该注解中又引入了@Import(AspectJAutoProxyRegistrar.class)

AspectJAutoProxyRegistrar会注册一个AnnotationAwareAspectJAutoProxyCreator到当前的BeanDefinitionRegistry，AnnotationAwareAspectJAutoProxyCreator是AbstractAutoProxyCreator的一个子类，它会扫描出Spring容器中带有@Aspect注解的bean，然后在getAdvicesAndAdvisorsForBean方法中会根据这个aspect查看是否被拦截，如果被拦截那么就wrap成代理类。

## MethodValidationPostProcessor
支持方法级别的JSR-303规范校验。

通过条件注解，判断classpath下是否存在javax.validation.spi.ValidationProvider的SPI机制的实现类，来自动注册一个MethodValidationPostProcessor实例到Spring容器中，自动配置类如下：

```
@Configuration
@ConditionalOnClass(ExecutableValidator.class)
@ConditionalOnResource(resources = "classpath:META-INF/services/javax.validation.spi.ValidationProvider")
@Import(PrimaryDefaultValidatorPostProcessor.class)
public class ValidationAutoConfiguration {

	@Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	@ConditionalOnMissingBean(Validator.class)
	public static LocalValidatorFactoryBean defaultValidator() {
		LocalValidatorFactoryBean factoryBean = new LocalValidatorFactoryBean();
		MessageInterpolatorFactory interpolatorFactory = new MessageInterpolatorFactory();
		factoryBean.setMessageInterpolator(interpolatorFactory.getObject());
		return factoryBean;
	}

	@Bean
	@ConditionalOnMissingBean
	public static MethodValidationPostProcessor methodValidationPostProcessor(
			Environment environment, @Lazy Validator validator) {
		MethodValidationPostProcessor processor = new MethodValidationPostProcessor();
		boolean proxyTargetClass = environment
				.getProperty("spring.aop.proxy-target-class", Boolean.class, true);
		processor.setProxyTargetClass(proxyTargetClass);
		processor.setValidator(validator);
		return processor;
	}

}
```

使用时需要在类上加上@Validated注解，以及在方法的参数中加上验证注解，比如@Max，@Min，@NotEmpty …。 下面这个BeanForMethodValidation就加上了@Validated注解，并且在方法validate的参数里加上的JSR-303的验证注解。


```
@Component
@Validated
public class BeanForMethodValidation {
    public void validate(@NotEmpty String name, @Min(10) int age) {
        System.out.println("validate, name: " + name + ", age: " + age);
    }
}
```

MethodValidationPostProcessor内部使用aop完成对方法的调用。


```
// MethodValidationPostProcessor.class
@Override
public void afterPropertiesSet() {
  // 基于validatedAnnotationType属性构造出Pointcut，这个validatedAnnotationType属性默认是@Validated注解类型，可以进行修改
  Pointcut pointcut = new AnnotationMatchingPointcut(this.validatedAnnotationType, true);
  // 基于Pointcut和Advice构造出Advisor
  this.advisor = new DefaultPointcutAdvisor(pointcut, createMethodValidationAdvice(this.validator));
}
// MethodValidationInterceptor这个Advice内部使用JSR完成方法参数的验证
protected Advice createMethodValidationAdvice(Validator validator) {
	return (validator != null ? new MethodValidationInterceptor(validator) : new MethodValidationInterceptor());
}
```

## ScheduledAnnotationBeanPostProcessor
默认不添加，使用@EnableScheduling注解后，会被注册到Spring容器中。主要使用Spring Scheduling功能对bean中使用了@Scheduled注解的方法进行调度处理。实现了BeanPostProcessor接口。
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(SchedulingConfiguration.class)
@Documented
public @interface EnableScheduling {

}

// SchedulingConfiguration.class
@Configuration
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public class SchedulingConfiguration {

    // name = org.springframework.context.annotation.internalScheduledAnnotationProcessor
	@Bean(name = TaskManagementConfigUtils.SCHEDULED_ANNOTATION_PROCESSOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public ScheduledAnnotationBeanPostProcessor scheduledAnnotationProcessor() {
		return new ScheduledAnnotationBeanPostProcessor();
	}
}
```
找出class中带有@Scheduled注解的方法，进行调度处理
```
@Override
public Object postProcessAfterInitialization(final Object bean, String beanName) {
  // 判断是否是代理类，如果是代理类，拿到真正的目标类
  Class<?> targetClass = AopUtils.getTargetClass(bean);
  // 判断是否已经处理过。nonAnnotatedClasses属性是个Class集合，用于存储bean对应的class是否有@Scheduled注解的方法，如果没有，则添加到这个集合中
  if (!this.nonAnnotatedClasses.contains(targetClass)) {
    // 找出class中带有@Scheduled注解的方法
    Map<Method, Set<Scheduled>> annotatedMethods = MethodIntrospector.selectMethods(targetClass,
        new MethodIntrospector.MetadataLookup<Set<Scheduled>>() {
          @Override
          public Set<Scheduled> inspect(Method method) {
            Set<Scheduled> scheduledMethods =
                AnnotationUtils.getRepeatableAnnotations(method, Scheduled.class, Schedules.class);
            return (!scheduledMethods.isEmpty() ? scheduledMethods : null);
          }
        });
    // 如果不存在@Scheduled注解的方法
    if (annotatedMethods.isEmpty()) {
      // 添加到nonAnnotatedClasses集合中。下次不用重复处理该类
      this.nonAnnotatedClasses.add(targetClass);
      if (logger.isTraceEnabled()) {
        logger.trace("No @Scheduled annotations found on bean class: " + bean.getClass());
      }
    }
    else { // 如果存在@Scheduled注解的方法
      // 遍历这些@Scheduled注解的方法
      for (Map.Entry<Method, Set<Scheduled>> entry : annotatedMethods.entrySet()) {
        Method method = entry.getKey();
        for (Scheduled scheduled : entry.getValue()) {
          // 进行调度处理
          processScheduled(scheduled, method, bean);
        }
      }
      if (logger.isDebugEnabled()) {
        logger.debug(annotatedMethods.size() + " @Scheduled methods processed on bean '" + beanName +
            "': " + annotatedMethods);
      }
    }
  }
  return bean;
}
```


## AsyncAnnotationBeanPostProcessor
默认不添加，使用@EnableAsync注解后，会被注册到Spring容器中。AsyncAnnotationBeanPostProcessor内部使用aop处理方法的调用，Pointcut会找出带有@Async的类和@Async的方法。

```
// AsyncAnnotationBeanPostProcessor.class
// 实现了BeanFactoryAware接口，这里会得到beanFactory
@Override
public void setBeanFactory(BeanFactory beanFactory) {
  super.setBeanFactory(beanFactory);
  // 构造一个AsyncAnnotationAdvisor
  // AsyncAnnotationAdvisor内部的Advice是AnnotationAsyncExecutionInterceptor，Pointcut会找出带有@Async的类和@Async的方法
  AsyncAnnotationAdvisor advisor = new AsyncAnnotationAdvisor(this.executor, this.exceptionHandler);
  if (this.asyncAnnotationType != null) {
    advisor.setAsyncAnnotationType(this.asyncAnnotationType);
  }
  advisor.setBeanFactory(beanFactory);
  this.advisor = advisor;
}

```

```
// AsyncExecutionInterceptor.class。 AnnotationAsyncExecutionInterceptor的父类
@Override
public Object invoke(final MethodInvocation invocation) throws Throwable {
  // 得到方法的对应类
  Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);
  // 得到方法
  Method specificMethod = ClassUtils.getMostSpecificMethod(invocation.getMethod(), targetClass);
  final Method userDeclaredMethod = BridgeMethodResolver.findBridgedMethod(specificMethod);
  // 得到Executor线程池。如果没有在Spring容器中找到TaskExecutor类型的线程池，直接构造一个SimpleAsyncTaskExecutor
  AsyncTaskExecutor executor = determineAsyncExecutor(userDeclaredMethod);
  if (executor == null) {
    throw new IllegalStateException("No executor specified and no default executor set on AsyncExecutionInterceptor either");
  }
  // 把方法在调用封装到Callable中
  Callable<Object> task = new Callable<Object>() {
    @Override
    public Object call() throws Exception {
      try {
        Object result = invocation.proceed();
        if (result instanceof Future) {
          return ((Future<?>) result).get();
        }
      }
      catch (ExecutionException ex) {
        handleError(ex.getCause(), userDeclaredMethod, invocation.getArguments());
      }
      catch (Throwable ex) {
        handleError(ex, userDeclaredMethod, invocation.getArguments());
      }
      return null;
    }
  };
  // 提交任务
  return doSubmit(task, executor, invocation.getMethod().getReturnType());
}
```

## ServletContextAwareProcessor
默认不添加，如果Spring容器是个Web容器，那么会被添加。比如GenericWebApplicationContext容器就在postProcessBeanFactory中添加了ServletContextAwareProcessor。postProcessBeanFactory方法是在Spring容器的refresh过程中被调用的。

ServletContextAwareProcessor实现了BeanPostProcessor接口，如果Spring容器中的bean实现了ServletContextAware或ServletConfigAware接口，那么会进行处理。


```
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
  if (getServletContext() != null && bean instanceof ServletContextAware) {
    ((ServletContextAware) bean).setServletContext(getServletContext());
  }
  if (getServletConfig() != null && bean instanceof ServletConfigAware) {
    ((ServletConfigAware) bean).setServletConfig(getServletConfig());
  }
  return bean;
}
```
