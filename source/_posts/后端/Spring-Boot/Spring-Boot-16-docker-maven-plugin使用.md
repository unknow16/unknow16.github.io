---
title: Spring Boot-16-docker-maven-plugin使用
date: 2018-07-24 17:13:26
tags: Spring Boot
---

## 场景
使用Spring Boot编写了一个微服务后，如何将构建应用并打包成docker镜像，推送到docker仓库，以供测试环境测试。

docker-maven-plugin插件可以完成这项任务，关于其各种配置其Github上说的很清楚。

Github地址：https://github.com/spotify/docker-maven-plugin

## 配置方式
1. 通过Dockerfile文件
1. 直接在pom.xml配置

第二种方式只支持一些简单的Dockerfile命令，并没有通过Dockerfile文件配置强大。

默认情况下，该插件通过访问localhost:2375来连接本地docker，可以通过设置DOCKER_HOST 环境变量来连接其他Docker进程。

```
DOCKER_HOST=tcp://<host>:2375
```

如：可在win下开发，连接linux下的docker进程构建镜像。

##  直接在pom.xml配置
可参考Github上的Readme

## 使用Dockerfile
为了使用Dockerfile,必须在pom的文件中通过dockerDirectory来指明Dockerfile文件的所在目录。

如果配置了dockerDirectory后，其它的baseImage,maintainer,cmd和entryPoint配置将忽略。

## 构建命令
注意：构建的镜像名称不要含有SNAPSHOT，因为镜像名称只允许 [a-z0-9-_.]
- 生成一个镜像
```
mvn clean package docker:build
```

- 删除一个名称为foobar的镜像

```
mvn docker:removeImage -DimageName=foobar
```

## 其他特性
- 将生成的镜像推送到镜像注册中心
- 推送制定tags 的镜像
- 绑定docker命令到maven phases
- 使用私有镜像中心
- 推送到镜像中心需要认证，支持三种认证方式

https://www.jianshu.com/p/3b91b8958c3e