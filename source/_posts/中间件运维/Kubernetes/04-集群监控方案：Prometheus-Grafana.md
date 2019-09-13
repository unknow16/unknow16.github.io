---
title: Kubernetes集群监控方案：Prometheus+Grafana
toc: true
date: 2019-08-30 20:32:45
tags:
categories:
---



一个集群系统管理离不开监控，同样的Kubernetes也需要根据数据指标来采集相关数据，从而完成对集群系统的监控状况进行监测。这些指标总体上分为两个组成：监控集群本身和监控Pod对象，通常一个集群的衡量性指标包括以下几个部分：

- 节点资源状态：主要包括网络带宽、磁盘空间、CPU和内存使用率
- 节点的数量：即时性了解集群的可用节点数量可以为用户计算服务器使用的费用支出提供参考。
- 运行的Pod对象：正在运行的Pod对象数量可以评估可用节点数量是否足够，以及节点故障时是否能平衡负载。

另一个方面，对Pod资源对象的监控需求大概有以下三类：

- Kubernetes指标：监测特定应用程序相关的Pod对象的部署过程、副本数量、状态信息、健康状态、网络等等。
- 容器指标：容器的资源需求、资源限制、CPU、内存、磁盘空间、网络带宽的实际占用情况。
- 应用程序指标：应用程序自身的内建指标，和业务规则相关



## Weave Scope

Weave Scope 是 Docker 和 Kubernetes 可视化监控工具。Scope 
提供了至上而下的集群基础设施和应用的完整视图，用户可以轻松对分布式的容器化应用进行实时监控和问题诊断。 
对于复杂的应用编排和依赖关系，scope可以使用清晰的图标一览应用状态和拓扑关系。

官网： https://www.weave.works/oss/scope/





## metrics-server

在最初的系统资源监控，是通过`cAdvisor`去收集单个节点以及相关Pod资源的指标数据，但是这一功能仅能够满足单个节点，在集群日益庞大的过程中，该功能就显得low爆了。于是将各个节点的指标数据进行汇聚并通过一个借口进行向外暴露传送是必要的。

`Heapster`就是这样的一种方式，通过为集群提供指标API和实现并进行监控，它是集群级别的监控和事件数据的聚合工具，但是一个完备的Heapster监控体系是需要进行数据存储的，为此其解决方案就是引入了`Influxdb`作为后端数据的持久存储，`Grafana`作为可视化的接口。

原理就是Heapster从各个节点上的cAdvisor采集数据并存储到Influxdb中，再由Grafana展示。



时代在变迁，陈旧的东西将会被淘汰，由于功能和系统发展的需求，Heapster无法满足k8s系统监控的需求，逐渐地Heapster用于提供核心指标API的功能也被聚合方式的指标API服务器`metrics-server`所替代。

在新一代的`Kubernetes`指标监控体系当中主要由核心指标流水线和监控指标流水线组成：

- 核心指标流水线：是指由kubelet、、metrics-server以及由API  server提供的api组成，它们可以为K8S系统提供核心指标，从而了解并操作集群内部组件和程序。其中相关的指标包括CPU的累积使用率、内存实时使用率，Pod资源占用率以及容器磁盘占用率等等。其中核心指标的获取原先是由heapster进行收集，但是在1.11版本之后已经被废弃，从而由新一代的metrics-server所代替对核心指标的汇聚。核心指标的收集是必要的。如下图：

  ![]()

- 监控指标流水线：用于从系统收集各种指标数据并提供给终端用户、存储系统以及HPA。它们包含核心指标以及许多非核心指标，其中由于非核心指标本身不能被Kubernetes所解析，此时就需要依赖于用户选择第三方解决方案。如下图：

  ![]()



- 1.7版本以后引入了自定义指标(custom metrics API)
- 1.8版本引入了资源指标（resource metrics API）。
- 1.11版本之后废弃了heapster，由新一代的metrics-server代替对核心指标的汇聚。

一个可以同时使用资源指标API和自定义指标API的组件是HPAv2，其实现了通过观察指标实现自动扩容和缩容。而目前资源指标API的实现主流是`metrics-server`。

自1.8版本后，容器的cpu和内存资源占用利用率都可以通过客户端指标API直接调用，从而获取资源使用情况，要知道的是API本身并不存储任何指标数据，仅仅提供资源占用率的实时监测数据。

资源指标和其他的API指标并没有啥区别，它是通过API Server的URL路径`/apis/metrics.k8s.io/`进行存取，只有在k8s集群内部署了`metrics-server`应用才能只用API，其简单的结构图如下：

![]()

