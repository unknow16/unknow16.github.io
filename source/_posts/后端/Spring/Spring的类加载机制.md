---
title: Spring的类加载机制
toc: true
date: 2021-03-25 09:51:37
tags:
categories:
---

## OverridingClassLoader
OverridingClassLoader 是 Spring 自定义的类加载器，构造方法中要指定自己的父类加载器，默认会先自己加载(excludedPackages 或 excludedClasses 例外)，只有加载不到才会委托给双亲加载，这就破坏了 JDK 的双亲委派模式。

它的继承链很简单：OverridingClassLoader -> DecoratingClassLoader -> java的ClassLoader

证明OverridingClassLoader的加载是先自己加载：
```java
import org.junit.Assert;
import org.junit.Test;
import org.springframework.core.OverridingClassLoader;

public class TestSpring {
    @Test
    public void testOverridingClassLoader() throws Exception {
        ClassLoader appClassLoader = Thread.currentThread().getContextClassLoader();

        // 添加到 excludedPackages 或 excludedClasses 的类就不会被代理的 ClassLoader 加载
        // 而会使用 JDK 默认的双亲委派机制
        // 因此 TestBean 不会被 OverridingClassLoader 重新加载，而 ITestBean 会重新加载
        OverridingClassLoader overridingClassLoader = new OverridingClassLoader(appClassLoader);
        overridingClassLoader.excludeClass(TestBean.class.getName());

        Class<?> excludedClazz1 = appClassLoader.loadClass(TestBean.class.getName());
        Class<?> excludedClazz2 = overridingClassLoader.loadClass(TestBean.class.getName());
        Assert.assertTrue("TestBean will exclude from OverridingClassLoader, so no reload",
                excludedClazz1 == excludedClazz2);

        Class<?> nonExcludedClazz1 = appClassLoader.loadClass(ITestBean.class.getName());
        Class<?> nonExcludedClazz2 = overridingClassLoader.loadClass(ITestBean.class.getName());
        Assert.assertFalse("ITestBean will not exclude, so reload again",
                nonExcludedClazz1 == nonExcludedClazz2);
    }


    class TestBean {

    }

    class ITestBean {

    }
}
```
可以看到，ITestBean 被 OverridingClassLoader 重新加载了一次，而 TestBean 添加到了 excludedClasses 中还是使用 JDK 的默认加载器，因此不会被重新加载。

## DecoratingClassLoader
DecoratingClassLoader 很简单，内部维护了两个集合，即excludedPackages和excludedClasses，如果你不想你的类被自定义的类加载器管理，可以把它添加到这两个集合中，这样仍使用 JDK 的默认类加载机制。上面代码中就是excludeClass排除了TestBean。
```java
private final Set<String> excludedPackages = Collections.newSetFromMap(new ConcurrentHashMap<>(8));
private final Set<String> excludedClasses = Collections.newSetFromMap(new ConcurrentHashMap<>(8));

// isExcluded 返回 true 时仍使用 JDK 的默认类加载机制，返回 false 时自定义的类加载器生效
protected boolean isExcluded(String className) {
    if (this.excludedClasses.contains(className)) {
        return true;
    }
    for (String packageName : this.excludedPackages) {
        if (className.startsWith(packageName)) {
            return true;
        }
    }
    return false;
}
```


## 参考资料
> - []()
> - []()
