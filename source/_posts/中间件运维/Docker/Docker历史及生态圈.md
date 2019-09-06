---
title: Docker历史及生态圈
toc: true
date: 2019-09-04 11:57:07
tags:
categories:
---



## 发展历史
Docker 最初是 dotCloud 公司创始人 Solomon Hykes 在法国期间发起的一个公司内部项目，它是基于 dotCloud 公司多年云服务技术的一次革新，并于 2013 年 3 月以 Apache 2.0 授权协议开源，主要项目代码在 GitHub 上进行维护。Docker 项目后来还加入了 Linux 基金会，并成立推动 开放容器联盟（OCI）。

Docker 自开源后受到广泛的关注和讨论，至今其 GitHub 项目已经超过 4 万 6 千个星标和一万多个 fork。甚至由于 Docker 项目的火爆，在 2013 年底，dotCloud 公司决定改名为Docker。Docker 最初是在 Ubuntu 12.04 上开发实现的；Red Hat 则从 RHEL 6.5 开始对Docker 进行支持；Google 也在其 PaaS 产品中广泛应用 Docker。

Docker 使用 Google 公司推出的 Go 语言 进行开发实现，基于 Linux 内核的cgroup，namespace，以及 AUFS 类的 Union FS 等技术，对进程进行封装隔离，属于 操作系统层面的虚拟化技术。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。

最初实现是基于 LXC从 0.7 版本以后开始去除 LXC，转而使用自行开发的libcontainer，从 1.11 开始，则进一步演进为使用 runC 和 containerd。

- 2013年3月：Solomon Hykes的PyCon Lightning演讲介绍了Docker
- 2013年底：dotCloud 公司决定改名为Docker
- 2014年6月：Docker Engine 1.0 GA和DockerCon 1
- 2015年6月：Docker成立开放容器联盟（OCI）以定义行业标准
- 2016年6月：发布具有集成编排功能（Orchestration）的Docker 1.12
- 2016年12月：Docker将Containerd捐赠给CNCF
- 2017年4月：Docker开源Moby项目和LinuxKit
- 2017年10月：Docker将Kubernetes集成到Docker企业版中

名词解释

- LXC: linxu内核层的容器技术实现，docker最初底层实现
- libcontainer：docker早期底层实现库
- runC: OCI的参考实现
- containerd：行业标准的核心容器运行时,基于runC，捐赠给CNCF，重点是简单性，健壮性和可移植性

资源链接

- 官网：https://www.docker.com/
- 文档：https://docs.docker.com/
- Linux基金会：https://www.linuxfoundation.org
- OCI官网：https://www.opencontainers.org/news
- LXC官网：https://linuxcontainers.org/
- CNCF基金会: https://www.cncf.io/

注: OCI和CNCF都属于Linux基金会


## Docker Registry
一个镜像仓库是指：一个镜像的多个版本存放的地址，两个镜像需要两个仓库。一个Registry是指存放仓库的中心，一个Registry 中可以包含多个仓库（ Repository ），每个仓库可以包含多个标签（ Tag ）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。

以 Ubuntu 镜像 为例， ubuntu 是仓库的名字，其内包含有不同的版本标签，如， 14.04 ,16.04 。我们可以通过 ubuntu:14.04 ，或者 ubuntu:16.04来具体指定所需哪个版本的镜像。如果忽略了标签，比如 ubuntu ，那将视为 ubuntu:latest 。

仓库名经常以 两段式路径 形式出现，比如 jwilder/nginx-proxy ，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 Docker Registry 的软件或服务。

- 公开 Docker Registry 

Docker Registry 公开服务是开放给用户使用、允许用户管理镜像的 Registry 服务。一般这类公开服务允许用户免费上传、下载公开的镜像，并可能提供收费服务供用户管理私有镜像。

由于某些原因，在国内访问这些服务可能会比较慢。国内的一些云服务商提供了针对 Docker Hub 的镜像服务（ Registry Mirror ），这些镜像服务被称为加速器。常见的有 阿里云加速器、DaoCloud 加速器 等。使用加速器会直接从国内的地址下载 Docker Hub 的镜像，比直接从 Docker Hub 下载速度会提高很多。

- 国际常用Registry

1. https://hub.docker.com/   Docker引擎默认的 Registry
2. https://quay.io/ CoreOS 相关的镜像存储在这里
3. https://gcr.io/google_containers   Kubernetes 的镜像使用的就是这个服务

- 国内常用Registry

1. 时速云镜像仓库
2. 网易云镜像服务
3. DaoCloud 镜像市场
4. 阿里云镜像库

- 搭建私有Docker Registry

除了使用公开服务外，用户还可以在本地搭建私有 Docker Registry。提供私有 Docker Registry服务的软件需实现Docker Registry API。

1. Docker 官方提供了Docker Registry 镜像，可以直接使用做为私有 Registry 服务。
2. VMWare Harbor: https://goharbor.io
3. Sonatype Nexus: https://www.sonatype.com



