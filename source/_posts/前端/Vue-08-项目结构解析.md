---
title: Vue-08-项目结构解析
date: 2018-11-15 15:09:55
tags: Vue
---

## 主入口
src/main.js是入口，相当于main函数，最终该js会被渲染到index.html中

#### index.html
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title>admin</title>
  </head>
  <body>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>
```

#### main.js
```
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'          // 导入vue核心库
import App from './App'        // 导入src/App.vue组件
import router from './router'  // 导入src/router目录下的index.js

Vue.config.productionTip = false  // vue全局配置

import 'font-awesome/css/font-awesome.min.css'

import ElementUI from 'element-ui' // 导入ElementUI插件
import './assets/theme/element-#409eff/index.css'

Vue.use(ElementUI)  // 使用插件

/* eslint-disable no-new */
new Vue({
  el: '#app',   // 以index.html中的id为app的div为载体
  router,       // 路由配置
  components: { App }, // 可用组件
  template: '<App/>' //一个字符串模版作为Vue实例的标识使用，App组件会替换该模版
})
```

#### App.vue
顶层路由出口，其只用做展示通过src/router/index.js路由到的组件
```
<template>
  <div id="app">
    <!-- <img src="./assets/logo.png">
     -->
    <transition name="fade" mode="out-in">
      <router-view/>
    </transition>
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {}
}
</script>

<style lang="scss">
// 可写些公共样式
</style>

```

## 路由分发
前面基本结构已成形，然后就是根据src/router/index.js的路由信息，展示相应组件.

#### src/router/index.js
```
import Vue from 'vue'
import Router from 'vue-router' // 导入vue-router插件

Vue.use(Router) // 使用vue-router插件

import HelloWorld from '@/components/HelloWorld'
import Main from '@/pages/Main'
import Dashboard from '@/pages/Dashboard'
import Member from '@/pages/Member'

let routes = [{
  path: '/',    // 匹配/*的都会展示Main组件的<router-view/>出口处
  name: 'Main',
  component: Main,
  hidden: true,  // 用于不在首页侧边导航栏显示
  children: [{  // 嵌套路由
    path: '/',
    name: '首页',
    component: Dashboard
  }]
}]

routes.push({
  path: '/member',  // 匹配/member也到Main组件
  name: '会员管理',
  component: Main,
  iconCls: 'fa fa-user-circle-o',
  children: [{
    path: '/member/data',  //可写成data，匹配到Memer组件
    component: Member,
    name: '会员信息管理'
  }]
})

const router = new Router({
  routes: routes
})

export default router

// export default new Router({
//   routes: routes
// })

```

