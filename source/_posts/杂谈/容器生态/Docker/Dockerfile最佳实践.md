---
title: Dockerfile最佳实践
toc: true
date: 2020-04-16 13:40:07
tags:
categories:
---



<https://docs.docker.com/develop/develop-images/dockerfile_best-practices/>





## 使用 **.dockerignore** 文件

使用 Dockerfile 构建镜像时最好是将 Dockerfile 放置在一个新建的空目录下。然后将构建镜像所需要的文件添加到该目录中。为了提高构建镜像的效率，你可以在目录下新建一个.dockerignore 文件来指定要忽略的文件和目录。 .dockerignore 文件的排除模式语法和 Git的 .gitignore 文件相似。 

## 参考资料
> - []()
> - []()
