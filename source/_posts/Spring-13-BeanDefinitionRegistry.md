---
title: Spring-13-BeanDefinitionRegistry
date: 2018-12-14 15:56:42
tags: Spring
---

在介绍本文主体BeanDefinitionRegistry之前，先看一下BeanDefinition，两者有紧密的联系。

## BeanDefinition接口
```
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
    
    void setBeanClassName(@Nullable String beanClassName);

	@Nullable
	String getBeanClassName();

	void setScope(@Nullable String scope);

	@Nullable
	String getScope();

	void setLazyInit(boolean lazyInit);

	boolean isLazyInit();

	void setDependsOn(@Nullable String... dependsOn);

	@Nullable
	String[] getDependsOn();
	
	// 省略。。。
}
```

可以看到上面的很多属性和方法都很熟悉，基本上和用xml配置中的<bean>标签支持的配置一致，例如类名、scope、属性、构造函数参数列表、依赖的bean、是否是单例类、是否是懒加载等，其实就是将Bean的定义信息存储到这个BeanDefinition相应的属性中，后面对Bean的操作就直接对BeanDefinition进行，例如拿到这个BeanDefinition后，可以根据里面的类名、构造函数、构造函数参数，使用反射进行对象创建。

BeanDefinition是一个接口，是一个抽象的定义，实际使用的是其实现类，如 ChildBeanDefinition、RootBeanDefinition、GenericBeanDefinition等。

- BeanDefinition继承了AttributeAccessor，说明它具有处理属性的能力；
- BeanDefinition继承了BeanMetadataElement，说明它可以持有Bean元数据元素，作用是可以持有XML文件的一个bean标签对应的Object。

##### BeanDefinition继承图

![BeanDefinition继承图](https://note.youdao.com/yws/api/personal/file/B167970D42B44A1C83B888CA8E31FDA1?method=download&shareKey=6f150d3b7dbb8fd37a107d048080af95)

1. AnnotatedBeanDefinition接口是BeanDefinition接口的子接口，包括了实例的注解信息
2. ScannedGenericBeanDefinition类是AnnotatedBeanDefinition接口的实现类，ClassPathScanningCandidateComponentProvider扫描出来的类信息就会封装成ScannedGenericBeanDefinition，这是一个被扫描到的类定义
3. AnnotatedGenericBeanDefinition类也是AnnotatedBeanDefinition接口的实现类，在AnnotatedBeanDefinitionReader读取器读取类后封装成的，这是一个被注解过的类定义

## BeanDefinitionRegistry接口
见名知意，BeanDefinitionRegistry是BeanDefinition的一个注册中心，提供了注册、移除、获取等方法，其父接口定义了给BeanDefinition设置别名的一些方法。
```
public interface BeanDefinitionRegistry extends AliasRegistry {

	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition) throws BeanDefinitionStoreException;

	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	boolean containsBeanDefinition(String beanName);

	String[] getBeanDefinitionNames();

	int getBeanDefinitionCount();

    // beanName（标识）是否被占用
	boolean isBeanNameInUse(String beanName);
}

public interface AliasRegistry {

	void registerAlias(String name, String alias);

	void removeAlias(String alias);

	boolean isAlias(String name);

	String[] getAliases(String name);
}
```
1. SimpleBeanDefinitionRegistry是一个简单的BeanDefinitionRegistry接口的实现类，内部使用Map<String, BeanDefinition>类型的map存储BeanDefinition信息
2. DefaultListableBeanFactory这个BeanFactory接口的实现类也实现了BeanDefinitionRegistry接口，内部也是使用Map<String, BeanDefinition>类型的map存储BeanDefinition信息
3. 其它的BeanFactory或ApplicationContext都通过间接的组合或继承来使用DefaultListableBeanFactory对该功能实现。


## registerBeanDefinition()实现
DefaultListableBeanFactory的实现方法

成员变量：

```
	/** Map of bean definition objects, keyed by bean name. */
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
	
	/** List of bean definition names, in registration order. */
	private volatile List<String> beanDefinitionNames = new ArrayList<>(256);
	
	/** List of names of manually registered singletons, in registration order. */
	private volatile Set<String> manualSingletonNames = new LinkedHashSet<>(16);
```
方法实现：
```

public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
    throws BeanDefinitionStoreException {

    if (beanDefinition instanceof AbstractBeanDefinition) {
        try {
            // BeanDefinition 校验
            ((AbstractBeanDefinition) beanDefinition).validate();
 
        }catch (BeanDefinitionValidationException ex) {
            // 抛出异常...
        }
    }
 
    BeanDefinition oldBeanDefinition;
 
    oldBeanDefinition = this.beanDefinitionMap.get(beanName);
    if (oldBeanDefinition != null) {
        if (!isAllowBeanDefinitionOverriding()) {
            // 抛出异常...
        }else if (oldBeanDefinition.getRole() < beanDefinition.getRole()) {
            // 日志输出...
        }
        else if (!beanDefinition.equals(oldBeanDefinition)) {
            // 日志输出...
        }
        else {
            // 日志输出...
        }
    }else {
        // 添加标识进 List
        this.beanDefinitionNames.add(beanName);
        this.manualSingletonNames.remove(beanName);
        this.frozenBeanDefinitionNames = null;
    }
    // 关键 -> 添加进 map
    this.beanDefinitionMap.put(beanName, beanDefinition);
 
    if (oldBeanDefinition != null || containsSingleton(beanName)) {
        resetBeanDefinition(beanName);
    }
}
```
