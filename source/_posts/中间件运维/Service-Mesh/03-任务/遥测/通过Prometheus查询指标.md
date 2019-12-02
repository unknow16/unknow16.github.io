---
title: 通过Prometheus查询指标
toc: true
date: 2019-11-28 16:12:41
tags:
categories:
---





默认istio-v1.0安装了prometheus。

## 访问UI

有两种方式

- 端中转发，需要iptable支持

- prometheus服务的NodePort方式暴露



## 查询测试

- 对productpage服务的所有请求总数：

```
istio_requests_total{destination_service="productpage.default.svc.cluster.local"}
```

- 对v3版本的评论服务的所有请求总数：

```
istio_requests_total{destination_service="reviews.default.svc.cluster.local", destination_version="v3"}
```

- 过去5分钟内对productpage服务所有实例的请求率：

```
rate(istio_requests_total{destination_service=~"productpage.*", response_code="200"}[5m])
```



## 关于Prometheus

Mixer带有内置的Prometheus适配器，该适配器公开了为生成的度量值提供服务的端点。Prometheus附加组件是Prometheus服务器，已预先配置为刮取Mixer端点以收集公开的指标。它提供了一种持久存储和查询Istio指标的机制。

配置的Prometheus附加组件将刮以下端点：

1. `istio-telemetry.istio-system:42422` :  istio-mesh作业将返回Mixer生成的指标。
2. `istio-telemetry.istio-system:15014`: istio-telemetry作业将返回Mixer生成的指标。使用这个端点监视Mixer自己。
3. `istio-proxy:15090`: envoy-stats作业返回Envoy生成的原始状态。Prometheus配置为查找带有envoy-prom端点的Pod。附加配置在收集过程中会过滤掉大量的特使指标，以试图限制附加过程的数据规模。
4. `istio-pilot.istio-system:15014`:  pilot作业将返回Pilot生成的指标。
5. `istio-galley.istio-system:15014`: galley作业将返回Galley生成的指标。
6. `istio-policy.istio-system:15014`:   istio-policy作业返回策略相关指标。
7. `istio-citadel.istio-system:15014`:  istio-citadel作业返回Citadel生成的指标。



## 参考资料
> - []()
> - []()
