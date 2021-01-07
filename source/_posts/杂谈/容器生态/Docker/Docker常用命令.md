---
title: Docker常用命令
toc: true
date: 2020-04-16 11:59:54
tags:
categories:
---



## 镜像相关

```
# 完整命令格式，[]的都可以省略
docker pull [选项] [docker-registry-ip:port/] <用户名>/<软件名> [:标签]

# 从DockerHub中下载library用户的ubuntu软件的16.04版本的镜像
docker pull ubuntu:16.04

# 从DockerHub中拉取lizhenliang用户的kubernetes-dashboard-amd64软件的v1.10.1版本镜像
docker pull lizhenliang/kubernetes-dashboard-amd64:v1.10.1

# 显示列表中分别为仓库名、标签、镜像ID、创建时间、所占用空间
docker image ls

# 显示所有镜像，包括中间层镜像，不加-a参数，只会显示顶级镜像
docker image ls -a

# 显示镜像摘要hash
docker image ls --digests

# 查看本地镜像、容器、数据卷所占用的空间
docker system df

# 清理虚悬镜像
docker image prune

# 删除所有仓库名为redis的镜像
docker image rm $(docker image ls -q redis)
```

## 容器相关

```
# docker run 为运行容器的命令，-d参数来实现后台运行
# -it其实是两个参数，-i表示启动交互操作，-t表示采用终端
# --rm 让容器退出后即被删除。默认退出的容器不会被删除，除非主动用docker rm删除。
# ubuntu:16.04 指采用ubuntu:16.04为镜像启动容器
# bash 启动一个shell来执行命令
docker run -it --rm ubuntu:16.04 bash

# 查看容器
docker container ls -a

# 查看某容器的详细信息
docker inspect <container name>

# 查看日志
docker logs [container ID or NAMES]

# 进入容器
docker exec -it <container ID or NAMES> bash

# 清理所有处于终止状态的容器
docker container prune
```

## 数据卷

```
# 查看所有的 数据卷
docker volume ls

# 查看指定 数据卷 的信息
docker volume inspect my-vol

# 删除指定数据卷
docker volume rm my-vol

# 清除无主的数据卷
docker volume prune
```

## 网络相关

```
# 查看端口映射
docker port <container name>
```





## 参考资料
> - []()
> - []()
