---
title: 01-kubernetes架构
date: 2018-08-05 19:24:58
toc: true
tags: Kubernetes
---



先介绍一些kubernetes基础的概念

## Pod

Pod是Kubernetes最基本的操作单元，包含一个或多个紧密相关的容器，一个Pod可以被一个容器化的环境看作应用层的“逻辑宿主机”。一个Pod中的多个容器应用通常是紧密耦合的。同一个Pod里的容器之间仅需通过localhost就能互相通信。每个Pod分配一个IP地址。

## Pod Controller

通常不会直接创建 Pod，而是通过 Controller 来管理 Pod 的。Controller 中定义了 创建Pod的模版，比如有几个副本、用什么镜像、在什么样的 Node 上运行等。

Kubernetes通过RC中定义的Lable筛选出对应的Pod实例，并实时监控其状态和数量，如果实例数量少于定义的副本数量（Replicas），则会根据RC中定义的Pod模板来创建一个新的Pod，然后将此Pod调度到合适的Node上启动运行，直到Pod实例数量达到预定目标。

## Service
在Kubernetes的世界里，虽然每个Pod都会被分配一个单独的IP地址，但这个IP地址会随着Pod的销毁而消失，这就引出一个问题：如果有一组Pod组成一个集群来提供服务，那么如何来访问它呢？Service！

一个Service可以看作一组提供相同服务的Pod的对外访问接口，Service作用于哪些Pod是通过Label Selector来定义的

- 拥有一个虚拟IP（Cluster IP、Service IP或VIP）和端口号，销毁之前不会改变，只能内网访问；

- 能够提供某种远程服务能力；
- 被映射到了提供这种服务能力的一组容器应用上；

如果Service要提供外网服务，需指定公共IP和NodePort，或外部负载均衡器。

---

## Master节点组件

Kubernetes将集群中的机器划分为一个Master节点和一群工作节点（Node）。

- etcd 

负责保存k8s 集群的配置信息和各种资源的状态信息，当数据发生变化时，etcd会快速地通知k8s相关组件。

- kube-apiserver

提供了资源对象的唯一操作入口，其他所有组件都必须通过它提供的API来操作资源数据，通过对相关的资源数据“全量查询”+“变化监听”，这些组件可以很“实时”地完成相关的业务功能。

- kube-controller-manager 

集群内部的管理控制中心，其主要目的是实现Kubernetes集群的故障检测和恢复的自动化工作，保证资源处于预期的状态。比如根据Pod控制器的定义完成Pod的复制或移除，以确保Pod实例数符合副本的定义；根据Service与Pod的管理关系，完成服务的Endpoints对象的创建和更新；其他诸如Node的发现、管理和状态监控、死亡容器所占磁盘空间及本地缓存的镜像文件的清理等工作也是由Controller Manager完成的。

- kube-scheduler 

资源调度，负责决定将Pod放到哪个Node上运行。Scheduler在调度时会对集群的结构进行分析，当前各个节点的负载，以及应用对高可用、性能等方面的需求。 



## Node节点组件

- kubelet 

负责本Node节点上的Pod的创建、修改、监控、删除等全生命周期管理，同时Kubelet定时“上报”本Node的状态信息到API Server里。

- kube-proxy 

service在逻辑上代表了后端的多个Pod，外借通过service访问Pod。service接收到请求就需要kube-proxy完成转发到Pod的。每个Node都会运行kube-proxy服务，负责将访问的service的TCP/UDP数据流转发到后端的容器，如果有多个副本，kube-proxy会实现负载均衡，有2种方式：LVS或者Iptables 



## 客户端组件

- kubectl

客户端通过Kubectl命令行工具或Kubectl Proxy来访问Kubernetes系统，在Kubernetes集群内部的客户端可以直接使用Kuberctl命令管理集群



## 运行流程 
- 创建RC并创建Pod

1. 通过Kubectl提交一个创建RC的请求，该请求通过API Server被写入etcd中
2. 此时Controller Manager通过API Server的监听资源变化的接口监听到这个RC事件，分析之后，发现当前集群中还没有它所对应的Pod实例，于是根据RC里的Pod模板定义生成一个Pod对象，通过API Server写入etcd
3. 接下来，此事件被Scheduler发现，它立即执行一个复杂的调度流程，为这个新Pod选定一个落户的Node，然后通过API Server将这一结果写入到etcd中
4. 随后，目标Node上运行的Kubelet进程通过API Server监测到这个“新生的”Pod，并按照它的定义，启动该Pod并任劳任怨地负责它的下半生，直到Pod的生命结束。

- 创建Service

5. 随后，我们通过Kubectl提交一个新的映射到该Pod的Service的创建请求，Controller Manager会通过Label标签查询到相关联的Pod实例，然后生成Service的Endpoints信息，并通过API Server写入到etcd中
6. 接下来，所有Node上运行的Proxy进程通过API Server查询并监听Service对象与其对应的Endpoints信息，建立一个软件方式的负载均衡器来实现Service访问到后端Pod的流量转发功能。
