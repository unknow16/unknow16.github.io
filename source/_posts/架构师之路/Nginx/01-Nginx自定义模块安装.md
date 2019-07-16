---
title: 01-Nginx自定义模块安装
toc: true
date: 2019-07-14 11:21:55
tags:
categories:
---



##  默认安装

```
1. 下载源码： wget http://nginx.org/download/nginx-1.14.2.tar.gz
2. 解压：  tar -zxvf nginx-1.14.2.tar.gz 
3. 进入： cd nginx-1.14.2
4. 检查配置环境并设置安装路径：  ./configure --prefix=/usr/local/nginx-1.14
5. 编译安装： make && make install
```



## 常用命令

```
启动：sbin/nginx
重启：sbin/nginx -s reload
快速停止服务：sbin/nginx -s stop
正常停止服务：sbin/nginx -s quit
测试配置信息：sbin/nginx -t 
显示版本信息：sbin/nginx -v      【-V（大V）显示编译时的参数】
查看启动状态：ps -ef | grep nginx
```





## 参考资料
> - []()
> - []()
