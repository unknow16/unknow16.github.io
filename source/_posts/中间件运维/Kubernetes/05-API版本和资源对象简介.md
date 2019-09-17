---
title: 05-API版本和资源对象简介
toc: true
date: 2019-08-29 15:17:39
tags:
categories:
---

## API版本结构

获取当前kubernetes集群支持的API versions

```
$ kubectl api-versions
```

以我当前集群v1.15为例，支持API versions如下

```
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1
apiregistration.k8s.io/v1beta1
apps/v1
apps/v1beta1
apps/v1beta2
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
autoscaling/v2beta2
batch/v1
batch/v1beta1
certificates.k8s.io/v1beta1
coordination.k8s.io/v1
coordination.k8s.io/v1beta1
events.k8s.io/v1beta1
extensions/v1beta1
networking.k8s.io/v1
networking.k8s.io/v1beta1
node.k8s.io/v1beta1
policy/v1beta1
rbac.authorization.k8s.io/v1
rbac.authorization.k8s.io/v1beta1
scheduling.k8s.io/v1
scheduling.k8s.io/v1beta1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
```

如上支持的API版本列表中，除了v1外，其他的格式为：group/version。

- v1: 没有组，表示稳定版本，包含了核心API对象： pod、service等
- group: 表示一类非核心API对象，可以同时有多个版本，为了做向前兼容
- version:  表示该组的版本，下面对各种version的含义解释

**alpha**

```
* 该软件可能包含错误。启用一个功能可能会导致bug
* 随时可能会丢弃对该功能的支持，恕不另行通知
```

**beta**

```
* 软件经过很好的测试。启用功能被认为是安全的。
* 默认情况下功能是开启的
* 细节可能会改变，但功能在后续版本不会被删除
```

**stable**

```
* 该版本名称命名方式：vX这里X是一个整数
* 稳定版本、放心使用
* 将出现在后续发布的软件版本中
```



## 常见API版本Group

- **apps/v1beta2**

在kubernetes1.8版本中，新增加了apps/v1beta2的概念，apps/v1beta1同理 

DaemonSet，Deployment，ReplicaSet 和 StatefulSet的当时版本迁入apps/v1beta2，兼容原有的extensions/v1beta1

- **apps/v1**

包含一些通用的应用层的api组合，如：Deployments, RollingUpdates, and ReplicaSets

在kubernetes1.9版本中，引入apps/v1，deployment等资源从extensions/v1beta1, apps/v1beta1 和 apps/v1beta2迁入apps/v1，原来的v1beta1等被废弃。

- **batch/v1**

代表job相关的api组合

在kubernetes1.8版本中，新增了batch/v1beta1，后CronJob 已经迁移到了 batch/v1beta1，然后再迁入batch/v1

- **autoscaling/v1**

代表自动扩缩容的api组合，kubernetes1.8版本中引入。

这个组合中后续的alpha 和 beta版本将支持基于memory使用量、其他监控指标进行扩缩容

- **extensions/v1beta1**

deployment等资源在1.6版本时放在这个版本中，后迁入到apps/v1beta2,再到apps/v1中统一管理

- **certificates.k8s.io/v1beta1**

安全认证相关的api组合

- **authentication.k8s.io/v1**

资源鉴权相关的api组合

## API资源对象

可以使用下面命令查看当前集群支持的资源对象

```
kubectl api-resources
```

下面会介绍些常用的资源对象

| 类别               | 名称                                                         |
| ------------------ | ------------------------------------------------------------ |
| 工作负载型资源对象 | Pod  Replicaset  ReplicationController  Deployments StatefulSets Daemonset  Job CronJob |
| 服务发现及负载均衡 | Service  Ingress                                             |
| 配置与存储         | Volume、Persistent Volume、CSl 、 configmap、  secret        |
| 集群资源           | Namespace Node Role ClusterRole  RoleBinding  ClusterRoleBinding |
| 元数据资源         | HPA PodTemplate LimitRang                                    |

## Pod

Pod是Kubernetes最基本的操作单元，包含一个或多个紧密相关的容器，一个Pod可以被一个容器化的环境看作应用层的“逻辑宿主机”。一个Pod中的多个容器应用通常是紧密耦合的。

