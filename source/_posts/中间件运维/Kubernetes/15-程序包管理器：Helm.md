---
title: 15-程序包管理器：Helm
toc: true
date: 2019-09-02 23:53:51
tags:
categories:
---



官网： https://helm.sh/

apps官网： https://hub.kubeapps.com/



## 为什么要有Helm?

每个成功的软件平台都有一个优秀的打包系统，比如 Debian、Ubuntu 的 apt，Redhat、Centos 的 yum。而 Helm 则是 Kubernetes 上的包管理器。

**思考？？**Helm 到底解决了什么问题？为什么 Kubernetes 需要 Helm？

Kubernetes 能够很好地组织和编排容器，但它缺少一个更高层次的应用打包工具，而 Helm 就是来干这件事的。

举个例子，我们需要部署一个MySQL服务，Kubernetes则需要部署以下对象：

1. 为了能够让外界访问到MySQL，需要部署一个mysql的service；
2. 需要进行定义MySQL的密码，则需要部署一个Secret；
3. Mysql的运行需要持久化的数据存储，此时还需要部署PVC；
4. 保证后端mysql的运行，还需要部署一个Deployment，以支持以上的对象。

针对以上对象，我们可以使用YAML文件进行定义并部署，但是仅仅对于单个的服务支持，如果应用需要由一个甚至几十个这样的服务组成，并且还需要考虑各种服务的依赖问题，可想而知，这样的组织管理应用的方式就显得繁琐。为此就诞生了一个工具Helm，就是为了解决Kubernetes这种应用部署繁重的现象。



## Helm核心概念

- Chart：一个helm程序包，是创建一个应用的信息集合，包含各种Kubernetes对象的配置模板、参数定义、依赖关系、文档说明等。可以将Chart比喻为yum中的软件安装包；
- Repository：Charts仓库，用于集中存储和分发Charts；
- Config：应用程序实例化安装运行时所需要的配置信息；
- Release：特定的Chart部署于目标集群上的一个实例，代表这一个正在运行的应用。当chart被安装到Kubernetes集群，就会生成一个release，chart可以多次安装到同一个集群，每次安装都是一个release。



## Helm运行架构

Helm主要由Helm客户端、Tiller服务器和Charts仓库组成：

- helm：客户端，GO语言编写，实现管理本地的Chart仓库，可管理Chart，与Tiller服务进行交互，用于发送Chart，实例安装、查询、卸载等操作。
- Tiller：服务端，通常运行在K8S集群之上。用于接收helm发来的Charts和Conifg，合并生成release，完成部署。

简单的说：Helm 客户端负责管理 chart；Tiller 服务器负责管理 release。



## 部署Helm

Helm的部署方式有两种：预编译的二进制程序和源码编译安装，这里使用二进制的方式进行安装

- 下载helm

```
[root@k8s-master ~]# wget https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-linux-amd64.tar.gz --no-check-certificate
[root@k8s-master ~]# tar -xf helm-v2.9.1-linux-amd64.tar.gz 
[root@k8s-master ~]# cd linux-amd64/
[root@k8s-master linux-amd64]# ls
helm  LICENSE  README.md
[root@k8s-master linux-amd64]# mv helm /usr/bin
[root@k8s-master linux-amd64]# helm version
Client: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
```

## 部署Tiller

helm第一次init时，需要链接api-server并进行认证，所以在运行helm时，会去读取kube-config文件，所以必须确认当前用户存在kube-config文件。

Tiller运行在K8s集群之上，也必须拥有集群的管理权限，也就是需要一个serviceaccount，进行一个clusterrolebinding到cluster-admin。

```
#创建tiller的rbac清单
[root@k8s-master helm]# vim tiller-rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system

[root@k8s-master helm]# kubectl apply -f tiller-rbac.yaml 
serviceaccount/tiller created
clusterrolebinding.rbac.authorization.k8s.io/tiller created
[root@k8s-master helm]# kubectl get sa -n kube-system |grep tiller
tiller                               1         18s

# helm init命令进行初始化时，会用到gcr.io/kubernetes-helm中的景象，需要提前下载，镜像标签和Helm同版本号
# 注意要在pod被调度到的节点提前下载，并打标签
[root@k8s-node01 ~]# docker pull jmgao1983/tiller:v2.9.1
v2.9.1: Pulling from jmgao1983/tiller
53969ec691ff: Pull complete 
ea45de95cb26: Pull complete 
495df31ed85a: Pull complete 
Digest: sha256:417aae19a0709075df9cc87e2fcac599b39d8f73ac95e668d9627fec9d341af2
Status: Downloaded newer image for jmgao1983/tiller:v2.9.1
[root@k8s-node01 ~]# docker tag jmgao1983/tiller:v2.9.1 gcr.io/kubernetes-helm/tiller:v2.9.1

[root@k8s-master ~]# helm init
Creating /root/.helm 
Creating /root/.helm/repository 
Creating /root/.helm/repository/cache 
Creating /root/.helm/repository/local 
Creating /root/.helm/plugins 
Creating /root/.helm/starters 
Creating /root/.helm/cache/archive 
Creating /root/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com 
Adding local repo with URL: http://127.0.0.1:8879/charts 
$HELM_HOME has been configured at /root/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!


[root@k8s-master ~]# kubectl get pods -n kube-system |grep tiller
tiller-deploy-759cb9df9-ls47p          1/1       Running   0          16m

#安装完成后，执行helm version可以看到客户端和服务端的版本号，两个都显示表示正常安装。
[root@k8s-master ~]# helm version
Client: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
```

如果希望在安装时自定义一些参数，可以参考一下的一些参数：

- --canary-image：安装canary分支，即项目的Master分支
- --tiller-image：安装指定版本的镜像，默认和helm同版本
- --kube-context：安装到指定的Kubernetes集群
- --tiller-namespace：安装到指定的名称空间，默认为kube-system

Tiller将数据存储在ConfigMap资源当中，卸载或重装不会导致数据丢失，卸载Tiller的方法有以下两种：

```
1. kubectl delete deployment tiller-deploy --n kube-system
2. heml reset
```

## 参考资料

> - []()
> - []()
