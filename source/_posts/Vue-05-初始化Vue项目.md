---
title: Vue-05-初始化Vue项目
date: 2018-08-28 12:48:06
tags: Web
---


前提先安装node.js环境

#### 1. 安装cnpm淘宝镜像

参照https://npm.taobao.org/

在CMD中运行

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
　

#### 2. 安装Vue脚手架

```

cnpm install -g webpack

cnpm install -g vue-cli
```

这里的“-g”是全局安装，其目的是为了构建vue项目

#### 3. 构建项目

假设我们要开发一个“后台管理”的项目，项目名称是“admin”

在CMD中运行命令：



```
vue init webpack admin
```

其中 语法为 ： vue init webpack + 项目名

然后，一路回车。。。

#### 4. 安装依赖
然后，进入文件夹，安装依赖：


```
cd admin
cnpm install
```

　　

#### 5. 运行

以下命令来启动项目
```
npm run dev 
```
 

这时，提示打开浏览器，访问http://localhost:8080，如果出现大大的“V”字就说明项目已经构建完毕了。