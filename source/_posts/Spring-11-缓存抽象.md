---
title: Spring-11-缓存抽象
date: 2018-07-11 17:51:31
tags: Spring
---

## 缓存简介
缓存，我的理解是：让数据更接近于使用者。

#### 工作机制
先从缓存中读取数据，如果没有再从慢速设备上读取实际数据，然后将数据再存入缓存。

#### 缓存什么
1. 那些经常读取且不经常修改的数据
1. 那些昂贵（CPU/IO）的且对于相同的请求有相同的计算结果的数据

如CPU--L1/L2--内存--磁盘就是一个典型的例子，CPU需要数据时先从L1/L2中读取，如果没有到内存中找，如果还没有会到磁盘上找。

还有如用过Maven的朋友都应该知道，我们找依赖的时候，先从本机仓库找，再从本地服务器仓库找，最后到远程仓库服务器找。

#### 缓存命中率
即从缓存中读取数据的次数 与 总读取次数的比率，命中率越高越好：

命中率 = 从缓存中读取次数 / (总读取次数[从缓存中读取次数 + 从慢速设备上读取的次数])

Miss率 = 没有从缓存中读取的次数 / (总读取次数[从缓存中读取次数 + 从慢速设备上读取的次数])

这是一个非常重要的监控指标，如果做缓存一定要健康这个指标来看缓存是否工作良好。

#### 缓存策略/移除策略/Eviction policy
即如果缓存满了，从缓存中移除数据的策略,常见的有LFU、LRU、FIFO。

- FIFO（First In First Out）：先进先出算法，即先放入缓存的先被移除
- LRU（Least Recently Used）：最久未使用算法，使用时间距离现在最久的那个被移除
- LFU（Least Frequently Used）：最近最少使用算法，一定时间段内使用次数（频率）最少的那个被移除

#### TTL / Time To Live
存活期，即从缓存中创建时间点开始直到它到期的一个时间段（不管在这个时间段内有没有访问都将过期）

#### TTI / Time To Idle
空闲期，即一个数据多久没被访问将从缓存中移除的时间。

到此，基本了解了缓存的知识，在Java中，我们一般对调用方法进行缓存控制，比如我调用"findUserById(Long id)"，那么我应该在调用这个方法之前先从缓存中查找有没有，如果没有再掉该方法如从数据库加载用户，然后添加到缓存中，下次调用时将会从缓存中获取到数据。

## Spring Cache 抽象
自Spring 3.1起，提供了注解Cache支持，且提供了Cache抽象；在此之前一般通过AOP实现；使用Spring Cache的好处：

1. 提供基本的Cache抽象，方便切换各种底层Cache；
1. 通过注解Cache可以实现类似于事务一样，缓存逻辑透明的应用到我们的业务代码上，且只需要更少的代码就可以完成；
1. 提供事务回滚时也自动回滚缓存；
1. 支持比较复杂的缓存逻辑；

以下类都在spring-context.jar和spring-context-support.jar中。
#### Cache接口
Spring提供的核心Cache接口，提供了缓存操作的读取/写入/移除方法。
```
package org.springframework.cache;

import java.util.concurrent.Callable;

public interface Cache {

    // 缓存的名字
	String getName(); 
    
    // 得到底层使用的缓存，如Ehcache  
	Object getNativeCache(); 

    // 根据key得到一个ValueWrapper，然后调用其get方法获取值 
	ValueWrapper get(Object key); 

    // 根据key获取value,并转换为type
	<T> T get(Object key, Class<T> type);

    // 根据key获取value, 获取不到时，从valueLoader中获取，缓存再返回
    // 相当于 if cached, return; otherwise create, cache and return
	<T> T get(Object key, Callable<T> valueLoader);

    // 往缓存放数据
	void put(Object key, Object value);

    // 不存在时，原子的put，存在时，返回老值
	ValueWrapper putIfAbsent(Object key, Object value); 

    // 从缓存中移除key对应的缓存
	void evict(Object key);

    // 清空缓存 
	void clear(); 

    // ValueWrapper接口
	interface ValueWrapper {

		Object get();
	}

    // ValueRetrievalException异常
	class ValueRetrievalException extends RuntimeException {

		private final Object key;

		public ValueRetrievalException(Object key, Callable<?> loader, Throwable ex) {
			super(String.format("Value for key '%s' could not be loaded using '%s'", key, loader), ex);
			this.key = key;
		}

		public Object getKey() {
			return this.key;
		}
	}

}

```
默认提供了如下实现：

- ConcurrentMapCache：使用java.util.concurrent.ConcurrentHashMap实现的Cache；
- GuavaCache：对Guava com.google.common.cache.Cache进行的Wrapper，需要Google Guava 12.0或更高版本，@since spring 4；
- EhCacheCache：使用Ehcache实现
- JCacheCache：对javax.cache.Cache进行的wrapper，@since spring 3.2；spring4将此类更新到JCache 0.11版本；

#### CacheManager接口
因为我们在应用中并不是使用一个Cache，而是多个，因此Spring还提供了CacheManager抽象，用于缓存的管理。
```
package org.springframework.cache;

import java.util.Collection;

public interface CacheManager {

    // 根据Cache名字获取Cache 
	Cache getCache(String name);

    // 得到所有Cache的名字 
	Collection<String> getCacheNames();

}

```
默认提供的实现： 
- ConcurrentMapCacheManager/ConcurrentMapCacheFactoryBean：管理ConcurrentMapCache；
- GuavaCacheManager；
- EhCacheCacheManager/EhCacheManagerFactoryBean；
- JCacheCacheManager/JCacheManagerFactoryBean；

#### CompositeCacheManager
另外还提供了CompositeCacheManager用于组合CacheManager，即可以从多个CacheManager中轮询得到相应的Cache.

当我们调用cacheManager.getCache(cacheName) 时，会先从第一个cacheManager中查找有没有cacheName的cache，如果没有接着查找第二个，如果最后找不到,且fallbackToNoOpCache=true，那么将返回一个NOP的Cache，否则返回null。

除了GuavaCacheManager之外，其他Cache都支持Spring事务的，即如果事务回滚了，Cache的数据也会移除掉。

Spring不进行Cache的缓存策略的维护，这些都是由底层Cache自己实现，Spring只是提供了一个Wrapper，提供一套对外一致的API。

#### NoOpCacheManager和NoOpCache
NoOpCacheManager是一个没有操作的CacheManager实现，用于失效缓存。

NoOpCache是一个没有操作的Cache实现，用于失效缓存，items会进入缓存，但不会实际存储它。

如CompositeCacheManager中的fallbackToNoOpCache属性就是用来设置，当找不到相应缓存时，是否返回一个NoOpCache，还是返回null.

#### EhCacheManagerFactoryBean...
Spring为一些CacheManager创建了相应的FactoryBean来简化bean创建。

如提供EhCacheManagerFactoryBean来简化ehcache cacheManager的创建，这样注入configLocation，会自动根据路径从classpath下找，然后就可以从spring容器获取cacheManager进行操作了。