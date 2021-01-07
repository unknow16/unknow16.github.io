---
title: 19-程序包管理器-Helm
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



## Helm命令行

helm常用命令：
- helm search:    搜索charts
- helm fetch:     下载charts到本地目录
- helm install:   安装charts
- helm list:      列出charts的所有版本

用法:

- helm [command]

命令可用选项:

- completion  为指定的shell生成自动补全脚本（bash或zsh）
- create      创建一个新的charts
- delete      删除指定版本的release
- dependency  管理charts的依赖
- fetch       下载charts并解压到本地目录
- get         下载一个release
- history     release历史信息
- home        显示helm的家目录
- init        在客户端和服务端初始化helm
- inspect     查看charts的详细信息
- install     安装charts
- lint        检测包的存在问题
- list        列出release
- package     将chart目录进行打包
- plugin      add(增加), list（列出）, or remove（移除） Helm 插件
- repo        add(增加), list（列出）, remove（移除）, update（更新）, and index（索引） chart仓库
- reset       卸载tiller
- rollback    release版本回滚
- search      关键字搜索chart
- serve       启动一个本地的http server
- status      查看release状态信息
- template    本地模板
- test        release测试
- upgrade     release更新
- verify      验证chart的签名和有效期
- version     打印客户端和服务端的版本信息 



## Helm的使用

Charts是Helm的程序包，它们都存在在Charts仓库当中。Kubernetes官方的仓库保存了一系列的Charts，仓库默认的名称为`stable`。安装Charts到集群时，Helm首先会到官方仓库获取相关的Charts，并创建release。可执行 `helm search` 查看当前可安装的 chart 。

```
[root@k8s-master ~]# helm search
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
stable/acs-engine-autoscaler    2.1.3           2.1.1           Scales worker nodes within agent pools            
stable/aerospike                0.1.7           v3.14.1.2       A Helm chart for Aerospike in Kubernetes          
stable/anchore-engine           0.1.3           0.1.6           Anchore container analysis and policy evaluatio...
......
```

这些 chart 都是从哪里来的？

前面说过，Helm 可以像 yum 管理软件包一样管理 chart。 yum 的软件包存放在仓库中，同样的，Helm 也有仓库。

```
[root@k8s-master ~]# helm repo list
NAME    URL                                                   
local   http://127.0.0.1:8879/charts                          
stable  https://kubernetes-charts.storage.googleapis.com
```

Helm 安装时已经默认配置好了两个仓库：`stable` 和 `local`。`stable` 是官方仓库，`local` 是用户存放自己开发的`chart`的本地仓库。可以通过`helm repo list`进行查看。由于网络原因，国内无法更新仓库源，这里更改为阿里云的仓库源。

```
[root@k8s-master helm]# helm repo update        #仓库更新有时会提示无法连接
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Unable to get an update from the "stable" chart repository (https://kubernetes-charts.storage.googleapis.com):
    Get https://kubernetes-charts.storage.googleapis.com/index.yaml: dial tcp 216.58.220.208:443: connect: connection refused
Update Complete. ⎈ Happy Helming!⎈ 

[root@k8s-master helm]# helm repo list
NAME    URL                                             
stable  https://kubernetes-charts.storage.googleapis.com
local   http://127.0.0.1:8879/charts     

[root@k8s-master helm]# helm repo remove stable #移除stable repo
"stable" has been removed from your repositories
[root@k8s-master helm]# helm repo list
NAME    URL                         
local   http://127.0.0.1:8879/charts

#增加阿里云的charts仓库
[root@k8s-master helm]# helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts 
"stable" has been added to your repositories
[root@k8s-master helm]# helm repo list
NAME    URL                                                   
local   http://127.0.0.1:8879/charts                          
stable  https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
[root@k8s-master helm]# helm repo update #再次更新repo
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
```



与 yum 一样，helm 也支持关键字搜索：

```
[root@k8s-master ~]# helm search mysql
NAME                            CHART VERSION   APP VERSION DESCRIPTION                                       
stable/mysql                    0.3.5                       Fast, reliable, scalable, and easy to use open-...
stable/percona                  0.3.0                       free, fully compatible, enhanced, open source d...
stable/percona-xtradb-cluster   0.0.2           5.7.19      free, fully compatible, enhanced, open source d...
stable/gcloud-sqlproxy          0.2.3                       Google Cloud SQL Proxy                            
stable/mariadb                  2.1.6           10.1.31     Fast, reliable, scalable, and easy to use open-...
```



