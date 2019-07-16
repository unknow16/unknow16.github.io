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

如下提供一个自定义的logback-spring.xml日志配置，会自动根据spring.profiles.active的值自动获取相应的配置
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- 文件输出格式 -->
    <property name="log_pattern" value="%-12(%d{yyyy-MM-dd HH:mm:ss.SSS}) |-%-5level [%thread] %c [%L] -| %msg%n" />
    <!-- test文件路径 -->
    <property name="test_file_path" value="F:/ws_idea/fuyi/logs" />
    <!-- pro文件路径 -->
    <property name="prod_file_path" value="F:/ws_idea/fuyi/logs" />

    <!-- 开发环境 -->
    <springProfile name="dev">

        <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>${log_pattern}</pattern>
            </encoder>
        </appender>

        <logger name="com.fuyi" level="debug"/>

        <!-- show parameters for hibernate sql 专为 Hibernate 定制 -->
        <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE" />
        <logger name="org.hibernate.type.descriptor.sql.BasicExtractor" level="DEBUG" />
        <logger name="org.hibernate.SQL" level="DEBUG" />
        <logger name="org.hibernate.engine.QueryParameters" level="DEBUG" />
        <logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG" />

        <!--myibatis log configure-->
        <logger name="com.apache.ibatis" level="TRACE"/>
        <logger name="java.sql.Connection" level="DEBUG"/>
        <logger name="java.sql.Statement" level="DEBUG"/>
        <logger name="java.sql.PreparedStatement" level="DEBUG"/>

        <root level="error">
            <appender-ref ref="consoleAppender" />
        </root>
    </springProfile>

    <!-- 测试环境 -->
    <springProfile name="test">

        <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>${log_pattern}</pattern>
            </encoder>
        </appender>

        <!-- 每天产生一个文件 -->
        <appender name="testFileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <!-- 文件路径 -->
            <!--<file>${test_file_path}</file>-->
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <!-- 文件名称 -->
                <fileNamePattern>${test_file_path}/info.%d{yyyy-MM-dd}.log</fileNamePattern>
                <!-- 文件最大保存历史数量 -->
                <MaxHistory>100</MaxHistory>
            </rollingPolicy>

            <layout class="ch.qos.logback.classic.PatternLayout">
                <pattern>${log_pattern}</pattern>
            </layout>
        </appender>

        <logger name="com.fuyi" level="debug"/>

        <!-- show parameters for hibernate sql 专为 Hibernate 定制 -->
        <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE" />
        <logger name="org.hibernate.type.descriptor.sql.BasicExtractor" level="DEBUG" />
        <logger name="org.hibernate.SQL" level="DEBUG" />
        <logger name="org.hibernate.engine.QueryParameters" level="DEBUG" />
        <logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG" />

        <!--myibatis log configure-->
        <logger name="com.apache.ibatis" level="TRACE"/>
        <logger name="java.sql.Connection" level="DEBUG"/>
        <logger name="java.sql.Statement" level="DEBUG"/>
        <logger name="java.sql.PreparedStatement" level="DEBUG"/>

        <root level="error">
            <appender-ref ref="testFileAppender" />
            <appender-ref ref="consoleAppender" />
        </root>
    </springProfile>

    <!-- 生产环境 -->
    <springProfile name="prod">
        <appender name="prodFileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <!--<file>${prod_file_path}</file>-->
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${prod_file_path}/warn.%d{yyyy-MM-dd}.log</fileNamePattern>
                <MaxHistory>100</MaxHistory>
            </rollingPolicy>
            <layout class="ch.qos.logback.classic.PatternLayout">
                <pattern>${log_pattern}</pattern>
            </layout>
        </appender>

        <root level="warn">
            <appender-ref ref="prodFileAppender" />
        </root>
    </springProfile>
</configuration>

```


## logback的配置介绍
1. Logger: 作为日志的记录器，把它关联到应用的对应的context上后，主要用于存放日志对象，也可以定义日志类型、级别。
1. Appender: 主要用于指定日志输出的目的地，目的地可以是控制台、文件、远程套接字服务器、 MySQL、PostreSQL、 Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等。
1. Layout: 负责把事件转换成字符串，格式化的日志信息的输出。
1. Logger Context: 各个logger 都被关联到一个 LoggerContext，LoggerContext负责制造logger，也负责以树结构排列各logger。其他所有logger也通过org.slf4j.LoggerFactory 类的静态方法getLogger取得。 getLogger方法以 logger名称为参数。用同一名字调用LoggerFactory.getLogger 方法所得到的永远都是同一个logger对象的引用。

- 有效级别及级别的继承

Logger 可以配置级别，即配置运行时打印出该Logger的哪种级别的日志。级别包括：TRACE、DEBUG、INFO、WARN 和 ERROR，定义于ch.qos.logback.classic.Level类。如果 Logger没有被分配级别，那么它将从有被分配级别的最近的祖先那里继承级别。root logger 默认级别是 DEBUG。

打印方法决定记录请求的级别。例如，如果 L 是一个 logger 实例，那么，语句 L.info("..")是一条级别为 INFO的记录语句。记录请求的级别在高于或等于其 logger 的有效级别时被称为被启用，否则，称为被禁用。记录请求级别为 p，其 logger的有效级别为 q，只有则当 p>=q时，该请求才会被执行。
该规则是 logback 的核心。级别排序为： TRACE < DEBUG < INFO < WARN < ERROR

## logback的配置文件介绍
1. 根节点<configuration>，包含下面三个属性：
- scan: 当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
- scanPeriod: 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
- debug: 当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

```
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
　　  <!--其他配置省略--> 
</configuration>
```
2. 子节点<contextName>：用来设置上下文名称，每个logger都关联到logger上下文，默认上下文名称为default。但可以使用<contextName>设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改。

```
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
     <contextName>myAppName</contextName> 