Pod在Node上被创建、启动或者销毁，每个Pod里运行着一个特殊的被称之为Pause的容器，其他容器则为业务容器，这些业务容器共享Pause容器的网络栈和Volume挂载卷，因此他们之间通信和数据交换更为高效，在设计时我们可以充分利用这一特性将一组密切相关的服务进程放入同一个Pod中。同一个Pod里的容器之间仅需通过localhost就能互相通信。

一个Pod中的应用容器共享同一组资源：

- PID命名空间：Pod中的不同应用程序可以看到其他应用程序的进程ID；
- 网络命名空间：Pod中的多个容器能够访问同一个IP和端口范围；
- IPC命名空间：Pod中的多个容器能够使用SystemV IPC或POSIX消息队列进行通信；
- UTS命名空间：Pod中的多个容器共享一个主机名；
- Volumes（共享存储卷）：Pod中的各个容器可以访问在Pod级别定义的Volumes；

Kubernetes 运行容器（Pod）与访问容器（Pod）这两项任务分别由 Controller 和 Service 执行。

## Pod控制器

  在K8S的集群设计中，Pod是一个有生命周期的对象。那么用户通过手工创建或者通过Controller直接创建的Pod对象会被调度器（Scheduler）调度到集群中的某个工作节点上运行，等到容器应用进程运行结束之后正常终止，随后就会被删除。而需要注意的是，当节点的资源耗尽或者故障，也会导致Pod对象的回收。

  而K8S在这一设计上，使用了控制器实现对一次性的Pod对象进行管理操作。比如，要确保部署的应用程序的Pod副本数达到用户预期的数目，以及基于Pod模板来重建Pod对象等，从而实现Pod对象的扩容、缩容、滚动更新和自愈能力。例如，在某个节点故障，相关的控制器会将运行在该节点上的Pod对象重新调度到其他节点上进行重建。

  控制器本身也是一种资源类型，其中包括Replication、Controller、Deployment、StatefulSet、DaemonSet、Jobs等等，它们都统称为Pod控制器。

Pod控制器的定义通常由期望的副本数量、Pod模板、标签选择器组成。Pod控制器会根据标签选择器来对Pod对象的标签进行匹配筛选，所有满足选择条件的Pod对象都会被当前控制器进行管理并计入副本总数，确保数目能够达到预期的状态副本数。

  需要注意的是，在实际的应用场景中，在接收到的请求流量负载低于或接近当前已有Pod副本的承载能力时，需要我们手动修改Pod控制器中的期望副本数量以实现应用规模的扩容和缩容。而在集群中部署了HeapSet或者Prometheus的这一类资源监控组件时，用户还可以通过HPA（HorizontalPodAutoscaler）来计算出合适的Pod副本数量，并自动地修改Pod控制器中期望的副本数，从而实现应用规模的动态伸缩，提高集群资源的利用率。

  K8S集群中的每个节点上都运行着cAdvisor，用于收集容器和节点的CPU、内存以及磁盘资源的利用率直播数据，这些统计数据由Heapster聚合之后可以通过API  server访问。而HorizontalPodAutoscaler基于这些统计数据监控容器的健康状态并作出扩展决策。



为了满足不同的业务场景，Kubernetes 提供了多种 Controller，我们逐一讨论。

- ReplicaSet

代用户创建指定数量的pod副本数量，确保pod副本数量符合预期状态，并且支持滚动式自动扩容和缩容功能，帮助用户管理无状态的pod资源，精确反应用户定义的目标数量，但是RelicaSet不是直接使用的控制器，而是使用Deployment。

- Deployment

工作在ReplicaSet之上，用于管理无状态应用，目前来说最好的控制器。支持滚动更新和回滚功能，还提供声明式配置。

- ReplicationController

跟ReplicaSet没有本质的不同，只是名字不一样，在新版本的Kubernetes中建议使用ReplicaSet来取代ReplicationController，虽然ReplicaSet可以独立使用，但一般还是建议使用 Deployment 来自动管理ReplicaSet。

- DaemonSet

用于每个 Node 最多只运行一个 Pod 副本的场景。正如其名称所揭示的，DaemonSet 通常用于运行 daemon。

