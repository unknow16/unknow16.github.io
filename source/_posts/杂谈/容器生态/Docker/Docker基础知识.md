---
title: Docker基础知识
toc: true
date: 2020-04-16 12:56:49
tags:
categories:
---



## 镜像和容器

镜像是容器的基础，镜像是多层存储的，每一层在前一层的基础上进行的修改，容器同样也是多层存储的，是以镜像为基础层，在其基础上加一层作为容器运行时的存储层。



## Docker Registry

一个镜像仓库是指：一个镜像的多个版本存放的地址，两个镜像需要两个仓库。

一个Registry是指：存放仓库的中心，一个Registry 中可以包含多个仓库（ Repository ），每个仓库可以包含多个标签（ Tag ）；每个标签对应一个镜像。

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



## 制作镜像

制作镜像有两种方法：

1. 使用docker commit命令（不推荐）
2. 编写Dockerfile脚本文件

如下介绍使用Dockerfile定制镜像。定制镜像实际上就是定制镜像的每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本文件，用这个脚步来构建、定制镜像，这个脚本就是Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的指令，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

如下是一个定制nginx镜像并修改首页内容的Dockerfile文件内容：

```
FROM nginx
RUN echo '<h1>Hello world!!!</h1>' > /usr/share/nginx/html/index.html
```

- FORM: 指定基础镜像

该指令是Dockerfile必备的，且是第一条指令，指定以什么镜像为基础镜像进行定制。

1. 如服务类镜像：nginx、redis、mongo、mysql、php、tomcat等
2. 如语言环境类镜像：node、openjdk、python、ruby、golang等。
3. 如果没有合适的可以从操作系统定制，如操作系统镜像：ubuntu、debian、centos、fedora、alpine等。
4. Docker提供了一个特殊镜像，名为scratch。其实这是空白的镜像，不以任何系统为基础，直接将可执行文件复制进镜像执行，如：coreos/etcd用这个镜像会让镜像体积更加小。使用Go语言开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为Go是特别适合容器微服务架构的语言的原因之一。

- RUN: 执行命令

该指令是用来执行命令行命令的。其有两种格式：

1. shell格式：RUN <命令>，就像直接在命令行中输入的命令一样。如上例中一样就是。
2. exec格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。 

- 执行构建

```
//构建镜像格式
docker build [选项] <上下文路径/URL/>


// 以上文中的Dockerfile来构建
// 末尾有一个.   指的是上下文路径，还可以是一个url
// -t 指tag
docker build -t nginx:v3 .
```

一般来说，应该将Dockerfile置于一个空目录下，将所需文件复制一份过来，也可以向.gitignore一样写一个.dockerignore忽略上下文中不需传给你Docker引擎的文件。

如果不额外指定Dockerfile, 默认情况下，会将上下文目录下的名为Dockerfile的文件作为Dockerfile， 这只是默认行为，可通过-f ../Dockerfile.txt参数来指定某个文件作为Dockerfile，但一般习惯性采用Dockerfile文件名，且将其放置在镜像构建上下文目录中。

- 镜像构建上下文

如果注意，会看到 docker build 命令最后有一个 . 。 . 表示当前目录，而Dockerfile 就在当前目录，因此不少初学者以为这个路径是在指定Dockerfile 所在路径，这么理解其实是不准确的。如果对应上面的命令格式，你可能会发现，这是在指定上下文路径。那么什么是上下文呢？

使用Dockerfile构建镜像时，在docker build命令中需指定一个镜像构建上下文。

Docker在运行时分为Docker引擎（也就是服务端守护进程）和客户端工具。Docker引擎提供了一组REST API，被称为Docker Remote API, 而docker命令这样的客户端工具，则是通过这组API与Docker引擎交互，从而完成各种功能。因此表面上看像是在本地完成各种docker功能，但实际上是通过RPC调用Docker引擎完成的。这种C/S设计能使远程操作Docker服务端。

docker build 构建镜像其实也不是在本地构建，而是在Docker服务端引擎中构建，构建时，并非所有定制都是通过RUN指令完成的，有时需将本地文件复制进镜像，如通过COPY、ADD指令等。此时引入了上下文的概念，docker build命令得知上下文路径后，会将该路径下的所有内容打包，然后上传给Docker引擎。Docker引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。如下命令

```
COPY ./package.json  /app/
```

这不是复制执行docker build命令所在目录下的package.json,也不是复制Dockerfile所在目录下的package.json，而是复制上下文目录下的package.json。



## 网络配置

- 指定端口映射

```
-P(大写) : Docker 会随机映射一个 49000~49900 的端口到内部容器开放的网络端口
-p(小写字母) : 具体指定，可以多次使用来绑定多个端口，有如下三种格式

1. ip:hostPort:containerPort： 映射到指定地址的指定端口
2. ip::containerPort： 映射到指定地址的任意端口
3. hostPort:containerPort： 映射所有接口地址

# 指定udp端口
docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
```



