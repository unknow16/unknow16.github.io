---
title: 从Nuxt.js到SSR
toc: true
date: 2019-07-14 18:50:27
tags:
categories:
---


## SEO

SEO指搜索引擎优化（Search Engine Optimization），它是指通过站内优化，如：网站结构调整、网站内容建设、网站代码优化以及站外优化等方法，来进行搜索引擎优化。

简单说：通过各种技术（手段）来确保，你的Web内容被搜素引擎最大化收录，最大化提高权重，带来更多流量。

**常见关键词：** 白帽、黑帽、SEM、Backlink、Linkbait、PageRank、Keyword Stuffing...，总之都围绕着一个核心：SEO；流量是变现的快车道，SEO 是低成本获取流量的最佳方法。

目前大部分的搜索引擎仅能抓取URI直接输出的数据资源，对于 Ajax 类的异步请求的数据无法抓取；Google 除外，Google 有自己的[Google’s Webmaster AJAX Crawling Guidelines.](https://link.juejin.im?target=https%3A%2F%2Fdevelopers.google.com%2Fwebmasters%2Fajax-crawling%2F)技术支持。

## SPA
SPA指单页 Web 应用（single page web application，SPA），就是只有一张 Web 页面的应用，是加载单个 HTML 页面并在用户与应用程序交互时动态更新该页面的 Web 应用程序。

简单说： Web 不再是一张张页面，而是一个整体的应用，一个由路由系统、数据系统、页面（组件）系统...组成的应用程序，其中路由系统是非必须的。

大部分的 Vue 项目，本质是 SPA 应用，Angular.js、Angular、Vue、React...还有最早的"Pjax"均如此。

SPA 时代，主要是在Web端使用了history或hash（主要是为了低版本浏览器的兼容）API，在首次请求经服务端路由输出整个应用程序后，接下来的路由都由前端掌控了，前端通过路由作为中心枢纽控制一系列页面（组件）的渲染加载和数据交互。

而上面所述的各类框架则是将以：路由、数据、视图为基本结构进行的规范化的封装。

最早的 SPA 应用，由 Gmail、Google Docs、Twitter 等大厂产品实践布道，广泛用于对SEO要求不高的场景中。

## SSR
SSR指服务端渲染（Server Side Render），即：网页是通过服务端渲染生成后输出给客户端。

在 SPA 之前的时代，我们的Web架构大都是 SSR，如：Wordpress（PHP）、JSP技术、JavaWeb...或者 DEDECMS、Discuz! 等这些程序都是传统典型的 SSR 架构，
即：服务端取出数据和模板组合生成 html 输出给前端，前端发生请求时，重新向服务端请求 html 资源，路由也由服务端来控制。

其次，有个概念叫预渲染（Prerendering）。

如果你只是用服务端渲染来改善一个少数的营销页面（如 首页，关于，联系 等等）的 SEO，那你可以用预渲染来实现。预渲染不像服务器渲染那样即时编译 HTML，它只在构建时为了特定的路由生成特定的几个静态页面，等于我们可以通过 Webpack 插件将一些特定页面组件 build 时就编译为 html 文件，直接以静态资源的形式输出给搜索引擎。

但实际的商业应用中，大部分时候我们需要的是即时渲染，这也是我们今天讨论的主题。

## 为什么要SSR？ 为了体验，还有SEO！
首先，用户可能在网络比较慢的情况下从远处访问网站 - 或者通过比较差的带宽。 这些情况下，尽量减少页面请求数量，来保证用户尽快看到基本的内容。可以用Webpack的代码拆分避免强制用户下载整个单页面应用，但是，这样也远没有下载个单独的预先渲染过的 HTML 文件性能高。

在大部分的商业应用中，我们有 SEO 的需求，我们需要搜索引擎更多地抓取到我们的内容，更详细地认识到我们的网页结构，而不是仅对首页或特定静态页进行索引，这是 SSR 最重要的意义。

## Nuxt.js
Nuxt.js是使用 Webpack 和 Node.js 进行封装的基于Vue的SSR框架，使用它，你可以不需要自己搭建一套 SSR 程序，而是通过其约定好的文件结构和API就可以实现一个首屏渲染的 Web 应用。

之所以叫 Nuxt.js 也是因为受到了 Next.js 的启发。

首先，Nuxt.js 是一个 Node 程序，就像上面说的，我们是要把 Vue 跑在服务端，所以必须使用 Node 环境。

我们对 Nuxt.js 应用的访问，实际上是在访问这个 Node.js 程序的路由，程序输出首屏渲染内容 + 用以重新渲染的 SPA 的脚本代码，而路由是由 Nuxt.js 约定好的 pages 文件夹生成的。

所以，整体上，Nuxt.js 通过各个文件夹和配置文件的约束来管理我们的程序，而又不失扩展性，其有自己的插件机制。

## 参考资料
> - []()
> - []()
