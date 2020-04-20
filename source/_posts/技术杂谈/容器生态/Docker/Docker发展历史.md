---
title: Docker发展历史
toc: true
date: 2019-05-04 11:57:07
tags:
categories:
---



Docker 最初是 dotCloud 公司创始人 Solomon Hykes 在法国期间发起的一个公司内部项目，它是基于 dotCloud 公司多年云服务技术的一次革新，并于 2013 年 3 月以 Apache 2.0 授权协议开源，主要项目代码在 GitHub 上进行维护。Docker 项目后来还加入了 Linux 基金会，并成立推动开放容器联盟（OCI）。

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

## 名词解释

- LXC: linxu内核层的容器技术实现，docker最初底层实现
- libcontainer：docker早期底层实现库
- runC: OCI的参考实现
- containerd：行业标准的核心容器运行时,基于runC，捐赠给CNCF，重点是简单性，健壮性和可移植性

## 资源链接

- 官网：https://www.docker.com/
- 文档：https://docs.docker.com/
- Linux基金会：https://www.linuxfoundation.org
- OCI官网：https://www.opencontainers.org/news
- LXC官网：https://linuxcontainers.org/
- CNCF基金会: https://www.cncf.io/

注: OCI和CNCF都属于Linux基金会



## 参考资料
> - []()
> - []()