## 数据卷

在Docker容器中管理数据主要有两种方式：数据卷和挂载主机目录 。

- 数据卷

数据卷是一个可供一个或多个容器使用的特殊目录，数据卷的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示看的是挂载的数据卷 。它绕过 UFS， 可以提供很多有用的特性：

1. 数据卷 可以在容器之间共享和重用
2. 对 数据卷 的修改会立马生效
3. 对 数据卷 的更新，不会影响镜像
4. 数据卷 默认会一直存在，即使容器被删除

```
// 创建一个数据卷
docker volume create my-vol

// 查看所有的 数据卷
docker volume ls

// 查看指定 数据卷 的信息
docker volume inspect my-vol

// 查看某容器的数据卷信息,数据卷 信息在 "Mounts" Key 下面
docker inspect <container name>
```

在用 docker run 命令的时候，使用 --mount 标记来将 数据卷 挂载到容器里。在一次 docker run中可以挂载多个 数据卷 。下面创建一个名为 web 的容器，并加载一个 数据卷 到容器的 /webapp 目录:

```
docker run -d -P \
--name web \
--mount source=my-vol,target=/webapp \
training/webapp \
python app.py
```

数据卷 是被设计用来持久化数据的，它的生命周期独立于容器，Docker 不会在容器被删除后自动删除数据卷 ，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的 数据卷 。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用 docker rm -v 这个命令，或者如下方式删除：

```
// 删除指定数据卷
docker volume rm my-vol

// 清除无主的数据卷
docker volume prune
```

- 挂载主机目录

```
// 使用--mount参数挂载一个主机目录作为数据卷
// 也可以挂载一个本地主机文件作为数据卷
docker run -d -P \
    --name web \
    --mount type=bind,source=/src/webapp,target=/opt/webapp \
    training/webapp \
    python app.py
```

上面的命令加载主机的 /src/webapp 目录到容器的 /opt/webapp 目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径，以前使用 -v 参数时如果本地目录不存在 Docker 会自动为你创建一个文件夹，现在使用 --mount 参数时如果本地目录不存在，Docker 会报错。

Docker 挂载主机目录的默认权限是读写 ，用户也可以通过增加 readonly 指定为 只读 。

```
docker run -d -P \
    --name web \
    # -v /src/webapp:/opt/webapp:ro \
    --mount type=bind,source=/src/webapp,target=/opt/webapp,readonly \
    training/webapp \
    python app.py
```





## Docker三剑客

Docker三剑客分别是：Docker Compose、Docker Machine、Docker Swarm

- Docker Compose

我们知道使用一个 Dockerfile 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

Compose 恰好满足了这样的需求。它允许用户通过一个单独的 docker-compose.yml 模板文件，来定义一组相关联的应用容器为一个项目（project）。

该项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。其定位是 「定义和运行多个 Docker 容器的应用（Defining and running multi-container Docker applications）」，其前身是开源项目 Fig。项目由 Python 编写，实现上调用了 Docker 服务提供的 API来对容器进行管理。因此，只要所操作的平台支持Docker API，就可以在其上利用Compose 来进行编排管理。

主要有服务和项目的两个概念，服务 ( service)指一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。项目 ( project)由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml 文件中定义。默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

Compose 可以通过 Python 的包管理工具 pip 进行安装，也可以直接下载编译好的二进制文件使用，甚至能够直接在 Docker 容器中运行。

```
# 选择相应版本，执行命令即可
https://github.com/docker/compose/releases
```

- Docker Swarm

Docker 官方编排项目之一，提供 Docker 容器集群服务，是Docker 官方对容器云生态进行支持的核心方案。使用它，用户可以将多个 Docker主机封装为单个大型的虚拟 Docker 主机，快速打造一套容器云平台。Docker 1.12.0+ 将Swarm功能已经内嵌入 Docker 引擎，成为了 docker 子命令 docker swarm，称为Swarm mode。

Swarm mode 内置 kv存储功能，提供了众多的新特性，比如：具有容错能力的去中心化设计、内置服务发现、负载均衡、路由网格、动态伸缩、滚动更新、安全传输等。使得 Docker 原生的 Swarm 集群具备与 Mesos、Kubernetes 竞争的实力。

- Docker Machine（基本停更）

Docker 官方编排项目之一，负责在多种台上快速安装 Docker 环境，即创建一个包含docker环境的虚拟机，支持多种后端驱动，包括虚拟机、本地主机和云平台等。项目基于 Go 语言实现。

## 参考资料
> - []()
> - []()