包括 DESCRIPTION 在内的所有信息，只要跟关键字匹配，都会显示在结果列表中。

安装 chart 也很简单，执行如下命令可以安装 MySQL。

```
[root@k8s-master ~]# helm install stable/mysql
Error: no available release name found

#如果看到上面的报错，通常是因为 Tiller 服务器的权限不足。执行以下命令添加权限：
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'


#helm安装mysql
[root@k8s-master helm]# helm install stable/mysql
NAME:   reeling-bronco  ①
LAST DEPLOYED: Wed Mar 27 03:10:31 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:  ②
==> v1/Secret
NAME                  TYPE    DATA  AGE
reeling-bronco-mysql  Opaque  2     0s

==> v1/PersistentVolumeClaim
NAME                  STATUS   VOLUME  CAPACITY  ACCESS MODES  STORAGECLASS  AGE
reeling-bronco-mysql  Pending  0s

==> v1/Service
NAME                  TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)   AGE
reeling-bronco-mysql  ClusterIP  10.99.245.169  <none>       3306/TCP  0s

==> v1beta1/Deployment
NAME                  DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
reeling-bronco-mysql  1        1        1           0          0s

==> v1/Pod(related)
NAME                                   READY  STATUS   RESTARTS  AGE
reeling-bronco-mysql-84b897b676-59qhh  0/1    Pending  0         0s


NOTES:  ③
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
reeling-bronco-mysql.default.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default reeling-bronco-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h reeling-bronco-mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following commands to route the connection:
    export POD_NAME=$(kubectl get pods --namespace default -l "app=reeling-bronco-mysql" -o jsonpath="{.items[0].metadata.name}")
    kubectl port-forward $POD_NAME 3306:3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```

输出分为三部分：

- ① chart 本次部署的描述信息：

`NAME` 是 release 的名字，因为我们没用 `-n` 参数指定，Helm 随机生成了一个，这里是 `reeling-bronco`。

`NAMESPACE` 是 release 部署的 namespace，默认是 `default`，也可以通过 `--namespace` 指定。

`STATUS` 为 `DEPLOYED`，表示已经将 chart 部署到集群。

- ② 当前 release 包含的资源：Service、Deployment、Secret 和 PersistentVolumeClaim，其名字都是 `reeling-bronco-mysql`，命名的格式为 `ReleasName`-`ChartName`。
- ③ `NOTES` 部分显示的是 release 的使用方法。比如如何访问 Service，如何获取数据库密码，以及如何连接数据库等。

通过 `kubectl get` 可以查看组成 release 的各个对象：

```
[root@k8s-master helm]# kubectl get service reeling-bronco-mysql
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
reeling-bronco-mysql   ClusterIP   10.99.245.169   <none>        3306/TCP   3m

[root@k8s-master helm]# kubectl get deployment reeling-bronco-mysql
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
reeling-bronco-mysql   1         1         1            0           3m

[root@k8s-master helm]# kubectl get pvc  reeling-bronco-mysql
NAME                   STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
reeling-bronco-mysql   Pending                                                      4m

[root@k8s-master helm]# kubectl get secret  reeling-bronco-mysql
NAME                   TYPE      DATA      AGE
reeling-bronco-mysql   Opaque    2         4m
```

由于我们还没有准备 PersistentVolume，当前 release 还不可用。

`helm list` 显示已经部署的 release，`helm delete` 可以删除 release。

```
[root@k8s-master helm]# helm list
NAME            REVISION    UPDATED                     STATUS      CHART       NAMESPACE
reeling-bronco  1           Wed Mar 27 03:10:31 2019    DEPLOYED    mysql-0.3.5 default  

[root@k8s-master helm]# helm delete reeling-bronco
release "reeling-bronco" deleted
```



## chart 目录结构

chart 是 Helm 的应用打包格式。chart 由一系列文件组成，这些文件描述了 Kubernetes 部署应用时所需要的资源，比如  Service、Deployment、PersistentVolumeClaim、Secret、ConfigMap 等。