MetricsServer通过Kubernetes聚合器（kube-aggregator）注册到主APIServer之上，而后基于kubelet的SummaryAPI收集每个节点上的指标数据，并将它们存储于内存中然后以指标API格式提供，如下图：

![]()

MetricsServer基于内存存储，重启后数据将全部丢失，而且它仅能留存最近收集到的指标数据，因此，如果用户期望访问历史数据，就不得不借助于第三方的监控系统（如Prometheus等）。

一般说来，MetricsServer在每个集群中仅会运行一个实例，启动时，它将自动初始化与各节点的连接，因此出于安全方面的考虑，它需要运行于普通节点而非Master主机之上。直接使用项目本身提供的资源配置清单即能轻松完成metrics-server的部署。



## 部署metrics-server

===============



## Prometheus支持

自定义指标(custom metrics API)，其指标API的实现要指定相应的后端监视系统。而`Prometheus`是第一个开发了相应适配器的监控系统。这个就是适用于`Prometheus`的`Kubernetes Customm Metrics Adapter`是属于Github上的[k8s-prometheus-adapter](https://github.com/DirectXMan12/k8s-prometheus-adapter)项目提供的。其原理图如下：

![]()

要知道的是`prometheus`本身就是一监控系统，也分为`server`端和`agent`端，`server`端从被监控主机获取数据，而`agent`端需要部署一个`node_exporter`，主要用于数据采集和暴露节点的数据，那么 在获取Pod级别或者是mysql等多种应用的数据，也是需要部署相关的`exporter`。我们可以通过`PromQL`的方式对数据进行查询，但是由于本身`prometheus`属于第三方的 解决方案，原生的k8s系统并不能对`Prometheus`的自定义指标进行解析，就需要借助于`k8s-prometheus-adapter`将这些指标数据查询接口转换为标准的`Kubernetes`自定义指标。

Prometheus是一个开源的服务监控系统和时序数据库，其提供了通用的数据模型和快捷数据采集、存储和查询接口。它的核心组件Prometheus服务器定期从静态配置的监控目标或者基于服务发现自动配置的目标中进行拉取数据，新拉取到的数据大于配置的内存缓存区时，数据就会持久化到存储设备当中。Prometheus组件架构图如下：

![]()

如上图，每个被监控的主机都可以通过专用的`exporter`程序提供输出监控数据的接口，并等待`Prometheus`服务器周期性的进行数据抓取。如果存在告警规则，则抓取到数据之后会根据规则进行计算，满足告警条件则会生成告警，并发送到`Alertmanager`完成告警的汇总和分发。当被监控的目标有主动推送数据的需求时，可以以`Pushgateway`组件进行接收并临时存储数据，然后等待`Prometheus`服务器完成数据的采集。

任何被监控的目标都需要事先纳入到监控系统中才能进行时序数据采集、存储、告警和展示，监控目标可以通过配置信息以静态形式指定，也可以让Prometheus通过服务发现的机制进行动态管理。下面是组件的一些解析：

- 监控代理程序：如node_exporter：收集主机的指标数据，如平均负载、CPU、内存、磁盘、网络等等多个维度的指标数据。
- kubelet（cAdvisor）：收集容器指标数据，也是K8S的核心指标收集，每个容器的相关指标数据包括：CPU使用率、限额、文件系统读写限额、内存使用率和限额、网络报文发送、接收、丢弃速率等等。
- API Server：收集API Server的性能指标数据，包括控制队列的性能、请求速率和延迟时长等等
- etcd：收集etcd存储集群的相关指标数据
- kube-state-metrics：该组件可以派生出k8s相关的多个指标数据，主要是资源类型相关的计数器和元数据信息，包括制定类型的对象总数、资源限额、容器状态以及Pod资源标签系列等。

Prometheus能够直接把KubernetesAPIServer作为服务发现系统使用进而动态发现和监控集群中的所有可被监控的对象。这里需要特别说明的是，Pod资源需要添加下列注解信息才能被Prometheus系统自动发现并抓取其内建的指标数据。

- prometheus.io/scrape：用于标识是否需要被采集指标数据，布尔型值，true或false。
- prometheus.io/path：抓取指标数据时使用的URL路径，一般为/metrics。
- prometheus.io/port：抓取指标数据时使用的套接字端口，如8080。

另外，仅期望Prometheus为后端生成自定义指标时仅部署Prometheus服务器即可，它甚至也不需要数据持久功能。但若要配置完整功能的监控系统，管理员还需要在每个主机上部署node_exporter、按需部署其他特有类型的exporter以及Alertmanager。



## 部署Prometheus

==========

## 参考资料
> - []()
> - []()
