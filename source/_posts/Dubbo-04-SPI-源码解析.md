---
title: Dubbo-04-SPI-源码解析
date: 2018-05-11 14:52:47
tags: Dubbo
---

### 功能介绍

dubbo的扩展机制和java的SPI机制非常相似，但是又增加了如下功能：

1. 可以方便的获取某一个想要的扩展实现，java的SPI机制就没有提供这样的功能
2. 对于扩展实现IOC依赖注入功能：

举例来说：接口A，实现者A1、A2。接口B，实现者B1、B2。

现在实现者A1含有setB()方法，会自动注入一个接口B的实现者，此时注入B1还是B2呢？都不是，而是注入一个动态生成的接口B的实现者B$Adpative，该实现者能够根据参数的不同，自动引用B1或者B2来完成相应的功能

3. 对扩展采用装饰器模式进行功能增强，类似AOP实现的功能

还是以上面的例子，接口A的另一个实现者AWrapper1。大体内容如下：


```
public class AWrapper1 {
    private A a; 
    
    AWrapper1（A a）{
        this.a=a;
    } 
}

```


因此，我们在获取某一个接口A的实现者A1的时候，已经自动被AWrapper1包装了。

### 解析扩展过程

##### 1. 解析入口
```
 ExtensionLoader extensionLoader = ExtensionLoader.getExtensionLoader(Protocol.class);
 Protocol protocol = extensionLoader.getAdaptiveExtension();  
```
##### 2. 调用 ExtensionLoader静态方法getExtensionLoader
ExtensionLoader中有一个如下属性，用于缓存所有的扩展加载实例

```
    private static final ConcurrentMap<Class<?>, ExtensionLoader<?>> EXTENSION_LOADERS = new ConcurrentHashMap<Class<?>, ExtensionLoader<?>>();

```
这里加载Protocol.class，就以Protocol.class为key，创建的ExtensionLoader为value存储到上述EXTENSION_LOADERS中。

```
    public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {
        if (type == null)
            throw new IllegalArgumentException("Extension type == null");
        if(!type.isInterface()) {
            throw new IllegalArgumentException("Extension type(" + type + ") is not interface!");
        }
        if(!withExtensionAnnotation(type)) {
            throw new IllegalArgumentException("Extension type(" + type + 
                    ") is not extension, because WITHOUT @" + SPI.class.getSimpleName() + " Annotation!");
        }
        
        // 1. 先从EXTENSION_LOADERS中获取
        ExtensionLoader<T> loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        
        // 2. 获取不到时，new一个ExtensionLoader实例，put进EXTENSION_LOADERS
        if (loader == null) {
            EXTENSION_LOADERS.putIfAbsent(type, new ExtensionLoader<T>(type));
            loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        }
        return loader;
    }
    
    // 3. newExtensionLoader实例时，通过SPI获取objectFactory，用来实现IOC功能
    private ExtensionLoader(Class<?> type) {
        this.type = type;
        objectFactory = (type == ExtensionFactory.class ? null : ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension());
    }
    
```
这里没有进行任何的加载实现操作。

##### 3. 调用ExtensionLoader实例的getAdaptiveExtension方法


```
    // 保存缓存的扩展代理实例的Hodler，通过set/get来获取和设置
    private final Holder<Object> cachedAdaptiveInstance = new Holder<Object>();

    public T getAdaptiveExtension() {
        Object instance = cachedAdaptiveInstance.get();
        if (instance == null) {
            if(createAdaptiveInstanceError == null) {
                synchronized (cachedAdaptiveInstance) {
                    instance = cachedAdaptiveInstance.get();
                    if (instance == null) {
                        try {
                        
                            // 创建扩展代理的实例并返回
                            instance = createAdaptiveExtension();
                            
                            // 设置进缓存
                            cachedAdaptiveInstance.set(instance);
                        } catch (Throwable t) {
                            createAdaptiveInstanceError = t;
                            throw new IllegalStateException("fail to create adaptive instance: " + t.toString(), t);
                        }
                    }
                }
            }
            else {
                throw new IllegalStateException("fail to create adaptive instance: " + createAdaptiveInstanceError.toString(), createAdaptiveInstanceError);
            }
        }

        return (T) instance;
    }
```

