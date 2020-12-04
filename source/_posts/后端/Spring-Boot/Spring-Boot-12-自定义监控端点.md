---
title: Spring Boot-12-自定义监控端点
date: 2018-07-09 20:12:20
tags: Spring Boot
---

Spring Boot 提供的端点不能满足我们的业务需求时，我们可以自定义一个端点。

本文，我将演示一个简单的自定义端点，用来查看服务器的当前日期和时间，它将返回两个参数，分别是日期和时间对象。

#### 1. 继承 AbstractEndpoint 抽象类
首先，我们需要继承 AbstractEndpoint 抽象类。因为它是 Endpoint 接口的抽象实现，此外，我们还需要重写 invoke 方法。

值得注意的是，通过设置 @ConfigurationProperties(prefix = “endpoints.serverdatetime”)，我们就可以在 application.properties 中通过 endpoints.servertime 配置我们的端点。
```
package com.fuyi.actuator.endpoint;

import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

import org.springframework.boot.actuate.endpoint.AbstractEndpoint;
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "endpoints.servertime")
public class ServerTimeEndpoint extends AbstractEndpoint<Map<String, Object>> {

    // servertime作为访问路径
    // 可在application.yml中,通过endpoints.servertime.id修改
	public ServerTimeEndpoint() {
		super("servertime", false);
	}

	@Override
	public Map<String, Object> invoke() {
		Map<String, Object> result = new HashMap<String, Object>();
        
		LocalDateTime now = LocalDateTime.now();
        result.put("server_time", now.toLocalTime());
        result.put("server_date", now.toLocalDate());
        return result;
	}

}

```
上面的代码，我解释下，两个重要的细节。

1. 构造方法 ServerTimeEndpoint，两个参数分别表示端点 ID 和是否端点默认是敏感的。我这边设置端点 ID 是 servertime，它默认不是敏感的。
2. 我们需要通过重写 invoke 方法，返回我们要监控的内容。这里我定义了一个 Map，它将返回两个参数，分别是日期和时间对象。

#### 2. 创建端点配置类
创建端点配置类，并注册我们的端点 ServerTimeEndpoint。
```
package com.fuyi.actuator.endpoint;

import java.util.Map;

import org.springframework.boot.actuate.endpoint.Endpoint;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class EndpointConfig {

	@Bean
	public static Endpoint<Map<String, Object>> servertime() {
		return new ServerTimeEndpoint();
	}
}

```

#### 3.启动应用
访问 http://localhost:8080/servertime ，此时，服务器会返回如下信息：
```
{
    "server_time":{
        "hour":17,
        "minute":51,
        "second":59,
        "nano":546000000
    },
    "server_date":{
        "year":2018,
        "month":"JULY",
        "monthValue":7,
        "dayOfMonth":9,
        "dayOfWeek":"MONDAY",
        "era":"CE",
        "dayOfYear":190,
        "leapYear":false,
        "chronology":{
            "id":"ISO",
            "calendarType":"iso8601"
        }
    }
}
```
