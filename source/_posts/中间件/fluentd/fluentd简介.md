---
title: fluentd简介
toc: true
date: 2019-11-20 10:22:03
tags:
categories:
---





Fluentd是用于统一日志记录层的开源数据收集器。 Fluentd允许您统一数据收集和使用，以更好地使用和理解数据。



## 一个Fluentd 事件的生命周期

下面的文章使用示例描述Fluentd如何处理事件的全局概述。它涵盖了完整的周期，包括设置，输入，过滤器，匹配和标签。



#### 基本设定

配置文件是将所有事物连接在一起的基础部分，因为它允许定义Fluentd将拥有哪些输入或监听器，并设置通用的匹配规则以将事件数据路由到特定的输出。

我们将使用in_http和out_stdout插件作为示例来描述事件周期。以下是配置文件中用于指定http输入的基本定义，简而言之：我们将监听HTTP请求：

```
<source>
  @type http
  port 8888
  bind 0.0.0.0
</source>
```

该定义指定HTTP服务器将侦听TCP端口8888。现在让我们定义一个Matching规则和一个所需的输出，该输出将仅将到达每个请求的数据打印到标准输出：

```
<match test.cycle>
  @type stdout
</match>
```

Match设置了一个规则，其中每个带有Tag等于test.cycle的传入事件都将匹配并使用称为stdout的Output插件类型。此时，我们有一个输入类型，一个匹配项和一个输出。让我们使用Curl测试设置：

```
$ curl -i -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.cycle
HTTP/1.1 200 OK
Content-type: text/plain
Connection: Keep-Alive
Content-length: 0
```

在Fluentd服务器端，输出应如下所示：

```
$ bin/fluentd -c in_http.conf
2015-01-19 12:37:41 -0600 [info]: reading config file path="in_http.conf"
2015-01-19 12:37:41 -0600 [info]: starting fluentd-0.12.3
2015-01-19 12:37:41 -0600 [info]: using configuration file: <ROOT>
  <source>
    @type http
    bind 0.0.0.0
    port 8888
  </source>
  <match test.cycle>
    @type stdout
  </match>
</ROOT>
2015-01-19 12:37:41 -0600 [info]: adding match pattern="test.cycle" type="stdout"
2015-01-19 12:37:41 -0600 [info]: adding source type="http"
2015-01-19 12:39:57 -0600 test.cycle: {"action":"login","user":2}
```



## 参考资料
> - []()
> - []()