- Job

只要完成就立即退出，不需要重启或重建。

- Cronjob

周期性任务控制，不需要持续后台运行

- StatefulSet

管理有状态应用，能够保证 Pod 的每个副本在整个生命周期中名称是不变的。而其他 Controller 不提供这个功能，当某个 Pod 发生故障需要删除并重新启动时，Pod 的名称会发生变化。同时 StatefuleSet 会保证副本按照固定的顺序启动、更新或者删除。

## **Service** 

Deployment 可以部署多个副本，每个 Pod 都有自己的 IP，外界如何访问这些副本呢？

通过 Pod 的 IP 吗？要知道 Pod 很可能会被频繁地销毁和重启，它们的 IP 会发生变化，用 IP 来访问不太现实。答案是 Service。

一个Service可以看作一组提供相同服务的Pod的对外访问接口，Service作用于哪些Pod是通过Label Selector来定义的。

如果K8S存在DNS附件（如coredns）它就会在Service创建时为它自动配置一个DNS名称，用于客户端进行服务发现。

通常我们直接请求Service  IP，该请求就会被负载均衡到后端的端点，即各个Pod对象，从这点上，是不是有点像负载均衡器呢，因此Service本质上是一个4层的代理服务，另外Service还可以将集群外部流量引入至集群，这就需要Service的NodePort或外部负载均衡器.

## NameSpace

如果有多个用户或项目组使用同一个 Kubernetes Cluster，如何将他们创建的 Controller、Pod 等资源分开呢？答案就是 Namespace。

Namespace 可以将一个物理的 Cluster 逻辑上划分成多个虚拟 Cluster，每个 Cluster 就是一个 Namespace。不同 Namespace 里的资源是完全隔离的。

默认创建了两个 Namespace。

- default： 创建资源时如果不指定，将被放到这个 Namespace 中。

- kube-system： Kubernetes 自己创建的系统资源将放到这个 Namespace 中。

## ServiceAccount

见名知义，ServiceAccount即服务的账户，有些情况下，我们希望在pod内部访问api-server，获取集群的信息，甚至对集群进行改动。针对这种情况，pod做为一个访问者，需要有一个ServiceAccount去访问。

Service Account 是面向 namespace 的，每个 namespace 创建的时候，kubernetes 会自动在这个 namespace 下面创建一个名字为default的默认 Service Account；并且这个 Service Account 只能访问该 namespace 的资源。

## Secret

Secret解决了密码、token、密钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者Pod Spec中。

创建集群时，默认会为每个namespace下创建的一个名为default-token-xxxx的secret，被每个namespace下默认创建的名为default的ServiceAccount引用。

Secret可以以Volume或者环境变量的方式使用。

## Ingress

Ingress是一组基于虚拟主机或URL路径把请求转发到指定的Service资源的规则。

我们需要明白的是，Ingress资源自身不能进行“流量穿透”，仅仅是一组规则的集合，这些集合规则还需要其他功能的辅助，比如监听某套接字，然后根据这些规则的匹配进行路由转发，这些能够为Ingress资源监听套接字并将流量转发的组件就是Ingress Controller。

Ingress 控制器不同于Deployment 控制器的是，Ingress控制器不直接运行为kube-controller-manager的一部分，它仅仅是Kubernetes集群的一个附件，类似于CoreDNS，需要在集群上单独部署。

目前主流的Ingress 控制器的选型是nginx和traefik。

##  Volume
在使用容器时，我们知道，当数据存放于容器之中，容器销毁后，数据也会随之丢失。这就是需要一个外部存储，以保证数据的持久化存储。而存储卷就是这样的一个东西。

存储卷（Volume）是独立于容器文件系统之外的存储空间，常用于扩展容器的存储空间并为其提供持久存储能力。存储卷在K8S中的分类为：临时卷、本地卷和网络卷。临时卷和本地卷都位于Node本地，一旦Pod被调度至其他Node节点，此类型的存储卷将无法被访问，因为临时卷和本地卷通常用于数据缓存，持久化的数据通常放置于持久卷（persistent  volume）之中

## 参考资料

> - []()
> - []()