单个的 chart 可以非常简单，只用于部署一个服务，比如 Memcached；chart 也可以很复杂，部署整个应用，比如包含 HTTP Servers、 Database、消息中间件、cache 等。

chart 将这些文件放置在预定义的目录结构中，通常整个 chart 被打成 tar 包，而且标注上版本信息，便于 Helm 部署。

以前面 MySQL chart 为例。一旦安装了某个 chart，我们就可以在 ~/.helm/cache/archive 中找到 chart 的 tar 包。

```
[root@k8s-master ~]# cd .helm/cache/archive/
[root@k8s-master archive]# ll
-rw-r--r-- 1 root root 5536 Oct 29 22:04 mysql-0.3.5.tgz
-rw-r--r-- 1 root root 6189 Oct 29 05:03 redis-1.1.15.tgz
[root@k8s-master archive]# tar -xf mysql-0.3.5.tgz
[root@k8s-master archive]# tree mysql
mysql
├── Chart.yaml  
├── README.md   
├── templates
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── NOTES.txt
│   ├── pvc.yaml
│   ├── secrets.yaml
│   └── svc.yaml
└── values.yaml 
```

- Chart.yaml：YAML 文件，描述 chart 的概要信息。
- README.md：Markdown 格式的 README 文件，相当于 chart 的使用文档，此文件为可选。
- LICENSE：文本文件，描述 chart 的许可信息，此文件为可选。
- requirements.yaml ：chart 可能依赖其他的 chart，这些依赖关系可通过 requirements.yaml 指定。
- values.yaml：chart 支持在安装的时根据参数进行定制化配置，而 values.yaml 则提供了这些配置参数的默认值。
- templates目录：各类 Kubernetes 资源的配置模板都放置在这里。Helm 会将 values.yaml 中的参数值注入到模板中生成标准的 YAML 配置文件。
- templates/NOTES.txt：chart 的简易使用文档，chart 安装成功后会显示此文档内容。 与模板一样，可以在 NOTE.txt 中插入配置参数，Helm 会动态注入参数值。



## 	 chart模板

Helm 通过模板创建 Kubernetes 能够理解的 YAML 格式的资源配置文件，我们将通过例子来学习如何使用模板。

以 `templates/secrets.yaml` 为例，自行查看其内容

从结构上看，文件的内容和我们在定义Secret的配置上大致相似，只是大部分的属性值变成了用双中括号包起来的xxx。这些实际上是模板的语法。Helm采用了Go语言的模板来编写chart。

- ①template "mysql.fullname" . 定义 Secret 的 name。

关键字 `template` 的作用是引用一个子模板 `mysql.fullname`。这个子模板是在 `templates/_helpers.tpl` 文件中定义的。

`templates/_helpers.tpl`定义还是很复杂的，因为它用到了模板语言中的对象、函数、流控制等概念。现在看不懂没关系，这里我们学习的重点是：如果存在一些信息多个模板都会用到，则可在 `templates/_helpers.tpl` 中将其定义为子模板，然后通过 `templates` 函数引用。

这里 `mysql.fullname` 是由 release 与 chart 二者名字拼接组成。

根据 chart 的最佳实践，所有资源的名称都应该保持一致，对于我们这个 chart，无论 Secret 还是 Deployment、PersistentVolumeClaim、Service，它们的名字都是子模板 `mysql.fullname` 的值。

- ② `Chart` 和 `Release` 是 Helm 预定义的对象，每个对象都有自己的属性，可以在模板中使用。如果使用下面命令安装 chart：

```
[root@k8s-master templates]# helm search stable/mysql
NAME            CHART VERSION   APP VERSION DESCRIPTION                                       
stable/mysql    0.3.5                       Fast, reliable, scalable, and easy to use open-...
[root@k8s-master templates]# helm install stable/mysql -n my
```

那么：

