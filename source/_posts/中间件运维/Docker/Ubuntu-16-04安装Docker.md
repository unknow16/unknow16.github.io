---
title: Ubuntu-16.04安装Docker
toc: true
date: 2019-10-22 11:05:56
tags:
categories:
---



1. 更新APT的源，安装https和ca证书的库，默认这2个库都已经装了

```
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates
```

2. 添加秘钥GPG到APT配置中。

```
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```

3. 增加Docker的源到/etc/apt/souces.list文件中

```
sudo vim /etc/apt/sources.list
```

​	最后一行增加内容

```
deb https://apt.dockerproject.org/repo ubuntu-trusty main
deb http://cz.archive.ubuntu.com/ubuntu trusty main
```

4. 安装Docker

```
sudo apt-get update
sudo apt-get install docker-engine
```



## 参考资料
> - []()
> - []()
