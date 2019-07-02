---
title: 02-node和npm常用命令
toc: true
date: 2018-05-19 20:02:35
---
## node init
* 交互式创建一个package.json文件


## npm install X -g
* 全局安装xxx
* 会安装到node安装目录下的node_modules

## npm install X

- 会把X包安装到node_modules目录中
- 不会修改package.json
- 之后运行npm install命令时，不会自动安装X
- 之后运行npm install –production或者注明NODE_ENV变量值为production时，会自动安装msbuild到node_modules目录中


## npm install X –save

- 会把X包安装到node_modules目录中
- 会在package.json的dependencies属性下添加X
- 之后运行npm install命令时，会自动安装X到node_modules目录中

## npm install X –save-dev

- 会把X包安装到node_modules目录中
- 会在package.json的devDependencies属性下添加X
- 之后运行npm install命令时，会自动安装X到node_modules目录中
- 之后运行npm install  –production或者注明NODE_ENV变量值为production时，不会自动安装X到node_modules目录中
- 安装开发阶段依赖，一般调试工具居多