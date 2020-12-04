---
title: go语言环境安装与介绍
toc: true
date: 2020-01-16 11:19:54
tags:
categories:
---





```
## 进入目录
cd /usr/local

## 下载
sudo wget https://dl.google.com/go/go1.13.6.linux-amd64.tar.gz

## 解压
sudo tar -zxvf go1.13.6.linux-amd64.tar.gz

## 编辑/etc/profile,新增如下环境变量
export GOPATH=/home/fuyi/go
export GOROOT=/usr/local/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin

## 使配置生效
source /etc/profile

## 验证
go version 或 go env
```



## 参考资料
> - []()
> - []()
