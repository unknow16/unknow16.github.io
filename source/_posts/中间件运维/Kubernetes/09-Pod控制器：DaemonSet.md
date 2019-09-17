---
title: 09-Pod控制器：DaemonSet
toc: true
date: 2019-08-31 01:04:51
tags:
categories:
---



## 什么是DaemonSet？

DaemonSet 确保全部（或者一些）Node 上运行一个 Pod 的副本。当有 Node 加入集群时，也会为他们新增一个 Pod 。当有 Node 从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。

使用 DaemonSet 的一些典型用法：

- 运行集群存储 daemon，例如在每个 Node 上运行 `glusterd`、`ceph`。
- 在每个 Node 上运行日志收集 daemon，例如`fluentd`、`logstash`。
- 在每个 Node 上运行监控 daemon，例如 [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)、`collectd`、Datadog 代理、New Relic 代理，或 Ganglia `gmond`。



## Pod通信方式

与 DaemonSet 中的 Pod 进行通信，几种可能的模式如下：

- Push：配置 DaemonSet 中的 Pod 向其它 Service 发送更新，例如统计数据库。它们没有客户端。
- NodeIP 和已知端口：DaemonSet 中的 Pod 可以使用 `hostPort`，从而可以通过 Node IP 访问到 Pod。客户端能通过某种方法知道 Node IP 列表，并且基于此也可以知道端口。
- DNS：创建具有相同 Pod Selector 的 [Headless Service](https://kubernetes.io/docs/user-guide/services/#headless-services)，然后通过使用 `endpoints` 资源或从 DNS 检索到多个 A 记录来发现 DaemonSet。
- Service：创建具有相同 Pod Selector 的 Service，并使用该 Service 访问到某个随机 Node 上的 daemon。（没有办法访问到特定 Node）



## 部署配置示例

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat-ds
  namespace: default
spec:
  selector:
    matchLabels:
      app: filebeat
      release: stable
  template:
    metadata:
      labels: 
        app: filebeat
        release: stable
    spec:
      containers:
      - name: filebeat
        image: ikubernetes/filebeat:5.6.5-alpine
        env:
        - name: REDIS_HOST
          value: redis.default.svc.cluster.local
        - name: REDIS_LOG_LEVEL
          value: info
```

## 更新策略

DaemonSet有两种更新策略类型：

- OnDelete：这是向后兼容性的默认更新策略。使用 `OnDelete`更新策略，在更新DaemonSet模板后，只有在手动删除旧的DaemonSet pod时才会创建新的DaemonSet pod。这与Kubernetes 1.5或更早版本中DaemonSet的行为相同。
- RollingUpdate：使用`RollingUpdate`更新策略，在更新DaemonSet模板后，旧的DaemonSet pod将被终止，并且将以受控方式自动创建新的DaemonSet pod。

要启用DaemonSet的滚动更新功能，必须将其设置`.spec.updateStrategy.type`为`RollingUpdate`。

- 查看当前的更新策略：

  ```
  [root@k8s-master mainfests]# kubectl get ds/filebeat-ds -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}'
  RollingUpdate
  ```

- 滚动更新特性

DaemonSet的.spec.template任何更新都将触发滚动更新，在更新过程中，是先终止旧的pod，再创建一个新的pod，逐步进行替换的，仅仅更新容器镜像还可以使用以下命令：

```
kubectl set image ds/<daemonset-name> <container-name>=<container-new-image>
```

## 参考资料

> - []()
> - []()