##### 4. 创建扩展代理的核心方法createAdaptiveExtension()

```
    private T createAdaptiveExtension() {
        try {
            return injectExtension((T) getAdaptiveExtensionClass().newInstance());
        } catch (Exception e) {
            throw new IllegalStateException("Can not create adaptive extenstion " + type + ", cause: " + e.getMessage(), e);
        }
    }
    
```
先来熟悉几个成员变量，事前约定类路径下的 META-INF/services/com.alibaba.dubbo.rpc.Protocol文件下文称配置文件。

```
    // 接口Class,如Procotol.class
    private final Class<?> type;
    
    // @SPI中配置的默认实现类key字符串，在createAdaptiveExtensionClassCode方法中，创建扩展代理类时使用
    private String cachedDefaultName;

```
* getAdaptiveExtensionClass方法

```
    private Class<?> getAdaptiveExtensionClass() {
        // 5. 解析配置文件中key=value格式的接口实现类
        getExtensionClasses();
        
        // 6. 如果该接口有自定义的扩展代理类，即配置文件中有用@Adaptive注解标注的实现类，直接返回。
        if (cachedAdaptiveClass != null) {
            return cachedAdaptiveClass;
        }
        
        // 7. 没有自定义的扩展代理类，创建扩展代理类Code,并编译成Class返回。
        return cachedAdaptiveClass = createAdaptiveExtensionClass();
    }
```
##### 5. 解析配置文件中key=value格式的接口实现类
```

    private Map<String, Class<?>> getExtensionClasses() {
        Map<String, Class<?>> classes = cachedClasses.get();
        if (classes == null) {
            synchronized (cachedClasses) {
                classes = cachedClasses.get();
                if (classes == null) {
                    // 看下文
                    classes = loadExtensionClasses();
                    cachedClasses.set(classes);
                }
            }
        }
        return classes;
	}
	
	// 此方法已经getExtensionClasses方法同步过。
    private Map<String, Class<?>> loadExtensionClasses() {
        final SPI defaultAnnotation = type.getAnnotation(SPI.class);
        if(defaultAnnotation != null) {
            String value = defaultAnnotation.value();
            if(value != null && (value = value.trim()).length() > 0) {
                String[] names = NAME_SEPARATOR.split(value);
                if(names.length > 1) {
                    throw new IllegalStateException("more than 1 default extension name on extension " + type.getName()
                            + ": " + Arrays.toString(names));
                }
                
                // 如果接口上的@SPI注解有指定默认实现类，则赋值cachedDefaultName
                if(names.length == 1) cachedDefaultName = names[0];
            }
        }
        
        Map<String, Class<?>> extensionClasses = new HashMap<String, Class<?>>();
        // 加载相应路径下的配置文件中实现类到相应集合对象中
        loadFile(extensionClasses, DUBBO_INTERNAL_DIRECTORY); // classpath下：META-INF/services/
        loadFile(extensionClasses, DUBBO_DIRECTORY); // classPath下：META-INF/dubbo/
        loadFile(extensionClasses, SERVICES_DIRECTORY); // classPath下：META-INF/dubbo/internal/
        return extensionClasses;
    }
```

* loadFile方法实现较长，详细可自行跟源码，此处我就不贴了。
* 针对上述的加载配置文件中的实现类，ExtensionLoader分四种情况存储到不同集合：
1. 如果该实现类上注解了@Adaptive,则存储到cachedAdaptiveClass成员变量中，只能有一个，在6中作判断使用。
2. 无@Adaptive注解，如果有仅以当前接口为参数的构造方法的实现类，即说明这是一个装饰类，则存储到cachedWrapperClasses成员变量中，如Protocol接口的ProtocolFilterWrapper、ProtocolListenerWrapper为装饰类。
3. 无@Adaptive注解，且不是装饰类，则根据key=value加载存储到cachedClasses成员变量中，其中如果某个实现类无key,如HttpProtocol，则截取以http为key。
4. 在3中解析时，如果某个实现类上有@Activate注解时，注意不是@Adaptive注解，还会存储到cachedActivates成员变量中，其作用稍后再说。
5. 在3中解析时，如果有多个key,则存每个key映射到实现类Class都存储，同时还会以Class字节码为键，以第一个key为值存储到cachedNames成员变量中。
6. 上述涉及到的成员变量源码如下：

