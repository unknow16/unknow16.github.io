---
title: Spring Boot-02-日志系统
date: 2018-01-25 21:53:19
tags: Spring Boot
---

### 聊聊日志系统的发展史

##### 规范接口
- commons-logging: 最早的日志接口，原名（Jakarta Commons Logging, JCL)，动态查找实现
- Slf4j: 现在的日志接口，兼容OSGi，编译时静态绑定
---

##### 接口实现
- Log4j: apache开源日志实现
- Logback: 与log4j同一作者，增强功能,三个模块（logback-core，logback-classic，logback-access）
- 其他实现：java.util.logging, Simple Logging(JCL自带简单实现)

##### 常见CP组合
- JCL+Log4j
- SLF4j+Log4j:   
    - slf4j-api-1.5.11.jar 
    - slf4j-log4j12-1.5.11.jar 
    - log4j-1.2.15.jar 
    - log4j.properties
- SLF4j+Logback: 
    - slf4j-api-1.5.11.jar 
    - logback-core-0.9.20.jar 
    - logback-classic-0.9.20.jar 
    - logback.xml 或 logback-test.xml
    
### 日志级别

日志级别 | 打印场景
---|---
DEBUG | 调试日志。目前管理相对宽松，我们暂时没有严格要求。
INFO | 业务日志。我们用来记录业务的主流程的走向。
WARN | 警告日志。一般来说，发生对整个系统没什么影响的异常时，可以打印该级别的日志。
ERROR | 错误日志。级别比较高，如果发生一些异常，并且任何时候都需要打印时使用。

```
使用接口引入：
public static final Logger LOGGER = LoggerFactory.getLogger(MyRealm.class);

使用占位符打印日志
LOGGER.debug("当前用户是:{},传入参数是:{},返回值是:{},用户信息:{}", a,b new Object[]{token, userId, userInfo, authcInfo});
```


### Spring Boot 日志系统
* 默认使用slf4j+logback
* 默认输出级别为ERROR,WRAN,INFO
* 在application.properties中配置debug=true，该属性置为true的时候，核心Logger（包含嵌入式容器、hibernate、spring）会输出更多内容，但是你自己应用的日志并不会输出为DEBUG级别。

### 日志格式

```
2016-04-13 08:23:50.120  INFO 37397 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {4.3.11.Final}
```

* 时间日期 — 精确到毫秒
* 日志级别 — ERROR, WARN, INFO, DEBUG or TRACE
* 进程ID
* 分隔符 — --- 标识实际日志的开始
* 线程名 — 方括号括起来（可能会截断控制台输出）
* Logger名 — 通常使用源代码的类名
* 日志内容

### 文件输出
Spring Boot默认只开启了控制台日志输出，若要输出到日志文件，则需在application.properties如下配置：
* logging.file  设置日志文件，可以是绝对路径或相对路径。如：logging.file=fuyi.log
* logging.path  设置目录，会在该目录下创建spring.log文件，并写入日志内容。

日志文件会在10M大小时被截断，产生新的日志文件，默认为ERROR,WRAN,INFO

### 级别控制
* 配置格式：logging.level.*=LEVEL
* （*）为包名或Logger名
* LEVEL选项TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF

e.g.
```
logging.level.com.fuyi=DEBUG：com.fuyi包下所有class以DEBUG级别输出
logging.level.root=WARN：root日志以WARN级别输出
```

### 自定义日志配置
由于日志服务一般都在ApplicationContext创建前就初始化了，它并不是必须通过Spring的配置文件控制。因此通过系统属性和传统的Spring Boot外部配置文件依然可以很好的支持日志控制和管理。

根据不同的日志系统，你可以按如下规则组织配置文件名，就能被正确加载：

* Logback：logback-spring.xml, logback-spring.groovy, logback.xml, logback.groovy
* Log4j：log4j-spring.properties, log4j-spring.xml, log4j.properties, log4j.xml
* Log4j2：log4j2-spring.xml, log4j2.xml
* JDK (Java Util Logging)：logging.properties

Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml）

如下提供一个自定义的logback-spring.xml日志配置
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

	<!-- 文件输出格式 -->
	<property name="PATTERN" value="%-12(%d{yyyy-MM-dd HH:mm:ss.SSS}) |-%-5level [%thread] %c [%L] -| %msg%n" />
	<!-- test文件路径 -->
	<property name="TEST_FILE_PATH" value="c:/opt/roncoo/logs" />
	<!-- pro文件路径 -->
	<property name="PRO_FILE_PATH" value="/opt/roncoo/logs" />

	<!-- 开发环境 -->
	<springProfile name="dev">
		<appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
			<encoder>
				<pattern>${PATTERN}</pattern>
			</encoder>
		</appender>
		
		<logger name="com.roncoo.education" level="debug"/>

		<root level="info">
			<appender-ref ref="CONSOLE" />
		</root>
	</springProfile>

	<!-- 测试环境 -->
	<springProfile name="test">
		<!-- 每天产生一个文件 -->
		<appender name="TEST-FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
			<!-- 文件路径 -->
			<file>${TEST_FILE_PATH}</file>
			<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
				<!-- 文件名称 -->
				<fileNamePattern>${TEST_FILE_PATH}/info.%d{yyyy-MM-dd}.log</fileNamePattern>
				<!-- 文件最大保存历史数量 -->
				<MaxHistory>100</MaxHistory>
			</rollingPolicy>
			
			<layout class="ch.qos.logback.classic.PatternLayout">
				<pattern>${PATTERN}</pattern>
			</layout>
		</appender>
		
		<root level="info">
			<appender-ref ref="TEST-FILE" />
		</root>
	</springProfile>

	<!-- 生产环境 -->
	<springProfile name="prod">
		<appender name="PROD_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
			<file>${PRO_FILE_PATH}</file>
			<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
				<fileNamePattern>${PRO_FILE_PATH}/warn.%d{yyyy-MM-dd}.log</fileNamePattern>
				<MaxHistory>100</MaxHistory>
			</rollingPolicy>
			<layout class="ch.qos.logback.classic.PatternLayout">
				<pattern>${PATTERN}</pattern>
			</layout>
		</appender>
		
		<root level="warn">
			<appender-ref ref="PROD_FILE" />
		</root>
	</springProfile>
</configuration>

```


