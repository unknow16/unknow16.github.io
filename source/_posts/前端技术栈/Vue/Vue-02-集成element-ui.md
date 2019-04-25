---
title: Vue-02-集成element-ui
date: 2018-11-15 14:51:58
tags: Vue
---

## 安装常用组件

安装axios，用于调用http请求

```
cnpm install --save axios
```
　


 

 

安装element-ui库


```
cnpm install --save element-ui
```
　　

安装font-awesome图标库

```
cnpm install --save font-awesome
```
　　

安装sass库


```
cnpm install --save-dev node-sass
cnpm install --save-dev sass-loader
```
　　

安装mock.js


```
cnpm install --save-dev mockjs
cnpm install --save-dev axios-mock-adapter
```
 

其中，axios-mock-adapter能拦截http请求的同时模拟需要的数据　

## 生成好看的主题风格
 
1. 进去https://elementui.github.io/theme-chalk-preview/#/zh-CN网站，选择喜欢的颜色
1. 这里，我选择了一个深蓝色作为主题颜色，并下载
1. 把下载的主题放置目录下 src\assets\theme

## 编写程序入口

 

在main.js中导入“font-awesome”和“element-ui”



```
import 'font-awesome/css/font-awesome.min.css'
 
import ElementUI from 'element-ui'
import './assets/theme/element-#0b0a3e/index.css'
Vue.use(ElementUI)
```
　

完整的main.js代码如下：

```
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'
import router from './router'
 
Vue.config.productionTip = false
 
import 'font-awesome/css/font-awesome.min.css'
 
import ElementUI from 'element-ui'
import './assets/theme/element-#0b0a3e/index.css'
Vue.use(ElementUI)
 
/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})
```

https://www.cnblogs.com/GoodHelper/p/8430422.html