```
.Chart.Name 的值为 mysql
.Chart.Version  的值为 0.3.5
.Release.Name }} 的值为 my
.Release.Service }} 始终取值为 Tiller
template "mysql.fullname" . }} 计算结果为 `my-mysql
```



- ③ 这里指定 `mysql-root-password` 的值，不过使用了 `if-else` 的流控制，其逻辑为：

如果 `.Values.mysqlRootPassword` 有值，则对其进行 base64 编码；否则随机生成一个 10 位的字符串并编码。

`Values` 也是预定义的对象，代表的是 `values.yaml` 文件。而 `.Values.mysqlRootPassword` 则是 `values.yaml` 中定义的 `mysqlRootPassword` 参数：

```
## mysql image version
## ref: https://hub.docker.com/r/library/mysql/tags/
##
image: "mysql"
imageTag: "5.7.14"

busybox:
  image: "busybox"
  tag: "1.29.3"

testFramework:
  image: "dduportal/bats"
  tag: "0.4.0"

## Specify password for root user
##
## Default: random 10 character string
# mysqlRootPassword: testing
```

因为 `mysqlRootPassword` 被注释掉了，没有赋值，所以逻辑判断会走 `else`，即随机生成密码。

`randAlphaNum`、`b64enc`、`quote` 都是 Go 模板语言支持的函数，函数之间可以通过管道 `|` 连接。` randAlphaNum 10 | b64enc | quote ` 的作用是首先随机产生一个长度为 10 的字符串，然后将其 base64 编码，最后两边加上双引号。

`templates/secrets.yaml` 这个例子展示了 chart 模板主要的功能，我们最大的收获应该是：模板将 chart 参数化了，通过 `values.yaml` 可以灵活定制应用。

无论多复杂的应用，用户都可以用 Go 模板语言编写出 chart。无非是使用到更多的函数、对象和流控制

## 定制安装MySQL chart

- chart安装准备

作为准备工作，安装之前需要先清楚 chart 的使用方法。这些信息通常记录在 values.yaml 和 README.md 中。除了下载源文件查看，执行 `helm inspect values` 可能是更方便的方法。

```
[root@k8s-master ~]# helm inspect values stable/mysql
## mysql image version
## ref: https://hub.docker.com/r/library/mysql/tags/
##
image: "mysql"
imageTag: "5.7.14"

## Specify password for root user
##
## Default: random 10 character string
# mysqlRootPassword: testing

## Create a database user
##
# mysqlUser:
# mysqlPassword:

## Allow unauthenticated access, uncomment to enable
##
# mysqlAllowEmptyPassword: true
......
```

输出的实际上是 values.yaml 的内容。阅读注释就可以知道 MySQL chart 支持哪些参数，安装之前需要做哪些准备。其中有一部分是关于存储的：

```
## Persist data to a persistent volume
persistence:
  enabled: true
  ## database data Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  # storageClass: "-"
  accessMode: ReadWriteOnce
  size: 8Gi
```

先去stor01的/data/volume下建立db目录。

chart 定义了一个 PersistentVolumeClaim，申请 8G 的 PersistentVolume。由于我们的实验环境不支持动态供给，所以得预先创建好相应的 PV，其配置文件 `mysql-pv.yml` 内容为：

```
[root@k8s-master volumes]# cat mysql-pv.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv2
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 8Gi
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/volume/db
    server: stor01

[root@k8s-master volumes]# kubectl apply -f mysql-pv.yaml 
persistentvolume/mysql-pv2 created

