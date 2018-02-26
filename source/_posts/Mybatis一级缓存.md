---
title: Mybatis一级缓存
date: 2018-02-26 23:56:24
tags: Mybatis
---

### 定义
一级缓存中SqlSession中的缓存，关闭后，将不可用

当创建一个新的SqlSession后，就会创建一个新的Executor，来负责对数据库的各种操作和维护缓存。

BaseExecutor中持有一个Cache接口的实现类PerpetualCache
 
### Cache接口
Mybatis定义的一个接口作为其实现类的SPI(Service Provider Infterface)

所有的Cache的实现类都应实现该接口。如：PerpetualCache类

### CacheKey类
由statementId + rowBound + 传递给JDBC Statement的SQL + 传递给JDBC Statement的SQL的参数组成
 
### PerpetualCache一级缓存实现
内部通过一个HashMap<K, V>来实现的，没有任何容量和大小的限制
 
### 一级缓存生命周期
1. sqlSession会话结束后，其几个内部的Executor、PerpetualCache都会被释放掉。
 
2. sqlSession.close()
3. sqlSession.clearCache(),清空缓存
4. sqlSession期间执行了update操作（增，删，改）都会清空缓存。

### 工作流程
1. 根据statementId, params, rowBounds来创建一个key值（为CacheKey实例），根据key获取缓存。
2. 命中则返回，否则查数据库，然后再放入缓存。

### 怎么判断两次查询是否相同

以下条件都相同则认为是相同查询
1. StatementId
2. 获取结果集的范围，即分页数据，由rowBound.offset和rowBound.limit指定。
3. 传给JDBC PreparedStatement的sql, 由boundSql.getSql()获取
4. 传给JDBC PreparedStatement的sql中？占位符参数一致。