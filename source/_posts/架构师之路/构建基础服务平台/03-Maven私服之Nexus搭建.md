---
title: 03-Maven私服之Nexus搭建
toc: true
date: 2019-06-28 22:18:57
tags:
categories:
---



## 采用版本

Nexus3.x支持Docker、Maven、Yum、PyPI等私有仓库。

默认访问地址：<http://ip:8081>

默认账号密码：admin/admin123

默认登录是游客身份：anonymous，可以查看仓库并下载依赖，但不能配置nexus

##  Linux安装

1. 官网下载链接  <https://www.sonatype.com/download-oss-sonatype>
2. 下载解压文件后：配置bin/nexus.vmoptions文件，适当调整内存参数，防止占用内存太大
3. etc目录下nexus-default.properties文件可配置默认端口和host及访问根目录
4. bin目录下执行sh nexus start启动服务，sh nexus stop停止服务

## Docker安装

前提要docker服务已安装

```
docker run -d --name nexus3 --restart=always -p 8081:8081 --mount src=nexus-data,target=/nexus-data sonatype/nexus3
```

## 常见用法

1. Blob Stores

依赖index存储目录，默认存储在default下：\sonatype-work\nexus3\blobs\default；

也可以自己新建一个目录专门存储某个库的索引，后面创建repository时可以选择；

另外，下载或上传到nexus3.10中的jar是被加密存储在\sonatype-work\nexus3\db下



2. repository

group：是一个仓库组，可包含hosted和proxy两种库，下载时按配置顺序从中依次查找

hosted：一般部署我们自己项目打包的jar，供内部人员下载

proxy：代理远程仓库，maven-central是默认maven中央仓库代理，maven-aliyun是后加的阿里远程库代理，通过代理从远程库中下载的依赖都会被缓存到nexus的proxy库中

创建hosted类型仓库时，需要将Deployment policy设为Allow redeploy，否则无法部署jar

创建hosted和proxy库是需要指定Version policy：

- release：专用于部署发布版本的jar
- snapshot：专用于部署快照版本的jar，jar都是以-SNAPSHOT结尾，pom中version需以SNAPSHOT(必须大写)结尾
- mixed：可包含release和snapshot版本



## 参考资料
> - []()
> - []()
