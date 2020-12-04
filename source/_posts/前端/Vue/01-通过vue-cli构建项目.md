---
title: 01-通过vue-cli构建项目
toc: true
date: 2018-11-15 14:49:09
tags: Vue
---

#### 1. node.js和npm环境安装
#### 2. 安装cnpm淘宝镜像
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
#### 3. 安装Vue脚手架
```
cnpm install -g webpack

cnpm install -g vue-cli
```
#### 4. 构架项目
```
// fuyi-ui为项目名
vue init webpack fuyi-ui
```
然后根据自己需求，选择相应功能。

#### 5. 出现提示运行命令，就说明已经构建成功了

```
npm install ：安装所需要的依赖；
npm run dev ：以开发模式启动项目；
npm run build ：以生产模式生成静态页面文件。
```
#### 6. 安装依赖
```
cd fuyi-ui
cnpm install
```

#### 7. 最后运行

```
npm run dev
```
#### 8. 打开浏览器

访问http://localhost:8080，出现大大的“V”字就说明项目已经构建完毕了。



