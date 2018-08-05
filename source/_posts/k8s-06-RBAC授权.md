---
title: k8s-06-RBAC授权
date: 2018-08-05 19:26:22
tags: Kubernetes
---

在Kubernetes中，授权(authorization)是在认证(authentication)之后的一个步骤。授权就是决定一个用户(普通用户或ServiceAccount)是否有权请求Kubernetes API做某些事情。

之前，Kubernetes中的授权策略主要是ABAC(Attribute-Based Access Control)。对于ABAC，Kubernetes在实现上是比较难用的，而且需要Master Node的SSH和根文件系统访问权限，授权策略发生变化后还需要重启API Server。

Kubernetes 1.6中，RBAC(Role-Based Access Control)基于角色的访问控制进入Beta阶段。RBAC访问控制策略可以使用kubectl或Kubernetes API进行配置。使用RBAC可以直接授权给用户，让用户拥有授权管理的权限，这样就不再需要直接触碰Master Node。在Kubernetes中RBAC被映射成API资源和操作。

## RBAC API的资源对象
RBAC API定义了四个资源对象用于描述RBAC中用户和资源之间的连接权限：

- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding

## Role和ClusterRole
Role是一系列权限的集合。Role是定义在某个Namespace下的资源，在这个具体的Namespace下使用。ClusterRole与Role相似，只是ClusterRole是整个集群范围内使用的。

如下命令打印集群中的Role和ClusterRole
```
kubectl get roles --all-namespaces

kubectl get ClusterRoles
```
可以看到集群中已经内置或创建很多的Role和ClusterRole。

#### 创建Role
下面在default命名空间内创建一个名称为pod-reader的Role，role-pord-reader.yaml文件如下：
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  namespace: default
  name: pod-reader
rules:
    # "" indicates the core API group
  - apiGroups: [""] 
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
```

#### 创建ClusterRole
下面再给一个ClusterRole的定义文件：
```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: secret-reader
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get", "watch", "list"]
```

## RoleBinding和ClusterRoleBinding
RoleBinding把Role绑定到账户主体Subject，让Subject继承Role所在namespace下的权限。ClusterRoleBinding把ClusterRole绑定到Subject，让Subject集成ClusterRole在整个集群中的权限。

账户主体Subject在这里还是叫“用户”吧，包含组group，用户user和ServiceAccount。

如下查看集群中的RoleBinding和ClusterRoleBinding

```
kubectl get rolebinding --all-namespaces

kubectl get clusterrolebinding
```

实际上一个RoleBinding既可以引用相同namespace下的Role；又可以引用一个ClusterRole，RoleBinding引用ClusterRole时用户继承的权限会被限制在RoleBinding所在的namespace下。

#### 创建RoleBinding

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: read-pods
  namespace: default
subjects:
  - kind: User
    name: jane
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Kubernetes中默认的Role和RoleBinding
API Server已经创建一系列ClusterRole和ClusterRoleBinding。这些资源对象中名称以system:开头的，表示这个资源对象属于Kubernetes系统基础设施。也就说RBAC默认的集群角色已经完成足够的覆盖，让集群可以完全在 RBAC的管理下运行。修改这些资源对象可能会引起未知的后果，例如对于system:node这个ClusterRole定义了kubelet进程的权限，如果这个角色被修改，可能导致kubelet无法工作。

可以使用kubernetes.io/bootstrapping=rbac-defaults这个label查看默认的ClusterRole和ClusterRoleBinding

```
kubectl get clusterrole -l kubernetes.io/bootstrapping=rbac-defaults

kubectl get clusterrolebinding -l kubernetes.io/bootstrapping=rbac-defaults
```

#### traefix-rabc.yaml示例

```
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
```
