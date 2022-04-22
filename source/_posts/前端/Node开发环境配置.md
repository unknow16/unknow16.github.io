---
title: Node.js开发环境配置
toc: true
date: 2022-04-22 10:36:36
tags:
categories:
---

Node有两个版本线: LTS 是长期维护的稳定版本 Current 是新特性版本

## win配置
1. 解压zip包到想放置目录
2. 在解压后的目录下建立 node_global和node_cache，node_global: npm全局安装路径；node_cache: npm全局缓存路径
3. 配置环境变量：新建变量 NODE_PATH , 指向 E:\xxx\node-v12.22.12-win-x64
4. 追加PATH：编辑Path环境变量，在后面追加 %NODE_PATH%   和    %NODE_PATH%\node_global
5. 配置npm全局安装路径: 打开cmd 执行 ，分开执行如下命令
```
npm config set cache "E:\xxx\node-v12.22.12-win-x64"
npm config set prefix  "E:\xxx\node-v12.22.12-win-x64"
```
如果执行命令卡死，可以删除C:\Users\用户名\.npmrc 后重新执行。（用户名：为当前电脑的用户名）

6. npm设置淘宝仓库, 官网地址：https://npmmirror.com/
```
设置镜像
npm config set registry http://registry.npm.taobao.org/

查看镜像设置
npm get registry 

如果要恢复成原来的设置，执行如下：
npm config set registry https://registry.npmjs.org/
```

## 参考资料
> - []()
> - []()
