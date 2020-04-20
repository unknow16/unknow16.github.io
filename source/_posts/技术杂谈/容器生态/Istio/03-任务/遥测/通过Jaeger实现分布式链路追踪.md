---
title: 分布式链路追踪
toc: true
date: 2019-11-28 11:15:01
tags:
categories:
---









默认istio-v1.0安装了jaeger作为分布式链路追踪实现。

## 访问UI

有两种方式

- 端中转发，需要iptable支持

```
$ kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 16686:16686 &
```

访问地址	http://ip :16686

- jaeger-query服务的NodePort方式暴露



## 理解发生什么

尽管Istio代理能够自动发送span，但它们仍需要一些提示才能将整个轨迹绑定在一起。应用程序需要传播适当的HTTP请求头，以便当代理发送span信息时，可以将span正确关联到单个trace中。

为此，应用程序需要收集以下标头并将其从传入请求传播到任何传出请求：

- `x-request-id`
- `x-b3-traceid`
- `x-b3-spanid`
- `x-b3-parentspanid`
- `x-b3-sampled`
- `x-b3-flags`
- `x-ot-span-context`



## Trace采样率配置

使用演示配置时（如本任务所述），Istio会捕获所有请求的跟踪。例如，当使用上面的Bookinfo示例应用程序时，每次访问/ productpage时，您都会在Jaeger仪表板中看到相应的跟踪。该采样率适用于测试或低流量的网格，这就是为什么将其用作演示安装的默认值。

在其他配置中，Istio默认为每100个请求中的1个生成跟踪范围（采样率为1％）。

您可以通过以下两种方式之一控制跟踪采样百分比：

- 在网格设置期间，使用Helm选项pilot.traceSampling设置跟踪采样的百分比。有关设置选项的详细信息，请参见Helm安装文档。
- 在运行的网格中，编辑istio-pilot部署，并通过以下步骤更改环境变量：

```
## 编辑配置
$ kubectl -n istio-system edit deploy istio-pilot
```

找到PILOT_TRACE_SAMPLING 环境变量，改成你期望的值，有效值为0.0到100.0，精确到两位小数。

## 参考资料
> - []()
> - []()
