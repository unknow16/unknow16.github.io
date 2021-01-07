---
title: 通过Grafana可视化指标
toc: true
date: 2019-11-28 14:36:19
tags:
categories:
---









默认istio-v1.0安装了prometheus和Granfana。

## 访问UI

有两种方式

- 端中转发，需要iptable支持

```
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000 &
```

访问地址	http://ip :16686

- granfana服务的NodePort方式暴露



## 关于Grafana附件

Grafana附加组件是Grafana的预配置实例。基本映像（grafana / grafana：5.0.4）已修改为从Prometheus数据源和安装的Istio仪表盘开始。Istio（尤其是Mixer）的基本安装文件附带了全局（用于每个服务）度量的默认配置。Istio仪表板旨在与默认Istio指标配置和Prometheus后端结合使用。

Istio仪表板包括三个主要部分：

1. 网格摘要视图。本节提供了网格的全局摘要视图，并显示了网格中的HTTP / gRPC和TCP工作负载。
2. 单个服务视图。本部分提供有关网格内每个单独服务（HTTP / gRPC和TCP）的请求和响应的度量。另外，提供有关此服务的客户端和服务工作负载的指标。
3. 单个工作负载视图：此部分提供有关网格内每个单独工作负载（HTTP / gRPC和TCP）的请求和响应的度量。另外，提供有关入站工作负载和该工作负载的出站服务的指标。

## 参考资料
> - []()
> - []()
