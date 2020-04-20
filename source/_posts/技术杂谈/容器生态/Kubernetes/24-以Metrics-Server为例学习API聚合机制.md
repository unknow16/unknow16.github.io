---
title: 24-以Metrics-Server为例学习API聚合机制
toc: true
date: 2020-04-14 13:29:22
tags:
categories:
---



## 关于API聚合机制

API聚合机制是Kubernetes 1.7版本引入的特性，能够将用户扩展的API注册到kube-apiserver上，仍然通过主kube-apiserver的HTTP URL对新的API进行访问和操作。为了实现这个机制，Kubernetes在kube-apiserver服务中引入了一个API聚合层（API Aggregation Layer），用于将扩展API的访问请求转发到用户服务的功能。聚合层需要启动apiserver的时候开启方可使用。

在用户注册扩展API之前，聚合层什么也不做。用户要注册扩展API，必需向系统中添加一个APIService对象，APIService也是k8s的一种资源类型，用来声明API的URL路径以及处理该请求的后端APIService。此后，聚合层会将发往该路径的所有请求都转发给注册的APIService。

一般情况下，APIService对象以extension-apiserver运行在集群中的一个pod中，如果需要主动管理添加的资源，extension-apiserver还需要与一个或多个controlller进行关联，apiserver-builder为双方提供了一个框架。

Service Catalog是Kubernetes的一种API扩展实现，方便Kubernetes集群内部应用访问集群外部、由第三方管理、提供的服务，如由云供应商提供的数据库服务。Service Catalog的安装会为它所提供的服务提供extension-apiserver和controller两个扩展组件。



## 在Master的kube-apiserver中启用API聚合功能

为了能够将用户自定义的API注册到Master的API Server中，首先需要配置kube-apiserver服务的以下启动参数来启用API聚合功能。

```
--requestheader-client-ca-file=/etc/kubernetes/ssl_keys/ca.crt：客户端CA证书。
--requestheader-allowed-names=：允许访问的客户端common names列表，通过header中--requestheader-username-headers参数指定的字段获取。客户端common names的名称需要在client-ca-file中进行设置，将其设置为空值时，表示任意客户端都可访问。
--requestheader-extra-headers-prefix=X-Remote-Extra-:：请求头中需要检查的前缀名。
--requestheader-group-headers=X-Remote-Group：请求头中需要检查的组名。
--requestheader-username-headers=X-Remote-User：请求头中需要检查的用户名。
--proxy-client-cert-file=/etc/kubernetes/ssl_keys/kubelet_client.crt：在请求期间验证Aggregator的客户端CA证书。
--proxy-client-key-file=/etc/kubernetes/ssl_keys/kubelet_client.key：在请求期间验证Aggregator的客户端私钥。
```

如果kube-apiserver所在的主机上没有运行kube-proxy，即无法通过服务的ClusterIP进行访问，那么还需要设置以下启动参数：

```bash
--enable-aggregator-routing=true
```



## 注册自定义APIService资源

在启用了API聚合功能之后，用户就能将自定义API资源注册到Master的kube-apiserver中了。用户只需配置一个APIService资源对象，就能进行注册了。metrics-server的APIService配置如下：

```
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  name: v1beta1.metrics.k8s.io
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  service:
    name: metrics-server
    namespace: kube-system
  group: metrics.k8s.io
  version: v1beta1
  insecureSkipTLSVerify: true
  groupPriorityMinimum: 100
  versionPriority: 100
```

如下设置的API的Group为metrics.k8s.io，版本为v1beta1。这两个字段将作API路径的子目录注册到API路径“/apis/”下。注册后，可以通过kubectl api-versions命令查看是否注册成功，如果输出列表中有metrics.k8s.io/v1beta1，就能通过路径“/apis/metrics.k8s.io/v1beta1”访问自定义的API。

在spec.service下通过name和namespace设置了后端的自定义API Server来响应该扩展API，本例中的服务名为metrics-server，命名空间为kube-system。

上面APIService配置应用之后，通过对“/apis/metrics.k8s.io/v1beta1”路径的访问都会被API聚合层代理转发到后端服务metrics-server.kube-system.svc上了。



## 实现和部署自定义API Server

自定义API Server服务通常能以普通Pod的形式在Kubernetes集群中运行。当然，这个服务需要由自定义API的开发者提供，并且需要遵循Kubernetes的开发规范，详细的开发示例可以参考官方给出的示例：https://github.com/kubernetes/sample-apiserver。

下面以部署Metrics-Server为例，说明一个聚合API的实现方式。随着API聚合机制的出现，Heapster也进入弃用阶段，逐渐被Metrics Server替代。在部署完成后，Metrics-Server将通过“/apis/metrics.k8s.io/v1beta1”路径提供Pod和Node的监控数据。Metrics-Server通过聚合API提供Pod和Node的资源使用数据，供HPA控制器、VPA控制器及kubectl top命令使用。

- metrics-server-deployment.yaml

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: metrics-server-config
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: EnsureExists
data:
  NannyConfiguration: |-
    apiVersion: nannyconfig/v1alpha1
    kind: NannyConfiguration
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server-v0.3.6
  namespace: kube-system
  labels:
    k8s-app: metrics-server
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
    version: v0.3.6
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
      version: v0.3.6
  template:
    metadata:
      name: metrics-server
      labels:
        k8s-app: metrics-server
        version: v0.3.6
      annotations:
        seccomp.security.alpha.kubernetes.io/pod: 'docker/default'
    spec:
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      nodeSelector:
        kubernetes.io/os: linux
      containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server-amd64:v0.3.6
        command:
        - /metrics-server
        - --metric-resolution=30s
        # These are needed for GKE, which doesn't support secure communication yet.
        # Remove these lines for non-GKE clusters, and when GKE supports token-based auth.
        - --kubelet-port=10255
        - --deprecated-kubelet-completely-insecure=true
        - --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP
        ports:
        - containerPort: 443
          name: https
          protocol: TCP
      - name: metrics-server-nanny
        image: k8s.gcr.io/addon-resizer:1.8.5
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 5m
            memory: 50Mi
        env:
          - name: MY_POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: MY_POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
        volumeMounts:
        - name: metrics-server-config-volume
          mountPath: /etc/config
        command:
          - /pod_nanny
          - --config-dir=/etc/config
          - --cpu=100m
          - --extra-cpu=0.5m
          - --memory=100Mi
          - --extra-memory=50Mi
          - --threshold=5
          - --deployment=metrics-server-v0.3.6
          - --container=metrics-server
          - --poll-period=300000
          - --estimator=exponential
          # Specifies the smallest cluster (defined in number of nodes)
          # resources will be scaled to.
          - --minClusterSize=10
      volumes:
        - name: metrics-server-config-volume
          configMap:
            name: metrics-server-config
      tolerations:
        - key: "CriticalAddonsOnly"
          operator: "Exists"
```
- metrics-server-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: "Metrics-server"
spec:
  selector:
    k8s-app: metrics-server
  ports:
  - port: 443
    protocol: TCP
    targetPort: https
```

- resource-reader.yaml

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:metrics-server
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  - nodes/stats
  - namespaces
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - "apps"
  resources:
  - deployments
  verbs:
  - get
  - list
  - update
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:metrics-server
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
```


- auth-delegator.yaml

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: metrics-server:system:auth-delegator
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
```

- auth-reader.yaml

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: metrics-server-auth-reader
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
```


## 参考资料

> - []()
> - []()
