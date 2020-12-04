---
title: 05-使用方通过Prop向子组件传递数据
toc: true
date: 2019-07-01 21:55:50
tags:
categories:
---



## 使用示例

一个组件默认可以拥有任意数量的 prop，任何值都可以传递给任何 prop。



1. 静态赋值和动态赋值

```
// 组件定义
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})


<!-- 静态赋值 -->
<blog-post title="My journey with Vue"></blog-post>

<!-- 动态赋值 -->
<blog-post v-bind:title="post.title"></blog-post>

<!-- 动态赋予一个复杂表达式的值 -->
<blog-post
  v-bind:title="post.title + ' by ' + post.author.name"
></blog-post>
```
2. 数组赋值

```
new Vue({
  el: '#blog-post-demo',
  data: {
    posts: [
      { id: 1, title: 'My journey with Vue' },
      { id: 2, title: 'Blogging with Vue' },
      { id: 3, title: 'Why Vue is so fun' }
    ]
  }
})


<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
></blog-post>
```

3. 将一个对象的所有属性都作为 prop 传入

```
<!-- 将一个对象的所有属性都作为 prop 传入 -->
<blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"
></blog-post>

等价于<blog-post v-bind="post"></blog-post>

```

4. 传入对象

```
   // 现在，不论何时为 post 对象添加一个新的属性，它都会自动地在 <blog-post> 内可用。
   Vue.component('blog-post', {
     props: ['post'],
     template: `
       <div class="blog-post">
         <h3>{{ post.title }}</h3>
         <div v-html="post.content"></div>
       </div>
     `
   })
   
   <blog-post
     v-for="post in posts"
     v-bind:key="post.id"
     v-bind:post="post"
   ></blog-post>
```

   

## Prop 的大小写

HTML 中的特性名是大小写不敏感的，所以浏览器会把所有大写字符解释为小写字符。Vue运行时将组件渲染为HTML时，也会解释为小写字符，所以使用方需要用短横线分隔命名，但如果你使用字符串模板，那么这个限制就不存在了。
```
Vue.component('blog-post', {
  // 在 JavaScript 中是 camelCase 的
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})

<!-- 在 HTML 中是 kebab-case 的 -->
<blog-post post-title="hello!"></blog-post>
```

## Prop 类型
1. 常用字符列出prop属性值
```
props: ['title', 'likes', 'isPublished', 'commentIds', 'author']
```

2. 以对象形式列出 prop，这些属性的名称和值分别是 prop 各自的名称和类型,会在它们遇到错误的类型时从浏览器的 JavaScript 控制台提示用户。
```
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // or any other constructor
}
```

## 单向数据流
所有的 prop 都使得其父子 prop 之间形成了一个单向下行绑定：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。

额外的，每次父级组件发生更新时，子组件中所有的 prop 都将会刷新为最新的值。这意味着你不应该在一个子组件内部改变 prop。如果你这样做了，Vue 会在浏览器的控制台中发出警告。

这里有两种常见的试图改变一个 prop 的情形：

1. 这个 prop 用来传递一个初始值；这个子组件接下来希望将其作为一个本地的 prop 数据来使用。在这种情况下，最好定义一个本地的 data 属性并将这个 prop 用作其初始值：

```
props: ['initialCounter'],
data: function () {
  return {
    counter: this.initialCounter
  }
}
```

2. 这个 prop 以一种原始的值传入且需要进行转换。在这种情况下，最好使用这个 prop 的值来定义一个计算属性：
```
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```
> 注意在 JavaScript 中对象和数组是通过引用传入的，所以对于一个数组或对象类型的 prop 来说，在子组件中改变这个对象或数组本身将会影响到父组件的状态。

## Prop 验证
我们可以为组件的 prop 指定验证要求，例如你知道的这些类型。如果有一个需求没有被满足，则 Vue 会在浏览器控制台中警告你。这在开发一个会被别人用到的组件时尤其有帮助。

为了定制 prop 的验证方式，你可以为 props 中的值提供一个带有验证需求的对象，而不是一个字符串数组。例如：
```
Vue.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```
> 注意那些 prop 会在一个组件实例创建之前进行验证，所以实例的属性 (如 data、computed 等) 在 default 或 validator 函数中是不可用的。

##  非 Prop 的特性
一个非 prop 特性是指传向一个组件，但是该组件并没有相应 prop 定义的特性。
因为显式定义的 prop 适用于向一个子组件传入信息，然而组件库的作者并不总能预见组件会被用于怎样的场景。这也是为什么组件可以接受任意的特性，而这些特性会被添加到这个组件的根元素上。
例如，想象一下你通过一个 Bootstrap 插件使用了一个第三方的 <bootstrap-date-input> 组件，这个插件需要在其 <input> 上用到一个 data-date-picker 特性。我们可以将这个特性添加到你的组件实例上：
```
<bootstrap-date-input data-date-picker="activated"></bootstrap-date-input>
```
然后这个 data-date-picker="activated" 特性就会自动添加到 <bootstrap-date-input> 的根元素上。

##  替换/合并已有的特性
在某种情况下，我们定义了两个不同的 class 的值：

- form-control，这是在组件的模板内设置好的
- date-picker-theme-dark，这是从组件的父级传入的

对于绝大多数特性来说，从外部提供给组件的值会替换掉组件内部设置好的值。所以如果传入 type="text" 就会替换掉 type="date" 并把它破坏！庆幸的是，class 和 style 特性会稍微智能一些，即两边的值会被合并起来，从而得到最终的值：form-control date-picker-theme-dark。

## 参考资料
> - [vue官网](cn.vuejs.org)