## Moby
一个致力于推进软件容器化运动的开源项目。

在DockerCon17上，Docker发布了两个新的开源项目LinuxKit和Moby。而原来在Github上托管的docker也随着PR #32691的合入正式变为Moby。

- 官网：https://mobyproject.org/
- github: https://github.com/moby/moby

可以说Moby是Docker之母。通过Moby可以将定制化的组件打包做成一个Docker，而Moby就相当于一个组件仓库与Docker工厂。Docker公司的这一举动可以说也是顺应了潮流，将Docker与操作系统解耦，并且将其以组件组装的形式模块化。可以说今后，操作系统也是Docker容器的一个基础组件。而Moby项目既是一个组件库又是一个框架，为今后组件制作与组装提供了一套规范。，它提供了一套组件的“乐高积木”，一个将组件装配成定制化基于容器的系统的框架和一个所有容器发烧友试验与交流想法的场所。把Moby想象成容器系统的“乐高俱乐部”吧。

Moby由以下几部分组成：

1. 一个容器化的后端组件库（例如：一个低层builder，日志功能、数据卷管理、网络、镜像管理、containered、SwarmKit）
1. 一个可以将这些组件装配成独立的容器平台并协助构建、测试、部署构件的框架
1. 一个叫Moby Origin的参考——它是Docker容器平台的开放基础，同时也是使用来自Moby库或是其它项目的组件的容器系统样例。

Moby是为系统构建者——想构建自己的基于容器的系统的人设计的，而不是使用Docker或者其他容器平台的开发者。Moby项目的参与者可以选择派生自Docker的组件库，也可以“bring your own components（自带组件）” (BYOC)，打包成可以混合搭配其他组件的容器来创建定制化的容器系统。

Docker使用Moby项目作为开放的研发实验室，用于试验、开发新组件并与生态圈就未来容器技术进行合作。我们所有的开源合作都会迁移到Moby项目。Docker现在、将来都一直会是一个可以让你构建、传播、运行容器的开源项目。从用户角度来说，一切都没有发生变化。用户可以继续在docker.com上下载Docker。

Moby本身的Logo是一头鲸鱼的尾巴，我认为这个Logo的含义便是：你只看到了那会是Docker大鲸鱼，但是到底会是一头怎样的鲸鱼，便要通过Moby孕育才能看完整了。

## CoreOS
CoreOS是一个基于Linux 内核的轻量级操作系统，为了计算机集群的基础设施建设而生，专注于自动化，轻松部署，安全，可靠，规模化。

- 官网： https://coreos.com/

CoreOS 内置了由自己开发的容器 Rkt和docker，另外还有服务发现组件etcd，etcd是以rkt 容器方式运行。

CoreOS 可以在一个机器上很好地运行，但是它被设计用来搭建集群。可以通过 k8s 很容易得使应用容器部署在多台机器上并且通过服务发现把他们连接
在一起。

## etcd
etcd 是 CoreOS 团队于 2013 年 6 月发起的开源项目，它的目标是构建一个高可用的分布式键值（ key-value ）数据库，基于 Go 语言实现。

- github: https://github.com/etcd-io/etcd

受到 Apache ZooKeeper 项目和 doozer 项目的启发，前者是一套知名的分布式系统中进行同步和一致性管理的工具，后者是一个一致性分布式数据库。

etcd 在设计的时候重点考虑了下面四个要素：
- 简单：具有定义良好、面向用户的 API (gRPC)
- 安全：支持 HTTPS 方式的访问
- 快速：支持并发 10 k/s 的写操作
- 可靠：支持分布式结构，基于 Raft 的一致性算法

Raft 是一套通过选举主节点来实现分布式系统一致性的算法，相比于大名鼎鼎的
Paxos 算法，它的过程更容易被人理解，由 Stanford 大学的 Diego Ongaro 和
John Ousterhout 提出。更多细节可以参考 https://raft.github.io。

一般情况下，用户使用 etcd 可以在多个节点上启动多个实例，并添加它们为一个集群。同一个集群中的 etcd 实例将会保持彼此信息的一致性。

默认 2379 端口处理客户端的请求， 2380 端口用于集群各成员间的通信。


## Kubernetes
Kubernetes 是 Google 团队发起的开源项目，它的目标是管理跨多个主机的容器，提供基本的部署，维护以及运用伸缩，主要实现语言为 Go 语言。

- 官网： https://kubernetes.io/
- github: https://github.com/kubernetes/kubernetes


## Mesos
Mesos 项目是源自 UC Berkeley 的对集群资源进行抽象和管理的开源项目，Mesos 项目主要由 C++ 语言编写。

