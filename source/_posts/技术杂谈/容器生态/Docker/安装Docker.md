---
title: 安装Docker
toc: true
date: 2019-04-27 14:40:54
tags:
categories:
---

## 前言
Docker从1.13版本之后采用时间线的方式作为版本号，分为社区版CE和企业版EE。

社区版是免费提供给个人开发者和小型团体使用的，企业版会提供额外的收费服务，比如经过官方测试认证过的基础设施、容器、插件等。

社区版按照stable和edge两种方式发布，每个季度更新stable版本，如17.06，17.09；每个月份更新edge版本，如17.09，17.10。


## CentOS-7安装
1. Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看前提条件来验证你的CentOS 版本是否支持 Docker 。


```
# 通过 uname -r 命令查看你当前的内核版本
$ uname -r
```

2. 使用 root 权限登录 Centos，并确保 yum 包更新到最新。

```
$ sudo yum update
```

3. 卸载旧版本(如果安装过旧版本的话)

```
$ sudo yum remove docker  docker-common docker-selinux docker-engine
```

4. 安装需要的软件包，yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

5. 设置yum源

```
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

6. 可以查看所有仓库中所有docker版本，并选择特定版本安装


```
$ yum list docker-ce --showduplicates | sort -r
```

7. 安装docker

```
$ sudo yum -y install docker-ce  #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0
$ sudo yum -y install <FQPN>  # 例如：sudo yum install docker-ce-17.12.0.ce
```

8. 启动并加入开机启动
```
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

9. 验证安装是否成功


```
# 有client和service两部分表示docker安装启动都成功了
$ docker version
```



## Ubuntu-16.04安装



1. 由于apt官方库里的docker版本可能比较旧，所以先卸载可能存在的旧版本

```
$ sudo apt-get remove docker docker-engine docker-ce docker.io
```

2. 更新apt包索引：

```
$ sudo apt-get update
```

3. 安装以下包以使apt可以通过HTTPS使用存储库（repository）：

```
$ sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```

4. 添加Docker官方的GPG密钥：

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

5. 使用下面的命令来设置stable存储库：

```
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

6. 再更新一下apt包索引：

```
$ sudo apt-get update
```

7. 安装指定版本

- 在生产系统上，可能会需要应该安装一个特定版本的Docker CE，而不是总是使用最新版本，列出可用的版本：

```
$ apt-cache madison docker-ce
```

- 选择要安装的特定版本，第二列是版本字符串，第三列是存储库名称，它指示包来自哪个存储库，以及扩展它的稳定性级别。要安装一个特定的版本，将版本字符串附加到包名中，并通过等号(=)分隔它们：

```
$ sudo apt-get install docker-ce=18.06.3~ce~3-0~ubuntu -y
```



8. 直接安装最新版本的Docker CE：

```
$ sudo apt-get install -y docker-ce
```

9. 验证docker，查看docker服务是否启动：

```
$ systemctl status docker
```



## 参考资料
> - []()
> - []()