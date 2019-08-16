---
title: 07-Docker三剑客
toc: true
date: 2019-04-28 19:30:16
tags:
categories:
---


Docker三剑客分别是：

1. Docker Compose
1. Docker Machine
1. Docker Swarm

## Docker Compose
该项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。

其定位是 「定义和运行多个 Docker 容器的应用（Defining and running
multi-container Docker applications）」，其前身是开源项目 Fig。

我们知道使用一个 Dockerfile 模板文件，可以让用户
很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容
器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

Compose 恰好满足了这样的需求。它允许用户通过一个单独的 docker-compose.yml 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目
（project）。

#### 主要概念
- 服务 ( service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 ( project)：由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml 文件中定义。

Compose 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

Compose 项目由 Python 编写，实现上调用了 Docker 服务提供的 API来对容器进行管理。因此，只要所操作的平台支持Docker API，就可以在其上利用Compose 来进行编排管理。

#### 安装

Compose 可以通过 Python 的包管理工具 pip 进行安装，也可以直接下载编译
好的二进制文件使用，甚至能够直接在 Docker 容器中运行。

```
// 选择相应版本，执行命令即可
https://github.com/docker/compose/releases
```

## Docker Machine
Docker Machine 是 Docker 官方编排（Orchestration）项目之一，负责在多种台上快速安装 Docker 环境，即创建一个包含docker环境的虚拟机，支持多种后端驱动，包括虚拟机、本地主机和云平台等。

Docker Machine 项目基于 Go 语言实现，目前在 Github 上进行维护。

#### 安装
从 官方 GitHub Release 处直接下载编译好的二进
制文件即可

```
// 选择相应版本，执行命令即可
https://github.com/docker/machine/releases
```

## Docker Swarm
Docker Swarm 是 Docker 官方三剑客项目之一，提供 Docker 容器集群服务，是Docker 官方对容器云生态进行支持的核心方案。

使用它，用户可以将多个 Docker主机封装为单个大型的虚拟 Docker 主机，快速打造一套容器云平台。

注意：Docker 1.12.0+ Swarm mode 已经内嵌入 Docker 引擎，成为了 docker 子命令 docker swarm ，绝大多数用户已经开始使用 Swarm mode ，Docker 引擎API 已经删除 Docker Swarm。为避免大家混淆旧的 Docker Swarm 与新的Swarm mode ，旧的 Docker Swarm 内容已经删除，请查看 Swarm mode 一节。

## Swarm mode
Docker 1.12 Swarm mode 已经内嵌入 Docker 引擎，成为了 docker 子命令docker swarm 。请注意与旧的 Docker Swarm 区分开来。

Swarm mode 内置 kv存储功能，提供了众多的新特性，比如：具有容错能力的去中心化设计、内置服务发现、负载均衡、路由网格、动态伸缩、滚动更新、安全传输等。使得 Docker 原生的 Swarm 集群具备与 Mesos、Kubernetes 竞争的实力。