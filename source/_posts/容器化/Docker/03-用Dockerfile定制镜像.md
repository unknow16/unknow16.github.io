---
title: 03-用Dockerfile定制镜像
toc: true
date: 2019-04-27 14:46:07
tags:
categories:
---





## 定制镜像
定制镜像有两种方法：
1. 使用docker commit命令（不推荐）
2. 编写Dockerfile脚本文件

镜像是容器的基础，镜像是多层存储的，每一层在前一层的基础上进行的修改，容器同样也是多层存储的，是以镜像为基础层，在其基础上加一层作为容器运行时的存储层。


## 使用Dockerfile定制镜像
定制镜像实际上就是定制镜像的每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本文件，用这个脚步来构建、定制镜像，这个脚本就是Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的指令，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

如下是一个定制nginx镜像并修改首页内容的Dockerfile文件内容：

```
FROM nginx
RUN echo '<h1>Hello world!!!</h1>' > /usr/share/nginx/html/index.html
```

- FORM: 指定基础镜像

该指令是Dockerfile必备的，且是第一条指令，指定以什么镜像为基础镜像进行定制。

1. 如服务类镜像：nginx、redis、mongo、mysql、php、tomcat等
1. 如语言环境类镜像：node、openjdk、python、ruby、golang等。
1. 如果没有合适的可以从操作系统定制，如操作系统镜像：ubuntu、debian、centos、fedora、alpine等。
1. Docker提供了一个特殊镜像，名为scratch。其实这是空白的镜像，不以任何系统为基础，直接将可执行文件复制进镜像执行，如：swarm、coreos/etcd,用这个镜像会让镜像体积更加小。使用Go语言开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为Go是特别适合容器微服务架构的语言的原因之一。

- RUN: 执行命令

该指令是用来执行命令行命令的。其有两种格式：
1. shell格式：RUN <命令>，就像直接在命令行中输入的命令一样。如上例中一样就是。
2. exec格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。 


- 执行构建

```
//构建镜像格式
docker build [选项] <上下文路径 / URL/>


// 以上文中的Dockerfile来构建
// 末尾有一个.   指的是上下文路径，还可以是一个url
// -t 指tag
docker build -t nginx:v3 .
```
一般来说，应该将Dockerfile置于一个空目录下，将所需文件复制一份过来，也可以向.gitignore一样写一个.dockerignore忽略上下文中不需传给你Docker引擎的文件。

如果不额外指定Dockerfile, 默认情况下，会将上下文目录下的名为Dockerfile的文件作为Dockerfile， 这只是默认行为，可通过-f ../Dockerfile.txt参数来指定某个文件作为Dockerfile，但一般习惯性采用Dockerfile文件名，且将其放置在镜像构建上下文目录中。

还可以使用以下方式构建：
1. 直接用 Git repo 进行构建
2. 用给定的 tar 压缩包构建
3. 从标准输入中读取 Dockerfile 进行构建
4. 从标准输入中读取上下文压缩包进行构建

## 镜像构建上下文
如果注意，会看到 docker build 命令最后有一个 . 。 . 表示当前目录，而
Dockerfile 就在当前目录，因此不少初学者以为这个路径是在指定
Dockerfile 所在路径，这么理解其实是不准确的。如果对应上面的命令格式，
你可能会发现，这是在指定上下文路径。那么什么是上下文呢？

使用Dockerfile构建镜像时，在docker build命令中需指定一个镜像构建上下文。

Docker在运行时分为Docker引擎（也就是服务端守护进程）和客户端工具。Docker引擎提供了一组REST API，被称为Docker Remote API, 而docker命令这样的客户端工具，则是通过这组API与Docker引擎交互，从而完成各种功能。因此表面上看像是在本地完成各种docker功能，但实际上是通过RPC调用Docker引擎完成的。这种C/S设计能使远程操作Docker服务端。

docker build 构建镜像其实也不是在本地构建，而是在Docker服务端引擎中构建，构建时，并非所有定制都是通过RUN指令完成的，有时需将本地文件复制进镜像，如通过COPY、ADD指令等。

此时引入了上下文的概念，docker build命令得知上下文路径后，会将该路径下的所有内容打包，然后上传给Docker引擎。Docker引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

如：COPY ./package.json  /app/

这不是复制执行docker build命令所在目录下的package.json,也不是复制dockerfile所在目录下的package.json，而是复制上下文目录下的package.json.

## 多阶段构建
在 Docker 17.05 版本之前，我们构建 Docker 镜像时，通常会采用两种方式：

1. 全部放入一个 Dockerfile

一种方式是将所有的构建过程编包含在一个 Dockerfile 中，包括项目及其依赖库的编译、测试、打包等流程，这里可能会带来的一些问题：
- 镜像层次多，镜像体积较大，部署时间变长
- 源代码存在泄露的风险

2. 分散到多个 Dockerfile

另一种方式，就是我们事先在一个 Dockerfile 将项目及其依赖库编译测试打包好后，再将其拷贝到运行环境中，这种方式需要我们编写两个 Dockerfile 和一些编译脚本才能将其两个阶段自动整合起来，这种方式虽然可以很好地规避第一种方式存在的风险，但明显部署过程较复杂。

为解决以上问题，Docker v17.05 开始支持多阶段构建 ( multistage builds )。使用多阶段构建我们就可以很容易解决前面提到的问题，并且只需要编写一个Dockerfile ，可以使用 as 来为某一阶段命名，构建时也可以从其他镜像复制文件等。