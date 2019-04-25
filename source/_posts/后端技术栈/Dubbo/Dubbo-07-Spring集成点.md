---
title: Dubbo-07-Spring集成点
date: 2018-05-16 13:13:17
tags: Dubbo
---

### Spring-xml标签解析
用过Spring就知道可以在xml文件中进行如下配置：
```
<context:component-scan base-package="com.demo.dubbo.server.serviceimpl"/>

<context:property-placeholder location="classpath:config.properties"/>

<tx:annotation-driven transaction-manager="transactionManager"/>
```
Spring是如何来解析这些配置呢？如果我们想自己定义配置该如何做呢？

对于上述的xml配置，分成三个部分

- 命名空间namespace，如tx、context
- 元素element，如component-scan、property-placeholder、annotation-driven
- 属性attribute，如base-package、location、transaction-manager

Spring定义了两个接口，来分别解析上述内容：

- NamespaceHandler：注册了一堆BeanDefinitionParser，利用他们来进行解析
- BeanDefinitionParser: 用于解析每个element的内容

如对于spring的context命名空间为例，对应的NamespaceHandler实现是ContextNamespaceHandler
```
public class ContextNamespaceHandler extends NamespaceHandlerSupport {

	@Override
	public void init() {
		registerBeanDefinitionParser("property-placeholder", new PropertyPlaceholderBeanDefinitionParser());
		registerBeanDefinitionParser("property-override", new PropertyOverrideBeanDefinitionParser());
		registerBeanDefinitionParser("annotation-config", new AnnotationConfigBeanDefinitionParser());
		registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());
		registerBeanDefinitionParser("load-time-weaver", new LoadTimeWeaverBeanDefinitionParser());
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
		registerBeanDefinitionParser("mbean-export", new MBeanExportBeanDefinitionParser());
		registerBeanDefinitionParser("mbean-server", new MBeanServerBeanDefinitionParser());
	}

}
```
注册了一堆BeanDefinitionParser，如果我们想看"component-scan"是如何实现的，就可以去看对应的ComponentScanBeanDefinitionParser的源码

### dubbo命名空间解析
如果自定义了NamespaceHandler，如何加入到Spring中呢？

Spring默认会在加载jar包下的 META-INF/spring.handlers文件下寻找NamespaceHandler实现。

* dubbo命名空间的自定义NamespaceHandler是DubboNamespaceHandler

```
public class DubboNamespaceHandler extends NamespaceHandlerSupport {

    static {
        Version.checkDuplicate(DubboNamespaceHandler.class);
    }

    public void init() {
        registerBeanDefinitionParser("application", new DubboBeanDefinitionParser(ApplicationConfig.class, true));
        registerBeanDefinitionParser("module", new DubboBeanDefinitionParser(ModuleConfig.class, true));
        registerBeanDefinitionParser("registry", new DubboBeanDefinitionParser(RegistryConfig.class, true));
        registerBeanDefinitionParser("monitor", new DubboBeanDefinitionParser(MonitorConfig.class, true));
        registerBeanDefinitionParser("provider", new DubboBeanDefinitionParser(ProviderConfig.class, true));
        registerBeanDefinitionParser("consumer", new DubboBeanDefinitionParser(ConsumerConfig.class, true));
        registerBeanDefinitionParser("protocol", new DubboBeanDefinitionParser(ProtocolConfig.class, true));
        registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));
        registerBeanDefinitionParser("reference", new DubboBeanDefinitionParser(ReferenceBean.class, false));
        registerBeanDefinitionParser("annotation", new AnnotationBeanDefinitionParser());
    }

}
```
给出的BeanDefinitionParser全部是DubboBeanDefinitionParser，如果我们想看看<dubbo:registry>是怎么解析的，就可以去看看DubboBeanDefinitionParser的源代码。

dubbo的jar包下，存在着META-INF/spring.handlers文件，内容如下：

```
http\://code.alibabatech.com/schema/dubbo=com.alibaba.dubbo.config.spring.schema.DubboNamespaceHandler
```
具体解析过程就不再说明了。结果就是不同的配置分别转换成Spring容器中的一个bean对象。

- application对应ApplicationConfig
- registry对应RegistryConfig
- monitor对应MonitorConfig
- provider对应ProviderConfig
- consumer对应ConsumerConfig
- protocol对应ProtocolConfig
- service对应ServiceConfig
- reference对应ReferenceConfig

上面的对象不依赖Spring，也就是说你可以手动去创建上述对象。

- 为了在Spring启动的时候，也相应的启动provider发布服务注册服务的过程：又加入了一个和Spring相关联的ServiceBean，继承了ServiceConfig。
- 为了在Spring启动的时候，也相应的启动consumer发现服务的过程：又加入了一个和Spring相关联的ReferenceBean，继承了ReferenceConfig

利用Spring就做了上述过程，得到相应的配置数据，然后启动相应的服务。如果想剥离Spring，我们就可以手动来创建上述配置对象，通过ServiceConfig和ReferenceConfig的API来启动相应的服务