Mesos 最初由 UC Berkeley 的 AMP 实验室于 2009 年发起，遵循 Apache 协议，目前已经成立了 Mesosphere 公司进行运营。Mesos 可以将整个数据中心的资源（包括 CPU、内存、存储、网络等）进行抽象和调度，使得多个应用同时运行在集群中分享资源，并无需关心资源的物理分布情况。如果把数据中心中的集群资源看做一台服务器，那么 Mesos 要做的事情，其实就是今天操作系统内核的职责：抽象资源 + 调度任务。Mesos 项目是 Mesosphere公司 Datacenter Operating System (DCOS) 产品的核心部件。

- Mesosphere官网：https://mesosphere.com/
- mesos项目官网：http://mesos.apache.org/
- github: https://github.com/apache/mesos

## Busybox
BusyBox 是一个集成了一百多个最常用 Linux 命令和工具（如 cat、echo、grep、mount、telnet 等）的精简工具箱，它只需要几 MB 的大小，很方便进行各种快速验证，被誉为“Linux 系统的瑞士军刀”。BusyBox 可运行于多款 POSIX 环境的操作系统中，如 Linux（包括 Android）、Hurd、FreeBSD 等。

- 官网：https://busybox.net/ 
- 官方仓库：https://git.busybox.net/busybox/ 
- 官方镜像：https://hub.docker.com/_/busybox/

## Alpine
Alpine 操作系统是一个面向安全的轻型 Linux 发行版。它不同于通常
Linux 发行版， Alpine 采用了 musl libc 和 busybox 以减小系统的体积
和运行时资源消耗，但功能上比 busybox 又完善的多，因此得到开源社区越来越
多的青睐。在保持瘦身的同时， Alpine 还提供了自己的包管理工具 apk ，可
以通过 https://pkgs.alpinelinux.org/packages 网站上查询包信息，也可以
直接通过 apk 命令直接查询和安装各种软件。

Alpine 由非商业组织维护的，支持广泛场景的 Linux 发行版，它特别为资深/
重度 Linux 用户而优化，关注安全，性能和资源效能。 Alpine 镜像可以适用于
更多常用场景，并且是一个优秀的可以适用于生产的基础系统/环境。

Alpine Docker 镜像也继承了 Alpine Linux 发行版的这些优势。相比于其他
Docker 镜像，它的容量非常小，仅仅只有 5 MB 左右（对比 Ubuntu 系列镜像接
近 200 MB），且拥有非常友好的包管理机制。官方镜像来自 docker-alpine 项
目。

目前 Docker 官方已开始推荐使用 Alpine 替代之前的 Ubuntu 做为基础镜像环
境。这样会带来多个好处。包括镜像下载速度加快，镜像安全性提高，主机之间的
切换更方便，占用更少磁盘空间等。

下表是官方镜像的大小比较：
```
REPOSITORY TAG IMAGE ID VIRTUAL SIZE
alpine latest 4e38e38c8ce0 4.799 MB
debian latest 4d6ce913b130 84.98 MB
ubuntu latest b39b81afc8ca 188.3 MB
centos latest 8efe422e6104 210 MB
```
目前，大部分 Docker 官方镜像都已经支持 Alpine 作为基础镜像，可以很容易进行迁移。

例如：

```
ubuntu/debian -> alpine
python:2.7 -> python:2.7-alpine
ruby:2.3 -> ruby:2.3-alpine
```

另外，如果使用 Alpine 镜像替换 Ubuntu 基础镜像，安装软件包时需要用apk 包管理器替换 apt 工具，如

```
$ apk add --no-cache <package>
```
Alpine 中软件安装包的名字可能会与其他发行版有所不同，可以在
https://pkgs.alpinelinux.org/packages 网站搜索并确定安装包名称。如果
需要的安装包不在主索引内，但是在测试或社区索引中。那么可以按照以下方法使
用这些安装包。


```
$ echo "http://dl-4.alpinelinux.org/alpine/edge/testing" >> /etc
/apk/repositories
$ apk --update add --no-cache <package>
```
- 官网：http://alpinelinux.org/ 
- 官方仓库：https://github.com/alpinelinux
- 官方镜像：https://hub.docker.com/_/alpine

## LinuxKit
LinuxKit 这个工具可以将多个 Docker 镜像组成一个最小化、可自由定制的Linux 系统，最后的生成的系统只有几十 M 大小，可以很方便的在云端进行部署。

## Drone

基于 Docker 的 CI/CD 工具 Drone 所有编译、测试的流程都在 Docker 容器中进行。

开发者只需在项目中包含 .drone.yml 文件，将代码推送到 git 仓库， Drone就能够自动化的进行编译、测试、发布。

- 官网： https://drone.io/
- github: https://github.com/drone/drone


## Travis CI
一个在线CI服务，与Github结合比较好。

当代码提交到 GitHub 时，Travis CI 会根据项目根目录 .travis.yml 文件设置的指令，执行一系列操作。

- 官网：https://travis-ci.org/
- github: https://github.com/travis-ci/travis-ci
- 控制台：https://travis-ci.com/



## 参考资料
> - []()
> - []()
