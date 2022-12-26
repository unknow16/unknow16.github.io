---
title: hexo相关
toc: true
date: 2022-04-24 12:51:23
tags:
categories:
---

## 环境问题

```js
0. hexo5.x要配合node版本-node-v12.22.1-win-x64
1. hexo clean // 清理db
2. hexo g // 生成html
3. hexo d // 部署

// commit & push md文件到远程hexo 分支
1. git add .
2. git commit -m "xxx"
3. git push origin hexo
```

## 预览pdf

```js
// 1. 安装插件
npm i hexo-pdf -S

// 2. 在md文章中加入下面代码
<br>
{% pdf ./SwClassCode_2021.pdf %}
<br>
```

## 参考资料
> - []()
> - []()
