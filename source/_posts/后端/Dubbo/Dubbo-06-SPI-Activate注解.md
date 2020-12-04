---
title: Dubbo-06-SPI-@Activate注解
date: 2018-05-16 13:11:19
tags: Dubbo
---

### 扩展点自动激活
Dubbo配置模块中，扩展点均有对应配置属性或标签，通过配置指定使用哪个扩展实现。比如采用协议：

```
<dubbo:protocol name="xxx" />
```
对于集合类扩展点，比如：Filter、InvokerListener、ExportListener、TelnetHandler、StatusChecker等，可以同时加载多个实现，此时，可以用自动激活来简化配置，如xx过滤器自动激活

```
@Activate // 无条件自动激活
public class XxxFilter implements Filter {
    // ...
}

// 当配置了xxx参数，并且参数为有效值时激活
// 比如配了cache="lru"，自动激活CacheFilter。
@Activate("xxx")
public class XxxFilter implements Filter {
    // ...
}

// 只对提供方激活，group可选"provider"或"consumer"
@Activate(group = "provider", value = "xxx") 
public class XxxFilter implements Filter {
    // ...
}
```