[root@k8s-master volumes]# kubectl get pv
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS        CLAIM                       STORAGECLASS   REASON    AGE
mysql-pv2   8Gi        RWO            Retain           Available                                                          5s
```

- 定制化安装chart

  除了接受 values.yaml 的默认值，我们还可以定制化 chart，比如设置 `mysqlRootPassword`。

  Helm 有两种方式传递配置参数：

  1. 指定自己的 values 文件。
      通常的做法是首先通过 `helm inspect values mysql > myvalues.yaml`生成 values 文件，然后设置 `mysqlRootPassword`，之后执行 `helm install --values=myvalues.yaml mysql`。
  2. 通过 `--set` 直接传入参数值，比如：

  ```
  [root@k8s-master ~]# helm install stable/mysql --set mysqlRootPassword=abc123 -n my
  ```

`mysqlRootPassword` 设置为 `abc123`。另外，`-n` 设置 release 为 `my`，各类资源的名称即为`my-mysql`。

通过 `helm list` 和 `helm status` 可以查看 chart 的最新状态。

-  升级和回滚release

  release 发布后可以执行 `helm upgrade` 对其升级，通过 `--values` 或 `--set`应用新的配置。比如将当前的 MySQL 版本升级到 5.7.15：

  ```
  [root@k8s-master ~]# helm upgrade --set imageTag=5.7.15 my stable/mysql
  Release "my" has been upgraded. Happy Helming!
  LAST DEPLOYED: Tue Oct 30 23:42:36 2018
  ......
  [root@k8s-master ~]# kubectl get deployment my-mysql -o wide
  NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES         SELECTOR
  my-mysql   1         1         1            0           11m       my-mysql     mysql:5.7.15   app=my-mysql
  ```

  `helm history` 可以查看 release 所有的版本。通过 `helm rollback` 可以回滚到任何版本。

  ```
  [root@k8s-master ~]# helm history my
  REVISION    UPDATED                     STATUS      CHART       DESCRIPTION     
  1           Tue Oct 30 23:31:42 2018    SUPERSEDED  mysql-0.3.5 Install complete
  2           Tue Oct 30 23:42:36 2018    DEPLOYED    mysql-0.3.5 Upgrade complete
  [root@k8s-master ~]# helm rollback my 1
  Rollback was a success! Happy Helming!
  回滚成功，MySQL 恢复到 5.7.14。
  
  [root@k8s-master ~]# helm history my
  REVISION    UPDATED                     STATUS      CHART       DESCRIPTION     
  1           Tue Oct 30 23:31:42 2018    SUPERSEDED  mysql-0.3.5 Install complete
  2           Tue Oct 30 23:42:36 2018    SUPERSEDED  mysql-0.3.5 Upgrade complete
  3           Tue Oct 30 23:44:28 2018    DEPLOYED    mysql-0.3.5 Rollback to 1   
  [root@k8s-master ~]# kubectl get deployment my-mysql -o wide
  NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES         SELECTOR
  my-mysql   1         1         1            1           13m       my-mysql     mysql:5.7.14   app=my-mysql
  ```

## 自定义chart

Kubernetes 给我们提供了大量官方 chart，不过要部署微服务应用，还是需要开发自己的 chart，下面就来实践这个主题。

- 创建chart

执行 `helm create mychart` 的命令创建 chart `mychart`：

```
[root@k8s-master ~]# helm create -h
[root@k8s-master ~]# helm create mychart
Creating mychart
[root@k8s-master ~]# tree mychart/
mychart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml
```

Helm 会帮我们创建目录 `mychart`，并生成了各类 chart 文件。这样我们就可以在此基础上开发自己的 chart 了。

- 调试chart

只要是程序就会有 bug，chart 也不例外。Helm 提供了 debug 的工具：`helm lint` 和 `helm install --dry-run --debug`。

`helm lint` 会检测 chart 的语法，报告错误以及给出建议。 故意修改mychart中的value.yaml，进行检测：

`helm lint mychart` 会指出这个语法错误。

```
[root@k8s-master ~]# helm lint mychart
==> Linting mychart
[INFO] Chart.yaml: icon is recommended
[ERROR] values.yaml: unable to parse YAML
    error converting YAML to JSON: yaml: line 11: could not find expected ':'

Error: 1 chart(s) linted, 1 chart(s) failed

mychart 目录被作为参数传递给 helm lint。错误修复后则能通过检测。

[root@k8s-master ~]# helm lint mychart
==> Linting mychart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, no failures
```

`helm install --dry-run --debug` 会模拟安装 chart，并输出每个模板生成的 YAML 内容。

```
[root@k8s-master ~]# helm install --dry-run mychart --debug

......
```

我们可以检视这些输出，判断是否与预期相符。

- 安装chart

安装 chart，Helm 支持四种安装方法：

1. 安装仓库中的 chart，例如：`helm install stable/nginx`
2. 通过 tar 包安装，例如：`helm install ./nginx-1.2.3.tgz`
3. 通过 chart 本地目录安装，例如：`helm install ./nginx`
4. 通过 URL 安装，例如：`helm install https://example.com/charts/nginx-1.2.3.tgz`

这里通过使用本地目录进行安装：

```
[root@k8s-master ~]# helm install mychart
[root@k8s-master ~]# kubectl get svc
```

