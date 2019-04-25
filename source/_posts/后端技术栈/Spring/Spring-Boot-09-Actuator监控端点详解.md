---
title: Spring Boot-09-Actuator监控端点详解
date: 2018-07-09 18:39:21
tags: Spring Boot
---

### 端点分类
如果根据端点的作用来说，我们可以原生端点分为三大类：

- 应用配置类：获取应用程序中加载的应用配置、环境变量、自动化配置报告等与Spring Boot应用密切相关的配置类信息。
- 度量指标类：获取应用程序运行过程中用于监控的度量指标，比如：内存信息、线程池信息、HTTP请求统计等。
- 操作控制类：提供了对应用的关闭等操作类功能。

### 应用配置类
由于Spring Boot为了改善传统Spring应用繁杂的配置内容，采用了包扫描和自动化配置的机制来加载原本集中于xml文件中的各项内容。虽然这样的做法，让我们的代码变得非常简洁，但是整个应用的实例创建和依赖关系等信息都被离散到了各个配置类的注解上，这使得我们分析整个应用中资源和实例的各种关系变得非常的困难。而这类端点就可以帮助我们轻松的获取一系列关于Spring 应用配置内容的详细报告，比如：自动化配置的报告、Bean创建的报告、环境属性的报告等。
- /autoconfig：该端点用来获取应用的自动化配置报告，其中包括所有自动化配置的候选项。同时还列出了每个候选项自动化配置的各个先决条件是否满足。所以，该端点可以帮助我们方便的找到一些自动化配置为什么没有生效的具体原因。该报告内容将自动化配置内容分为两部分：
 `positiveMatches`中返回的是条件匹配成功的自动化配置，`negativeMatches`中返回的是条件匹配不成功的自动化配置。

- /beans：该端点用来获取应用上下文中创建的所有Bean。在每个bean中都包含了下面这几个信息：
  - bean：Bean的名称
  - scope：Bean的作用域
  - type：Bean的Java类型
  - reource：class文件的具体路径
  - dependencies：依赖的Bean名称

- /configprops：该端点用来获取应用中配置的属性信息报告。在返回的配置信息中，`prefix`属性代表了属性的配置前缀，`properties`代表了各个属性的名称和值。比如我们要关闭该端点，就可以通过使用endpoints.configprops.enabled=false来完成设置。

- /env：该端点与`/configprops`不同，它用来获取应用所有可用的环境属性报告。包括：环境变量、JVM属性、应用的配置配置、命令行中的参数。它可以帮助我们方便地看到当前应用可以加载的配置信息，并配合`@ConfigurationProperties`注解将它们引入到我们的应用程序中来进行使用。另外，为了配置属性的安全，对于一些类似密码等敏感信息，该端点都会进行隐私保护，但是我们需要让属性名中包含：password、secret、key这些关键词，这样该端点在返回它们的时候会使用`*`来替代实际的属性值。

- /mappings：该端点用来返回所有Spring MVC的控制器映射关系报告。该报告的信息与我们在启用Spring MVC的Web应用时输出的日志信息类似，其中`bean`属性标识了该映射关系的请求处理器，`method`属性标识了该映射关系的具体处理类和处理函数。

- /info：该端点用来返回一些应用自定义的信息。默认情况下，它只会返回一个空的json内容。我们可以在`application.properties`配置文件中通过`info`前缀来设置一些属性。

```
info:
  appName: actuator
  appVersion: 1.0.0
  author: fuyi
  date: 2018-07-09
```

- /loggers:	显示当前应用程序所有包和类的日志级别

### 度量指标类
上面我们所介绍的应用配置类端点所提供的信息报告在应用启动的时候都已经基本确定了其返回内容，可以说是一个静态报告。而度量指标类端点提供的报告内容则是动态变化的，这些端点提供了应用程序在运行过程中的一些快照信息，比如：内存使用情况、HTTP请求统计、外部资源指标等。这些端点对于我们构建微服务架构中的监控系统非常有帮助，由于Spring Boot应用自身实现了这些端点，所以我们可以很方便地利用它们来收集我们想要的信息，以制定出各种自动化策略。下面，我们就来分别看看这些强大的端点功能。

