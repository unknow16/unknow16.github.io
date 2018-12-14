---
title: Spring-14-BeanDefinitionRegistryPostProcessor
date: 2018-12-14 15:57:00
tags: Spring
---

BeanDefinitionRegistryPostProcessor跟其他PostProcessor一样，见名知意，其提供了允许我们操作BeanDefinitionRegistry的入口，其接口定义如下：
```
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {
    
    // 通过BeanDefinitionRegistry可以修改BeanDefinition的属性，从而影响实例化出来的bean
	void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;
}
```
可见BeanDefinitionRegistryPostProcessor继承了BeanFactoryPostProcessor接口，两者提供的供用户自定义的入口不一样，前者提供了BeanDefinitionRegistry，即可修改生成bean实例的模版定义，后者提供了ConfigurableListableBeanFactory，也可获取BeanDefinition，但顺序相对前者更靠后。

Spring官方解释是:BeanDefinitionRegistryPostProcessor允许在正常的BeanFactoryPostProcessor检测开始之前注册更多的自定义bean。特别是，BeanDefinitionRegistryPostProcessor可以注册更多的BeanDefinition，然后定义BeanFactoryPostProcessor实例(因为继承自它)。也就是说可以借此方法实现自定义的bean。

调用该接口实现时，所有的BeanDefinition已经加载到BeanDefinitionRegistry完毕，但是还没有通过这些BeanDefinition实例化出bean实例，此时我们可以在此处修改BeanDefinition，从而影响实例化出来的bean。

## 示例
通过新建BeanDefinition直接注册上去。通过@Autowire直接就可以获取。
```
@Configuration
public class BeanConfiger implements BeanDefinitionRegistryPostProcessor,ApplicationContextAware{
    
    private ApplicationContext applicationContext;
    private final static Logger LOGGER = LoggerFactory.getLogger(BeanConfiger.class);
    
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        LOGGER.info("postProcessBeanDefinitionRegistry执行！");
        
        /*RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(UserDomain.class);
        rootBeanDefinition.setAutowireMode(RootBeanDefinition.AUTOWIRE_BY_TYPE);
        rootBeanDefinition.getPropertyValues().add("name","pepsi");
        registry.registerBeanDefinition("userDomain",rootBeanDefinition);*/

        //使用不同beanDefinition
        Class<?> cls = UserDomain.class;
        BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(cls);
        GenericBeanDefinition definition = (GenericBeanDefinition) builder.getRawBeanDefinition();
        definition.setAutowireMode(GenericBeanDefinition.AUTOWIRE_BY_TYPE);
        definition.getPropertyValues().add("name","pepsi02");
        // 注册bean名,一般为类名首字母小写
        registry.registerBeanDefinition("userDomain", definition);
    }
    
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        LOGGER.info("postProcessBeanFactory() 执行！");
    }
    
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```