```
    // 缓存有@Adaptive注解的Class, 用户自己实现的扩展代理类Class，只能一个
    private volatile Class<?> cachedAdaptiveClass = null;
    
    // 非@Adaptive注解的Class时，缓存有以该接口为参数的构造函数的Class
    private Set<Class<?>> cachedWrapperClasses;
    
    // 缓存以接口为文件名的文件中key=value的实现类
    private final Holder<Map<String, Class<?>>> cachedClasses = new Holder<Map<String,Class<?>>>();

    // 非@Adaptive注解的Class时，以接口为文件名的文件中带有@Activate注解的类，key=name[0], value为@Activate
    private final Map<String, Activate> cachedActivates = new ConcurrentHashMap<String, Activate>();
    
    // 类clazz为key, name[0]为value
    private final ConcurrentMap<Class<?>, String> cachedNames = new ConcurrentHashMap<Class<?>, String>();
```




##### 6. 如果该接口有自定义的扩展代理类，即有@Adaptive注解在实现该接口的类上，直接返回。
* 该cachedAdaptiveClass属性在loadFile方法中解析赋值

##### 7. 没有自定义的扩展代理类，创建扩展代理类Code,并编译成Class返回。

```
    private Class<?> createAdaptiveExtensionClass() {
        // 组装扩展代理类的代码Code
        String code = createAdaptiveExtensionClassCode();
        ClassLoader classLoader = findClassLoader();
        
        // 同样通过SPI方式获取编译器，仅支持Javassist和JdkCompiler。
        // Compiler有自定义的扩展代理类AdaptiveCompiler，即该类有@Adaptive注解
        com.alibaba.dubbo.common.compiler.Compiler compiler = ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.common.compiler.Compiler.class).getAdaptiveExtension();
        
        // 编译返回类字节码Class
        return compiler.compile(code, classLoader);
    }
    
    
```

##### 8. 创建好扩展代理类Class后回到4中的 <injectExtension>方法
* 实例化扩展代理类后，传入injectExtension方法
* 通过set方法注入完依赖后，返回参数instance，即上文中创建的扩展代理类
* 逐级返回扩展代理类，获取扩展代理类结束
```
    private T injectExtension(T instance) {
        try {
            if (objectFactory != null) {
                for (Method method : instance.getClass().getMethods()) {
                    if (method.getName().startsWith("set")
                            && method.getParameterTypes().length == 1
                            && Modifier.isPublic(method.getModifiers())) {
                        Class<?> pt = method.getParameterTypes()[0];
                        try {
                            // 通过setter方法,截取属性名
                            String property = method.getName().length() > 3 ? method.getName().substring(3, 4).toLowerCase() + method.getName().substring(4) : "";
                            
                            // 此处objectFactory也是使用了SPI机制
                            // spring实现如下文，从spring context中根据property获取pt类型的bean.
                            Object object = objectFactory.getExtension(pt, property);
                            if (object != null) {
                                // 获取到要注入的实例后执行set方法注入
                                method.invoke(instance, object);
                            }
                        } catch (Exception e) {
                            logger.error("fail to inject via method " + method.getName()
                                    + " of interface " + type.getName() + ": " + e.getMessage(), e);
                        }
                    }
                }
            }
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
        return instance;
    }
    
    public class SpringExtensionFactory implements ExtensionFactory {
    
        private static final Set<ApplicationContext> contexts = new ConcurrentHashSet<ApplicationContext>();
        
        public static void addApplicationContext(ApplicationContext context) {
            contexts.add(context);
        }
    
        public static void removeApplicationContext(ApplicationContext context) {
            contexts.remove(context);
        }
    
        @SuppressWarnings("unchecked")
        public <T> T getExtension(Class<T> type, String name) {
            for (ApplicationContext context : contexts) {
                if (context.containsBean(name)) {
                    Object bean = context.getBean(name);
                    if (type.isInstance(bean)) {
                        return (T) bean;
                    }
                }
            }
            return null;
        }
    
    }
```