　　  <!--其他配置省略-->
</configuration>
```
3. 子节点<property>：用来定义变量值，它有两个属性name和value，通过<property>定义的值会被插入到logger上下文中，可以使“${}”来使用变量。name: 变量的名称,value: 的值时变量定义的值

```
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
　　　<property name="APP_Name" value="myAppName" /> 
　　　<contextName>${APP_Name}</contextName> 
　　　<!--其他配置省略--> 
</configuration>
```
4. 子节点<timestamp>：获取时间戳字符串，他有两个属性key和datePattern
　　　　key: 标识此<timestamp> 的名字；
　　　　datePattern: 设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循java.txt.SimpleDateFormat的格式。
　　例如：

```
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
    <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"/> 
　　<contextName>${bySecond}</contextName> 
　　<!-- 其他配置省略--> 
</configuration>
```
5. 子节点<appender>：负责写日志的组件，它有两个必要属性name和class。name指定appender名称，class指定appender的全限定名，class实现类有如下几个，用户也可自己实现，后面详细介绍
- ConsoleAppender 把日志输出到控制台
- FileAppender：把日志添加到文件
- RollingFileAppender：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。
- SocketAppender、SMTPAppender、DBAppender、SyslogAppender、SiftingAppender，并不常用

6. 子节点<logger>：用来设置某一个包或具体的某一个类的日志打印级别、以及指定<appender>。仅有一个name属性，一个可选的level和一个可选的addtivity属性。可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个logger

- name: 用来指定受此loger约束的某一个包或者具体的某一个类。
- level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。 如果未设置此属性，那么当前loger将会继承上级的级别。
- addtivity: 是否向上级loger传递打印信息。默认是true。同<logger>一样，可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个loger。

7. 子节点<root>:它也是<loger>元素，但是它是根loger,是所有<loger>的上级。只有一个level属性，因为name已经被命名为"root",且已经是最上级了。
- level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，不能设置为INHERITED或者同义词NULL。 默认是DEBUG。

#### ConsoleAppender 
把日志输出到控制台，有以下子节点：
- <encoder>：对日志进行格式化。（具体参数稍后讲解 ）
- <target>：字符串System.out(默认)或者System.err（区别不多说了）
```
<configuration> 
　　　<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> 
　　　　　 <encoder> 
　　　　　　　　　<pattern>%-4relative [%thread] %-5level %logger{35} - %msg %n</pattern> 
　　　　　 </encoder> 
　　　</appender> 

　　　<root level="DEBUG"> 
　　　　　　<appender-ref ref="STDOUT" /> 
　　　</root> 
</configuration>
```
上述配置表示把>=DEBUG级别的日志都输出到控制台

#### RollingFileAppender
滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：
1. <file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
1. <append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。	
1. <rollingPolicy>:当发生滚动时，决定RollingFileAppender的行为，涉及文件移动和重命名。属性class定义具体的滚动策略类

#### 滚动策略类

1. ch.qos.logback.core.rolling.TimeBasedRollingPolicy

最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责触发滚动。有以下子节点：
- <fileNamePattern>：必要节点，包含文件名及“%d”转换符，“%d”可以包含一个java.text.SimpleDateFormat指定的时间格式，如：%d{yyyy-MM}。如果直接使用 %d，默认格式是 yyyy-MM-dd。RollingFileAppender的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。
- <maxHistory>:可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且<maxHistory>是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除。

2. ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy

查看当前活动文件的大小，如果超过指定大小会告知RollingFileAppender 触发当前活动文件滚动。只有一个节点:
- <maxFileSize>:这是活动文件的大小，默认值是10MB。


3. ch.qos.logback.core.rolling.FixedWindowRollingPolicy

根据固定窗口算法重命名文件的滚动策略。有以下子节点：
- <minIndex>:窗口索引最小值
- <maxIndex>:窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。
- <fileNamePattern>:必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip