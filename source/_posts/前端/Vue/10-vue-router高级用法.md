---
title: 10-vue-router高级用法
toc: true
date: 2019-07-02 15:54:58
tags: Vue
---

## 动态路由
上面我们定义的路由，都是严格匹配的，只有router-link 中的to属性和 js 中一条路由route中 path 一模一样，才能显示相应的组件component. 但有时现实却不是这样的，当我们去访问网站并登录成功后，它会显示 欢迎你，+ 你的名字。不同的用户登录， 只是显示“你的名字” 部分不同，其它部分是一样的。这就表示，它是一个组件，假设是user组件。不同的用户（就是用户的id不同），它都会导航到同一个user  组件中。这样我们在配置路由的时候，就不能写死, 就是路由中的path属性，不能写死，那要怎么设置? 导航到 user 组件，路径中肯定有user, id 不同，那就给路径一个动态部分来匹配不同的id.  在vue-router中，动态部分 以 : 开头，那么路径就变成了 /user/:id, 这条路由就可以这么写：  { path:"/user/:id", component: user }.

我们定义一个user组件，页面中再添加两个router-link 用于导航， 最后router.js中添加路由配置，来体验一下

1. app.vue 中添加两个router-link
```
<router-link to="/user/123">User123</router-link>
<router-link to="/user/456">User456</router-link>
```

2. src/router/index.js 中配置user动态路由

```
import User from "../components/User.vue"

 /*新增user路径，配置了动态的id*/
{
    path: "/user/:id",
    component: User
},
```
3. 定义User.vue组件

```
<template>
    <div>
        <h1>User</h1>
        <div>我是user组件</div>
    </div>
</template>
<script>
    export default {

    }
</script>
```

这时在页面中点击user123 和user456, 可以看到它们都导航到user组件，配置正确。 　　

在动态路由中，怎么获取到动态部分？ 因为在组件中是可以显示不同部分的，就是上面提到的“你的名字”。

其实，当整个vue-router注入到根实例后，在组件的内部，可以通过this.$route 来获取到 router 实例。它有一个params 属性，就是来获得这个动态部分的。它是一个对象，属性名，就是路径中定义的动态部分 id, 属性值就是router-link中to 属性中的动态部分，如123。ser 组件修改如下：
```
<template>
    <div>
        <h1>User</h1>
        <div>我是user组件, 动态部分是{{userId}}</div>
    </div>
</template>
<script>
    export default {
        computed: {
            userId () {
                return this.$route.params.id
            }
        }
    }
</script>
```
这里还有最后一个问题，就是动态路由在来回切换时，由于它们都是指向同一组件，vue不会销毁再创建这个组件，而是复用这个组件，就是当第一次点击（如：user123）的时候，vue 把对应的组件渲染出来，但在user123, user456点击来回切换的时候，这个组件就不会发生变化了，组件的生命周期不管用了。这时如果想要在组件来回切换的时候做点事情，那么只能在组件内部（user.vue中）利用watch 来监听$route 的变化。把上面的代码用监听$route 实现
```
<script>
    export default {
        // 定义组件时，data必须是一个函数
        data () {
            return {
                userId: ''
            }
        },
        watch: {
            $route (to,from){
                // to表示的是你要去的那个组件
                // from 表示的是你从哪个组件过来的，它们是两个对象
                // 你可以把它打印出来，它们也有一个param 属性
                console.log(to);
                console.log(from);
                this.userId = to.params.id
            }
        }
    }
</script>
```

## 嵌套路由

　　嵌套路由，主要是由我们的页面结构所决定的。当我们进入到home页面的时候，它下面还有分类，如手机系列，平板系列，电脑系列。当我们点击各个分类的时候，它还是需要路由到各个部分，如点击手机，它肯定到对应到手机的部分。

　　在路由的设计上，首先进入到 home ,然后才能进入到phone, tablet, computer.  Phone, tablet, compute 就相当于进入到了home的子元素。所以vue  提供了childrens 属性，它也是一组路由,相当于我们所写的routes。

　　首先，在home页面上定义三个router-link 标签用于导航，然后再定义一个router-view标签，用于渲染对应的组件。router-link 和router-view 标签要一一对应。home.vue 组件修改如下：
```
<template>
    <div>
        <h1>home</h1>
<!-- router-link 的to属性要注意，路由是先进入到home,
     然后才进入相应的子路由如 phone,所以书写时要把 home 带上 -->
        <p>
            <router-link to="/home/phone">手机</router-link>
            <router-link to="/home/tablet">平板</router-link>
            <router-link to="/home/computer">电脑</router-link>
        </p>
        <router-view></router-view>
    </div>
</template>
```

router/index.js中的home路由修改如下：
```
{
    path:"/home",
    // // 下面这个属性也不少，因为，我们是先进入home页面，才能进入子路由
    component: Home,
    children: [
        {
            path: "phone",
            component: Phone
        },
        {
            path: "computer",
            component: Computer
        },
        {
            path: "tablet",
            component: Tablet
        }
    ]
},
```
这时当我们点击home时，它下面没有任何对应的组件进行显示，这通常不是我们想要的。要想点击home时，要想渲染相对应的子组件，那还需要配置一条路由。当进入到home时，它在children中对应的路由path 是空 ‘’，完整的childrens 如下：
```
children: [
    {
        path: "phone",
        component: phone
    },
    {
        path: "tablet",
        component: tablet
    },
    {
        path: "computer",
        component: computer
    },
    // 当进入到home时，下面的组件显示
    {
        path: "",
        component: phone
    }
]
```

## 命名路由

　　命名路由，很简单，因为根据名字就可以知道，这个路由有一个名字，那就直接给这个路由加一个name 属性，就可以了。 给user 路由加一个name 属性：

```
{
        path: "/user/:id",
        name: "user",
        component: user
}
```
命名路由的使用, 在router-link 中to 属性就可以使用对象了, 


```
<router-link to="/user/123">User123</router-link> // 和下面等价 
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>   // 当使用对象作为路由的时候，to前面要加一个冒号,表示绑定
```

编程式导航：这主要应用到按钮点击上。当点击按钮的时候，跳转另一个组件, 这只能用代码，调用rourter.push() 方法。 当们把router 注入到根实例中后，组件中通过 this.$router 可以获取到router, 所以在组件中使用this.$router.push("home"), 就可以跳转到home界面
