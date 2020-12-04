---
title: Mybatis二级缓存
date: 2018-02-26 23:56:32
tags: Mybatis
---

### 开启二级缓存
默认关闭
```
	<!-- 在mapper映射文件中
	    开启二级缓存 -->
	<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true" />
```


### 工作模式
1. 一个新的sqlSession会创建一个新的Executor来执行实际的操作，但是开启二级缓存后，创建Executor时，会用CachingExecutor进行装饰。

2. 在使用CachingExecutor执行操作时，会先查询Application级别的二级缓存，如果有直接返回，没有再交给真正的Executor执行查询数据库。

3. 之后再放回到缓存中。

### 缓存界限划分
* Mapper级别的Cache缓存对象，即每个Mapper有一个Cache对象。

在Mapper文件中使用<cache>标签配置

* 多个Mapper共用一个Cache缓存对象

使用<cache-ref namespace="">节点，来指定你的这个Mapper使用到了哪一个Mapper的Cache缓存。

### 二级缓存实现选择
* 实现Cache接口，配置<cache type="">节点为实现类即可
* 自身实现提供了一系列的Cache接口装饰器。
    * LRU:最少使用，缓存容量满了，标记为最少使用的缓存清除掉
    * FIFO:先进先出，缓存满时，最先进入缓存的被清除
    * Scheduled: 指定时间间隔清空
* 集成memecached
