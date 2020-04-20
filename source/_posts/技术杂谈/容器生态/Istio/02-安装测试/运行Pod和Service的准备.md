---
title: 运行Pod和Service的准备
toc: true
date: 2019-10-19 13:15:12
tags:
categories:
---



要成为Istio服务网格的一部分，Kubernetes集群中的Pod和Service必须满足以下要求：

- 命名服务端口：服务端口必须命名。端口名称键/值对必须具有以下语法：name: <protocol>[-<suffix>]。要利用Istio的路由功能，请将<protocol>替换为以下值之一：grpc http http2 https mongo mysql redis tcp tls udp

  例如，名称： `name: http2-foo`或`name: http`是有效的端口名，但name：http2foo不是。如果端口名称不是以可识别的前缀开头，或者端口未命名，则将自动检测出站HTTP或TCP通信。除非端口明确使用协议：UDP表示UDP端口，否则将端口上的入站流量视为普通TCP流量。

- 服务关联：即使Pod不公开任何端口，pod也必须至少属于一个Kubernetes服务。如果Pod属于多个Kubernetes服务，则这些服务不能将相同的端口号用于不同的协议，例如HTTP和TCP。

- 具有app和version标签的deployment：我们建议为deployment添加显式的app标签和version标签。将标签添加到使用Kubernetes Deployment部署的Pod的部署规范中。app和version标签将上下文信息添加到Istio收集的指标和遥测中。

  app标签：每个部署规范都应有一个具有有意义值的不同app标签。app标签用于在分布式跟踪中添加上下文信息。

  version标签：此标签指示与特定部署相对应的应用程序版本。

- 应用程序UID：确保您的Pod不以用户ID（UID）值为1337的用户身份运行应用程序。

- NET_ADMIN功能：如果您的集群强制执行Pod安全策略，则Pod必须允许NET_ADMIN功能。如果使用Istio CNI插件，则此要求不再适用。



## Istio使用的端口

Istio使用以下端口和协议。确保没有TCP无头服务使用这些端口。

| Port  | Protocol | Used by                                                      | Description                                |
| ----- | -------- | ------------------------------------------------------------ | ------------------------------------------ |
| 8060  | HTTP     | Citadel                                                      | GRPC server                                |
| 8080  | HTTP     | Citadel agent                                                | SDS service monitoring                     |
| 9090  | HTTP     | Prometheus                                                   | Prometheus                                 |
| 9091  | HTTP     | Mixer                                                        | Policy/Telemetry                           |
| 9876  | HTTP     | Citadel, Citadel agent                                       | ControlZ user interface                    |
| 9901  | GRPC     | Galley                                                       | Mesh Configuration Protocol                |
| 15000 | TCP      | Envoy                                                        | Envoy admin port (commands/diagnostics)    |
| 15001 | TCP      | Envoy                                                        | Envoy Outbound                             |
| 15006 | TCP      | Envoy                                                        | Envoy Inbound                              |
| 15004 | HTTP     | Mixer, Pilot                                                 | Policy/Telemetry - `mTLS`                  |
| 15010 | HTTP     | Pilot                                                        | Pilot service - XDS pilot - discovery      |
| 15011 | TCP      | Pilot                                                        | Pilot service - `mTLS` - Proxy - discovery |
| 15014 | HTTP     | Citadel, Citadel agent, Galley, Mixer, Pilot, Sidecar Injector | Control plane monitoring                   |
| 15020 | HTTP     | Ingress Gateway                                              | Pilot health checks                        |
| 15029 | HTTP     | Kiali                                                        | Kiali User Interface                       |
| 15030 | HTTP     | Prometheus                                                   | Prometheus User Interface                  |
| 15031 | HTTP     | Grafana                                                      | Grafana User Interface                     |
| 15032 | HTTP     | Tracing                                                      | Tracing User Interface                     |
| 15443 | TLS      | Ingress and Egress Gateways                                  | SNI                                        |
| 15090 | HTTP     | Mixer                                                        | Proxy                                      |
| 42422 | TCP      | Mixer                                                        | Telemetry - Prometheus                     |

## 参考资料
> - []()
> - []()