可获取到ClusterIP，进行访问

- 将chart添加到仓库

chart 通过测试后可以将其添加到仓库，团队其他成员就能够使用。任何 HTTP Server 都可以用作 chart 仓库，下面演示在 `k8s-node1`192.168.56.12 上搭建仓库。

```
（1）在 k8s-node1 上启动一个 httpd 容器。
[root@k8s-node01 ~]# mkdir /var/www
[root@k8s-node01 ~]# docker run -d -p 8080:80 -v /var/www/:/usr/local/apache2/htdocs/ httpd

（2）通过 helm package 将 mychart 打包。
[root@k8s-master ~]# helm package mychart
Successfully packaged chart and saved it to: /root/mychart-0.1.0.tgz

（3）执行 helm repo index 生成仓库的 index 文件
[root@k8s-master ~]# mkdir myrepo
[root@k8s-master ~]# mv mychart-0.1.0.tgz myrepo/
[root@k8s-master ~]# 
[root@k8s-master ~]# helm repo index myrepo/ --url http://192.168.56.12:8080/charts
[root@k8s-master ~]# ls myrepo/
index.yaml  mychart-0.1.0.tgz

Helm 会扫描 myrepo 目录中的所有 tgz 包并生成 index.yaml。--url指定的是新仓库的访问路径。新生成的 index.yaml 记录了当前仓库中所有 chart 的信息：
当前只有 mychart 这一个 chart。

[root@k8s-master ~]# cat myrepo/index.yaml 
apiVersion: v1
entries:
  mychart:
  - apiVersion: v1
    appVersion: "1.0"
    created: 2018-10-31T02:02:45.599264611-04:00
    description: A Helm chart for Kubernetes
    digest: 08abeb3542e8a9ab90df776d3a646199da8be0ebfc5198ef032190938d49e30a
    name: mychart
    urls:
    - http://192.168.56.12:8080/charts/mychart-0.1.0.tgz
    version: 0.1.0
generated: 2018-10-31T02:02:45.598450525-04:00

（4）将 mychart-0.1.0.tgz 和 index.yaml 上传到 k8s-node1 的 /var/www/charts 目录。
[root@k8s-master myrepo]# scp ./* root@k8s-node01:/var/www/charts/
[root@k8s-node01 ~]# ls /var/www/charts/
index.yaml  mychart-0.1.0.tgz

（5）通过 helm repo add 将新仓库添加到 Helm。
[root@k8s-master ~]# helm repo add newrepo http://192.168.56.12:8080/charts
"newrepo" has been added to your repositories
[root@k8s-master ~]# helm repo list
NAME    URL                                                   
local   http://127.0.0.1:8879/charts                          
stable  https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
newrepo http://192.168.56.12:8080/charts                      

（6）现在已经可以 repo search 到 mychart 了。
[root@k8s-master ~]# helm search mychart
NAME            CHART VERSION   APP VERSION DESCRIPTION                
local/mychart   0.1.0           1.0         A Helm chart for Kubernetes
newrepo/mychart 0.1.0           1.0         A Helm chart for Kubernetes

除了 newrepo/mychart，这里还有一个 local/mychart。这是因为在执行第 2 步打包操作的同时，mychart 也被同步到了 local 的仓库。

（7）已经可以直接从新仓库安装 mychart 了。
[root@k8s-master ~]# helm install newrepo/mychart

（8）如果以后仓库添加了新的 chart，需要用 helm repo update 更新本地的 index。
[root@k8s-master ~]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "newrepo" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 

这个操作相当于 Centos 的 yum update。
```

## 总结

- Helm是Kubernetes的包管理器，Helm 让我们能够像 yum 管理 rpm 包那样安装、部署、升级和删除容器化应用。
- Helm 由客户端和 Tiller 服务器组成。客户端负责管理 chart，服务器负责管理 release。
- chart 是 Helm 的应用打包格式，它由一组文件和目录构成。其中最重要的是模板，模板中定义了 Kubernetes 各类资源的配置信息，Helm 在部署时通过 values.yaml 实例化模板。
- Helm 允许用户开发自己的 chart，并为用户提供了调试工具。用户可以搭建自己的 chart 仓库，在团队中共享 chart。

## 参考资料

> - []()
> - []()
