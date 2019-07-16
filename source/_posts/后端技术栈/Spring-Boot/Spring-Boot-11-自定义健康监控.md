---
title: Spring Boot-11-自定义健康监控
date: 2018-07-09 20:12:10
tags: Spring Boot
---

Health 信息是从 ApplicationContext 中所有的 HealthIndicator接口 的实现类 Bean 中收集的， Spring Boot 内置了一些 HealthIndicator。

## 内置的 HealthIndicator
在spring-boot-actuator的jar中

Name|	Description
---|---
CassandraHealthIndicator|	Checks that a Cassandra database is up.
DiskSpaceHealthIndicator|	Checks for low disk space.
DataSourceHealthIndicator|	Checks that a connection to DataSource can be obtained.
ElasticsearchHealthIndicator|	Checks that an Elasticsearch cluster is up.
JmsHealthIndicator|	Checks that a JMS broker is up.
MailHealthIndicator|	Checks that a mail server is up.
MongoHealthIndicator|	Checks that a Mongo database is up.
RabbitHealthIndicator|	Checks that a Rabbit server is up.
RedisHealthIndicator|	Checks that a Redis server is up.
SolrHealthIndicator|	Checks that a Solr server is up.

#### DataSourceHealthIndicator健康检测原理
1. 通过jdbcTemplate根据java.sql.Connection.getMetaData().getDatabaseProductName()获取数据库产品名，如获取到MySQL或Oracle
2. 根据不同数据库，构造不同校验查询sql,如下
```
// MySQL的ping
/* ping */ SELECT 1

// Oracle
SELECT 'Hello' from DUAL
```
3. 通过jdbcTemplate执行上一步构造的校验sql
4. 能正常返回则说明健康。

#### RedisHealthIndicator健康检测原理
1. 构造RedisConnection链接对象
2. 如果是cluster，则调用RedisClusterConnection的clusterGetClusterInfo()方法，获取redis服务器端集群信息，并展示。
2. 非集群时，通过RedisConnection的info()方法获取redis服务器端版本信息，返回并展示。

> 其他健康检测原理类似。。。

## 自定义 HealthIndicator 监控检测
一般情况下，Spring Boot 提供的健康监控无法满足我们复杂的业务场景，此时，我们就需要定制自己的 HealthIndicator， 扩展自己的业务监控。有两种方式

1. 实现 HealthIndicator 接口
1. 继承AbstractHealthIndicator

#### 实现 HealthIndicator 接口
我们，实现 HealthIndicator 接口创建一个简单的检测器类。它的作用很简单，只是进行服务状态监测。此时，通过重写 health() 方法来实现健康检查。
```
package com.fuyi.actuator.indicator;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;

@Component
public class CustHealthIndicator implements HealthIndicator {

	@Override
	public Health health() {
		int errorCode = check();
		if(errorCode != 0) {
			return Health.down()
						.withDetail("status", errorCode)
						.withDetail("message", "服务器故障")
						.build();
		}
		return Health.up().build();
	}

	/*
	 * 检测状态
	 */
	private int check() {
		return HttpStatus.NOT_FOUND.value();
	}

}
```
检测结果如下：

```
{
    "status":"DOWN",
    "cust":{
        "status":404,
        "message":"服务器故障"
    },
    "diskSpace":{
        "status":"UP",
        "total":315118665728,
        "free":293298757632,
        "threshold":10485760
    }
}
```

#### 继承AbstractHealthIndicator
AbstractHealthIndicator 实现 HealthIndicator 接口，并重写了 health() 方法来实现健康检查。因此，我们只需要重写 doHealthCheck 方法即可。

一般情况下，我们不会直接实现 HealthIndicator 接口，而是继承 AbstractHealthIndicator 抽象类。因为，我们只需要重写 doHealthCheck 方法，并在这个方法中我们关注于具体的健康检测的业务逻辑服务。

* 自定义磁盘健康指示器，以G为单位
```
package com.fuyi.actuator.indicator;

import java.io.IOException;
import java.nio.file.FileStore;
import java.nio.file.Files;
import java.nio.file.Paths;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.actuate.health.AbstractHealthIndicator;
import org.springframework.boot.actuate.health.Health.Builder;
import org.springframework.stereotype.Component;

@Component
public class CustDiskSpaceHealthIndicator extends AbstractHealthIndicator {
	
	private final FileStore fileStore;
    private final long thresholdBytes;

    @Autowired
    public CustDiskSpaceHealthIndicator(
    		@Value("${health.filestore.path:.}") String path, 
    		@Value("${health.filestore.threshold.bytes:10485760}") long thresholdBytes)  throws IOException {
    	this.fileStore = Files.getFileStore(Paths.get(path));
    	this.thresholdBytes = thresholdBytes;
    }
    
	@Override
	protected void doHealthCheck(Builder builder) throws Exception {
		long diskFreeInBytes = fileStore.getUnallocatedSpace();
		if(diskFreeInBytes >= this.thresholdBytes) {
			builder.up();
		}
		
		long totalSpaceInBytes = fileStore.getTotalSpace();
		builder.withDetail("total", totalSpaceInBytes/1024/1024/1024 + " G");
		builder.withDetail("free", diskFreeInBytes/1024/1024/1024 + " G");
		builder.withDetail("threshold", this.thresholdBytes/1024/1024 + " M");
	}

}


```
检测结果如下：

```
{
    "status":"DOWN",
    "custDiskSpace":{
        "status":"UP",
        "total":"293 G",
        "free":"273 G",
        "threshold":"10 M"
    },
    "cust":{
        "status":404,
        "message":"服务器故障"
    },
    "diskSpace":{
        "status":"UP",
        "total":315118665728,
        "free":293302198272,
        "threshold":10485760
    }
}
```