##### 9. 运行时根据传参动态调用实现类
* 回到解析入口
```
 ExtensionLoader extensionLoader = ExtensionLoader.getExtensionLoader(Protocol.class);
 Protocol protocol = extensionLoader.getAdaptiveExtension();  
```
* 此时我们知道了该protocol是一个扩展代理类，生成的类源码可以看[Dubbo-02-SPI-机制](https://note.youdao.com/)
* 调用扩展代理类相应方法时，其实是根据url中传参数，通过下列代码获取具体实现类用反射执行的。

```
    // 根据拿到的协议key从缓存的map中取具体协议实现类对象  
    com.alibaba.dubbo.rpc.Protocol extension = (com.alibaba.dubbo.rpc.Protocol)ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.rpc.Protocol.class).getExtension(extName);  

    // 当前代理调用具体协议实现类对象的方法
    return extension.export(arg0);   

```
* 下面看具体实现，先看几个成员变量

```
    // 缓存实现类的实例，可能是装饰类实例，以获取的name为key
    private final ConcurrentMap<String, Holder<Object>> cachedInstances = new ConcurrentHashMap<String, Holder<Object>>();
    
    // 缓存实现类本身实例
    private static final ConcurrentMap<Class<?>, Object> EXTENSION_INSTANCES = new ConcurrentHashMap<Class<?>, Object>();


```


* ExtensionLoader的getExtension()方法
```
	public T getExtension(String name) {
		if (name == null || name.length() == 0)
		    throw new IllegalArgumentException("Extension name == null");
		if ("true".equals(name)) {
		    return getDefaultExtension();
		}
		Holder<Object> holder = cachedInstances.get(name);
		if (holder == null) {
		    cachedInstances.putIfAbsent(name, new Holder<Object>());
		    holder = cachedInstances.get(name);
		}
		Object instance = holder.get();
		if (instance == null) {
		    synchronized (holder) {
	            instance = holder.get();
	            if (instance == null) {
	                // 创建扩展实现类
	                instance = createExtension(name);
	                holder.set(instance);
	            }
	        }
		}
		return (T) instance;
	}
	
	private T createExtension(String name) {
	    // getExtensionClasses()该方法用来加载配置文件中实现类到cachedClasses中
	    // 通过name获取到实现类Class
        Class<?> clazz = getExtensionClasses().get(name);
        if (clazz == null) {
            throw findException(name);
        }
        try {
            T instance = (T) EXTENSION_INSTANCES.get(clazz);
            if (instance == null) {
                // EXTENSION_INSTANCES中以class为Key，class本身实例为value存入
                EXTENSION_INSTANCES.putIfAbsent(clazz, (T) clazz.newInstance());
                instance = (T) EXTENSION_INSTANCES.get(clazz);
            }
            // 依赖注入
            injectExtension(instance);
            // 此前加载配置文件时，如果有装饰类，则对实现类本身进行装饰类包装
            // 如果装饰类有多个，进行多层包装，之后返回装饰后的实例
            Set<Class<?>> wrapperClasses = cachedWrapperClasses;
            if (wrapperClasses != null && wrapperClasses.size() > 0) {
                for (Class<?> wrapperClass : wrapperClasses) {
                    instance = injectExtension((T) wrapperClass.getConstructor(type).newInstance(instance));
                }
            }
            return instance;
        } catch (Throwable t) {
            throw new IllegalStateException("Extension instance(name: " + name + ", class: " +
                    type + ")  could not be instantiated: " + t.getMessage(), t);
        }
    }
```

