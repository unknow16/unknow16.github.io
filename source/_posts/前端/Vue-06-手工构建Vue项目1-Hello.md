---
title: Vue-06-手工构建Vue项目1-Hello
date: 2018-11-15 15:07:42
tags: Vue
---

## 目标
最近在学习vue的过程中发现网上的vue教程总有些不同的问题，有的教程上来只说语法，有的教程上来就用vue-cli来建项目，但是vue-cli是整合了webpack等多个插件的工具，不利于我们学习原理。我觉得一个好的教程应该具备以下几点：

- 浅显易懂，说人话
- 每节课都是一个完整的可以运行的例子
- 由浅入深的介绍知识点，中间不能有断层
- 所以我打算写一个我自己的vue入门教程。我们先从一个土得掉渣的例子开始吧

## 步骤

1. 新建一个空文件夹
```
mkdir learn-vue
cd learn-vue
```

2. 在该空文件夹中新建index.html
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>learn-vue</title>
  </head>
  <body>
Hello World
  </body>
</html>
```

在使用webpack，分component之前，你所能想到最土的js框架学习例子是什么？我猜就是像我们刚学js一样，在网页上直接调用vue的js文件，然后在页面上调用吧。我们现在就来试试。

3. 添加js引用

首先，从官网上把vue.js的源文件调用代码直接拷贝到我们的例子之中：


```
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
```

4. 创建dom组件

vue将页面上的元素都看成是一个个组件，每个组件在js内存中都会对应一个抽象模型。通过对这个模型的操作达到对dom元素的操作，从而让js编程变得很优雅，可读性很高。

我们来加入第一个组件。vue中的组件由html模板+js脚本+css样式组成。其中css样式是可选的，在这里我们就先不引入css样式。先写第一个html模板吧。我们把之前的hello world字样换成



```
<div id="app">
  <p>{{ message }}</p>
</div>
```

其中{{ message }} 语法vue的模板语言语法，表示输出变量 message 的内容。我们为这个div定义来id属性，值为app。目的是方便js脚本定位它，并将它抽象成一个对象。

5. 创建js组件

接下来，我们来写js脚本。在刚刚建立的 dome对象和body之间加入script代码片段：


```
<script>
new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue.js!'
  }
})
</script>
```

new Vue表示建立一个dom所对应的抽象对象。el属性定义了使用什么selector来获取到这个对象对应的dom，因为我们要对应的dom对象id为app，所以此处使用 #app 作为 selector 

6. 渲染页面
一旦vue将某个dom对象抽象为了一个js对象后，就会对其进行渲染，我们之前写的 {{ message }} 就会被渲染为真正你想在页面上显示的文本。data属性定义了渲染这个dom对象是需要赋值的属性集合。我们定义message属性的值为"Hello Vue.js!"

完整的页面代码为：


```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <title>learn-vue</title>
  </head>
  <body>
    <div id="app">
      <p>{{ message }}</p>
    </div>
    <script>
      new Vue({
        el: '#app',
        data: {
          message: 'Hello Vue.js!'
        }
      })
      </script>
  </body>
</html>
```

保存后刷新页面，你将会看到Hello Vue.js!

ok, 你完成了第一个vue的学习例子。这个例子极土，但是很接地气，在下一个例子中我们将会对这个例子进行改造，让它不那么土。