- /metrics：该端点用来返回当前应用的各类重要度量指标，比如：内存信息、线程信息、垃圾回收信息等。它主要报下下面这些重要度量信息：
  - 系统信息：包括处理器数量`processors`、运行时间`uptime`和`instance.uptime`、系统平均负载`systemload.average`。
  - `mem.*`：内存概要信息，包括分配给应用的总内存数量以及当前空闲的内存数量。这些信息来自`java.lang.Runtime`。
  - `heap.*`：堆内存使用情况。这些信息来自`java.lang.management.MemoryMXBean`接口中`getHeapMemoryUsage`方法获取的`java.lang.management.MemoryUsage`。
  - `nonheap.*`：非堆内存使用情况。这些信息来自`java.lang.management.MemoryMXBean`接口中`getNonHeapMemoryUsage`方法获取的`java.lang.management.MemoryUsage`。
  - `threads.*`：线程使用情况，包括线程数、守护线程数（daemon）、线程峰值（peak）等，这些数据均来自`java.lang.management.ThreadMXBean`。
  - `classes.*`：应用加载和卸载的类统计。这些数据均来自`java.lang.management.ClassLoadingMXBean`。
  - `gc.*`：垃圾收集器的详细信息，包括垃圾回收次数`gc.ps_scavenge.count`、垃圾回收消耗时间`gc.ps_scavenge.time`、标记-清除算法的次数`gc.ps_marksweep.count`、标记-清除算法的消耗时间`gc.ps_marksweep.time`。这些数据均来自`java.lang.management.GarbageCollectorMXBean`。
  - `httpsessions.*`：Tomcat容器的会话使用情况。包括最大会话数`httpsessions.max`和活跃会话数`httpsessions.active`。该度量指标信息仅在引入了嵌入式Tomcat作为应用容器的时候才会提供。
  - `gauge.*`：HTTP请求的性能指标之一，它主要用来反映一个绝对数值。比如`gauge.response.hello: 5`，它表示上一次`hello`请求的延迟时间为5毫秒。
  - `counter.*`：HTTP请求的性能指标之一，它主要作为计数器来使用，记录了增加量和减少量。比如：`counter.status.200.hello: 11`，它代表了`hello`请求返回`200`状态的次数为11。

- /health：该端点在一开始的示例中我们已经使用过了，它用来获取应用的各类健康指标信息。在`spring-boot-starter-actuator`模块中自带实现了一些常用资源的健康指标检测器。这些检测器都通过`HealthIndicator`接口实现，并且会根据依赖关系的引入实现自动化装配，比如用于检测磁盘的`DiskSpaceHealthIndicator`、检测DataSource连接是否可用的`DataSourceHealthIndicator`等。有时候，我们可能还会用到一些Spring Boot的Starter POMs中还没有封装的产品来进行开发，比如：当使用RocketMQ作为消息代理时，由于没有自动化配置的检测器，所以我们需要自己来实现一个用来采集健康信息的检测器。比如，我们可以在Spring Boot的应用中，为org.springframework.boot.actuate.health.HealthIndicator接口实现一个对RocketMQ的检测器类.

- /dump：该端点用来暴露程序运行中的线程信息。它使用`java.lang.management.ThreadMXBean`的`dumpAllThreads`方法来返回所有含有同步信息的活动线程详情。

- /heapdump: 返回一个gzip压缩 hprof 堆转储文件.

- /trace：该端点用来返回基本的HTTP跟踪信息。默认情况下，跟踪信息的存储采用`org.springframework.boot.actuate.trace.InMemoryTraceRepository`实现的内存方式，始终保留最近的100条请求记录.

- /auditevents:	显示当前应用程序授权时间的统计信息

### 操作控制类
实际上，由于之前介绍的所有端点都是用来反映应用自身的属性或是运行中的状态，相对于操作控制类端点没有那么敏感，所以他们默认都是启用的。而操作控制类端点拥有更强大的控制能力，如果要使用它们的话，需要通过属性来配置开启。

在原生端点中，只提供了一个用来关闭应用的端点：`/shutdown`。我们可以通过如下配置`endpoints.shutdown.enabled=true`开启它：

在配置了上述属性之后，只需要POST访问该应用的`/shutdown`端点就能实现关闭该应用的远程操作。由于开放关闭应用的操作本身是一件非常危险的事，所以真正在线上使用的时候，我们需要对其加入一定的保护机制，比如：定制Actuator的端点路径、整合Spring Security进行安全校验等。