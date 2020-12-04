---
title: 09-vue-router入门
toc: true
date: 2019-07-02 14:45:36
tags: Vue
---

路由，其实就是指向的意思，当我点击页面上的home按钮时，页面中就要显示home的内容，如果点击页面上的about 按钮，页面中就要显示about 的内容。Home按钮  => home 内容， about按钮 => about 内容，也可以说是一种映射. 所以在页面上有两个部分，一个是点击部分，一个是点击之后，显示内容的部分。 

点击之后，怎么做到正确的对应，比如，我点击home 按钮，页面中怎么就正好能显示home的内容。这就要在js 文件中配置路由。

## Vue中的路由概念
Vue-Router插件中有三个基本的概念 route, routes, router。

- route

route是一条路由，由这个英文单词也可以看出来，它是单数， Home按钮  => home内容， 这是一条route,  about按钮 => about 内容， 这是另一条路由。

- routes

routes是一组路由，把上面的每一条路由组合起来，形成一个数组。[{home 按钮 =>home内容 }， { about按钮 => about 内容}]

- router

router是一个机制，相当于一个管理者，它来管理路由。因为routes 只是定义了一组路由，它放在哪里是静止的，当真正来了请求，怎么办？ 就是当用户点击home 按钮的时候，怎么办？这时router 就起作用了，它到routes 中去查找，去找到对应的 home 内容，所以页面中就显示了 home 内容。

客户端中的路由，实际上就是dom 元素的显示和隐藏。当页面中显示home 内容的时候，about 中的内容全部隐藏，反之也是一样。客户端路由有两种实现方式：基于hash 和基于html5 history api。

## 路由运行流程

1. 页面实现

在vue-router中, 我们看到它定义了两个标签router-link 和router-view来对应点击和显示部分。

router-link 就是定义页面中点击的部分，router-view定义显示部分，就是点击后，区配的内容显示在什么地方。所以 router-link还有一个非常重要的属性to，定义点击之后，要到哪里去， 如：

```
<router-link  to="/home">Home</router-link>
```

2. js中配置路由

首先要定义route,一条路由的实现。它是一个对象，由两个部分组成： path和component。  path 指路径，component 指的是组件。如：{path:’/home’, component: home}

我们这里有两条路由，组成一个routes: 

```
const routes = [
  { path: '/home', component: Home },
  { path: '/about', component: About }
]
```

3. 最后创建router 对路由进行管理，它是由构造函数 new vueRouter() 创建，接受routes 参数。

```
const router = new VueRouter({
      routes // routes: routes 的简写
})
```

4. 配置完成后，把router 实例注入到 vue 根实例中,就可以使用路由了

```
const app = new Vue({
  router
}).$mount('#app')
```

5. 执行过程

当用户点击 router-link 标签时，会去寻找它的 to 属性， 它的 to 属性和 js 中配置的路径{ path: '/home', component: Home}  path 一一对应，从而找到了匹配的组件， 最后把组件渲染到 router-view 标签所在的地方。所有的这些实现才是基于hash 实现的。



## 使用示例

1. 添加vue-router依赖
```
npm i vue-router -s
```
2. 新建两个组件，Home.vue 和 About.vue
```
<template>
    <div>
        <h1>home</h1>
        <p>{{msg}}</p>
    </div>
</template>
<script>
    export default {
        data () {
            return {
                msg: "我是home 组件"
            }
        }
    }
</script>


<template>
    <div>
        <h1>about</h1>
        <p>{{aboutMsg}}</p>
    </div>
</template>
<script>
    export default {
        data () {
            return {
                aboutMsg: '我是about组件'
            }
        }
    }
</script>
```
3. 在 App.vue中 定义router-link和 router-view  
```
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <header>
    <!-- router-link 定义点击后导航到哪个路径下 -->
      <router-link to="/home">Home</router-link>
      <router-link to="/about">About</router-link>
    </header>
    
    <!-- 对应的组件内容渲染到router-view中 -->
    <router-view></router-view>   
  </div>
</template>

<script>
export default {
  
}
</script>
```

4. 在 src/router目录下再新建一个index.js, 其中就是定义 路径到 组件的 映射。
```
import Vue from "vue";
import VueRouter from "vue-router";

// 引入组件
import Home from "../components/Home.vue";
import About from "../components/About.vue";

// 要告诉 vue 使用 vueRouter
Vue.use(VueRouter);

const routes = [
    {
        path:"/home",
        component: Home
    },
    {
        path: "/about",
        component: About
    }
]

var router =  new VueRouter({
    routes
})

export default router;
```

5. 在main.js把路由注入到根实例中，启动路由。
```
import Vue from 'vue'
import App from './App'

// 默认找该目录下的index.js
import router from "./router"

Vue.config.productionTip = false

new Vue({
  el: '#app',
  router, // 注入到vue实例中
  components: { App },
  template: '<App/>'
})
```

6.  这时点击页面上的home 和about 可以看到组件来回切换。但是有一个问题，当首次进入页面的时候，页面中并没有显示任何内容。这是因为首次进入页面时，它的路径是 '/'，我们并没有给这个路径做相应的配置。一般，页面一加载进来都会显示home页面，我们也要把这个路径指向home组件。但是如果我们写{ path: '/', component: Home },vue 会报错，因为两条路径却指向同一个方向。这怎么办？这需要重定向，所谓重定向，就是重新给它指定一个方向，它本来是访问 / 路径，我们重新指向‘/home’, 它就相当于访问 '/home', 相应地, home组件就会显示到页面上。vueRouter中用 redirect 来定义重定向。
```
const routes = [
    {
        path:"/home",
        component: home
    },
    {
        path: "/about",
        component: about
    },
    // 重定向
    {
        path: '/', 
        redirect: '/home' 
    }
]
```
现在页面正常了，首次进入显示home, 并且点击也可以看到内容的切换。

7. 最后，我们看一下路由是怎么实现的

打开浏览器控制台，首先看到 router-link 标签渲染成了 a 标签，to 属性变成了a 标签的 href 属性，这时就明白了点击跳转的意思。router-view 标签渲染成了我们定义的组件，其实它就是一个占位符，它在什么地方，匹配路径的组件就在什么地方，所以 router-link 和router-view 标签一一对应，成对出现。

这里还看到，当点击Home和About 来回切换时，a 标签有一个样式类 .router-link-active 也在来回切换， 原来这是当router-link 处于选中状态时，vueRouter 会自动添加这个类，因此我们也可以利用这个类来改变选中时的状态，如选中时，让它变成红色。但当设置 .router-link-active {color: red;}，它并没有生效，这时还要在类前面加一个a, a.router-link-active {color: red;}, 这样就没有问题了。未处于选中状态的router-link， 我们也想给它更改样式，怎么办? 直接给它添加一个 class 就可以了

```
<router-link class="red">Home</router-link>
```

## 参考资料

> - https://www.cnblogs.com/SamWeb/p/6610733.html
> - https://www.cnblogs.com/fozero/p/6185492.html
> - https://blog.csdn.net/nsrainbow