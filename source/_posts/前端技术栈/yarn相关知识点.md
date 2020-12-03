---
title: yarn相关知识点
toc: true
date: 2020-12-03 11:33:01
tags:
categories:
---

## yarn相关文件
- package.json：包含包的所有依赖信息，包括开发依赖、运行依赖、可选依赖等。 每个依赖都需要指明依赖名称和最低可用版本。
- yarn.lock：记录每一个依赖项的确切版本信息，这可以确保你的包每次安装的一致性。

## yarn工作流
你的项目在引入了包管理器的同时，也引入了一套新的围绕着依赖项开发的工作流程。Yarn尽力不改变你的工作流程，并使流程中的每一步都简单明了。

1. 创建一个新项目, 用yarn init初始化生成package.json
2. 增加／更新／删除依赖，用yarn add [package]添加依赖，[package]会被加入到package.json文件中的依赖列表，同时yarn.lock也会被更新。
3. 将package.json/ yarn.lock加入版本控制系统
4. 项目的另一个开发者检出代码，用yarn install安装你的依赖

## 从 npm 迁移
对多数用户来说，从npm迁移的过程应该非常简单。Yarn和npm使用相同的package.json格式，而且Yarn可以从npm安装依赖包。如果你打算在现有项目中尝试Yarn，只需执行：yarn。Yarn将通过自己的解析算法来重新组织node_modules 目录，这个算法和node.js 模块解析算法是兼容的。
执行yarn命令或者yarn add <package>命令后，Yarn都会在项目根目录下生成yarn.lock文件。 你无需理解此文件的具体内容，但请记得将其提交到代码管理系统。 当其他开发者也从npm迁移到Yarn时，yarn.lock文件的存在会确保他们得到的依赖包与你的完全相同。

## 常用命令
```
# 帮助
yarn help

# 初始化一个新项目，交互式生成package.json
yarn init

# 安装所有依赖
yarn install

# 执行不带任何命令的yarn，等同于执行yarn install，并透传所有参数
yarn

# 执行用户自定义脚本
yarn run <script> [<args>]

# 执行用户自定义脚本可省略run
yarn <script> [<args>]

# 添加一个依赖
yarn add [package]
yarn add [package]@[version]
yarn add [package]@[tag]

# 添加不同分类的依赖
yarn add [package] --dev  # dev dependencies
yarn add [package] --peer # peer dependencies
yarn add [package] --optional

# 升级依赖
yarn up [package]
yarn up [package]@[version]
yarn up [package]@[tag]

# 移除依赖
yarn remove [package]
```

## 参考资料
> - [https://classic.yarnpkg.com/zh-Hans/docs](https://classic.yarnpkg.com/zh-Hans/docs)
> - []()
