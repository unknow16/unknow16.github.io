---
title: 02-手工构建模块化Vue项目
doc: true
date: 2018-11-15 15:07:57
tags: Vue
---



## 目标
最近在学习vue的过程中发现网上的vue教程总有些不同的问题，有的教程上来只说语法，有的教程上来就用vue-cli来建项目，但是vue-cli是整合了webpack等多个插件的工具，不利于我们学习原理。我觉得一个好的教程应该具备以下几点：

- 浅显易懂，说人话
- 每节课都是一个完整的可以运行的例子
- 由浅入深的介绍知识点，中间不能有断层
- 所以我打算写一个我自己的vue入门教程。我们先从一个土得掉渣的例子开始吧

## 分模块前

1. 新建一个空文件夹learn-vue，在该空文件夹中新建index.html，其完整的页面代码如下

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    
    <!-- 1. 引入vue.js -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <title>learn-vue</title>
  </head>
  <body>
  	
  	<!-- 3. 取值，注意此处的id为app，要和第2步中的el属性值匹配 -->
    <div id="app"> 
      <p>{{ message }}</p>
    </div>
    
    <!-- 2. 创建一个Vue实例 -->
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



实际开发中我们不可能把整个网站的js和html全写到一个页面上在下一个例子中我们将会对这个例子进行改造。



## vue和webpack的关系
很多教程上来把webpack跟vue绑在一起教，让很多原本不懂webpack的人以为webpack是vue的组成部分，或者是必不可少的部分。在这里我要声明一句：

> webpack不是vue的必须组成，只是webpack可以让你的js文件看起来更结构化。

webpack是一个js打包工具，使用webpack你可以在一个js文件中使用 import 或者 require 来引用另外一个js文件中定义的组件。这样你就可以把js组件分文件存放了。

虽然vue的基础例子提供了vue-cli，使用这个工具会自动调用webpack来帮你打包js并自动插入js到html。但是这不利于我们学习技术。我们现在从最基本的webpack使用开始了解webpack如何跟vue一起工作。



## 安装webpack
1. 新建package.json
```
{
    "name": "learn-vue",
    "version": "1.0.0"
  }
```
现在我们的learn-vue文件夹下有两个文件了，index.html和package.json

2. 安装webpack

```
npm i --save-dev webpack webpack-cli
```

运行完该指令后，npm自动将webpack和webpack-cli写到了package.json中。此时的package.json的内容如下


```
{
  "name": "learn-vue",
  "version": "1.0.0",
  "devDependencies": {
    "webpack": "^4.25.1",
    "webpack-cli": "^3.1.2"
  }
}
```

我们的learn-vue文件夹下也自动生成了 node_modules和package-lock.json文件

## 改造模块化
通过webpack就可以把多个js文件打包成一个js文件。使用了webpack就可以使用import语句来导入别的js文件，这样做有两个好处

- 不需要将公共库的js引用写到页面上来了，比如vue的引用就不需要写到index.html页面上了。
- 你可以将你的所有js脚本可以按模块来分文件存放了

1. 提取main.js

我们首先来试试把index.html上的js脚本抽取到一个js文件中。新建一个src文件夹，并在其中建立main.js。 并将index.html中的js语句移动到这个文件中，文件的内容为：


```
new Vue({
    el: '#app',
    data: {
      message: 'Hello Vue.js!'
    }
})
```

我们将index.html中的这段语句换成js引用

```
<script src="src/main.js"></script>
```
完整index.html如下：
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>learn-vue</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  </head>

  <body>
	<div id="app">
	      <p>{{ message }}</p>
    </div>
    <script src="src/main.js"></script>
  </body>
 
</html>
```


然后记得再次访问页面，保证我们还能看到Hello Vue.js!字样。到此为止，我们还没有用到webpack。不用急，我们这就把webpack用上。

2. 新建webpack.config.js文件，内容如下


```
var path = require('path');
 
module.exports = {
  mode: 'development',
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  }
};
```

简单的讲解一下这个webpack.config.js

- mode: 主要用到的模式有production和development。开发时使用development，所以我们现在用development，其实你就算用production对我们的例子也没什么影响。
- entry: 入口js，之前不是说过使用了webpack之后就可以引用import来导入别的js文件么？那么你可以想象一下，你项目中的js可以构成一个引用树。这个引用树总要有一个树根的，这个树根是不会被任何js所引用的，所有引用最后都可以回溯到它。这里的entry就是webpack的js引用树树根文件
- output：定义了打包后文件要存放的路径和文件名

从output我们可以知道最终的文件会被存放在dist文件夹中。所以我们还需要建立dist文件夹，最后，我们来看看webpack究竟能干什么。在项目根目录下运行

```
npx webpack
```

如果你看到类似以下的输出


```
$ npx webpack
Hash: 5b16bee4aec18c534801
Version: webpack 4.12.1
Time: 62ms
Built at: 2018-06-27 01:00:04
    Asset      Size  Chunks             Chunk Names
bundle.js  3.84 KiB    main  [emitted]  main
[./src/main.js] 77 bytes {main} [built]
```

那么恭喜你，你成功的构建了第一个webpack打包文件。

然后我们修改index.html页面上对main.js的引用为对bundle.js的引用

```
<script src="dist/bundle.js"></script>
```

然后再访问页面，确保你能看到Hello Vue.js! 

3. 安装vue

看看目录结构，是不是开始有点结构化了呢？有点意思吧。

接下来，我们来做更有意思的事情：将以下的vue的引用从页面上去除


```
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
```

我们来写你的第一个import语句。修改main.js脚本，在头部加上

```
import Vue from 'vue'
```

我们的main.js就变为了


```
import Vue from 'vue'
 
new Vue({
    el: '#app',
    data: {
      message: 'Hello Vue.js!'
    }
})
```

这个import怎么起作用呢？它总不会自己知道要去下载vue的源码吧？！当然没那么神奇了，你需要自己使用npm来安装vue


```
$ npm i --save vue
```

运行完后，我们的package.json中自动新增了vue


```
{
    "name": "learn-vue",
    "version": "1.0.0",
    "devDependencies": {
        "webpack": "^4.12.1",
        "webpack-cli": "^3.0.8"
    },
    "dependencies": {
        "vue": "^2.5.16"
    }
}
```
node_modules文件夹下也增加了vue的包。

我们再使用webpack来构建一次项目


```
$ npx webpack
Hash: cd3c4ea4c7c76aeb5beb
Version: webpack 4.12.1
Time: 410ms
Built at: 2018-06-27 01:16:20
    Asset     Size  Chunks             Chunk Names
bundle.js  235 KiB    main  [emitted]  main
[./node_modules/webpack/buildin/global.js] (webpack)/buildin/global.js 489 bytes {main} [built]
[./src/main.js] 100 bytes {main} [built]
    + 4 hidden modules
```

构建完毕了，此时我们打开dist/bundle.js看看。你会发现它把整个vue库都压缩了并写入了bundle.js文件。这样你的页面在打开时就可以一次性的把所有js库所有的依赖库都下载好了。

我们再刷新一次页面，确保你还能看到Hello Vue.js!

额。。。这回什么都没有。打开控制台可以看到以下警告信息


```
[Vue warn]: You are using the runtime-only build of Vue where the template compiler is not available. Either pre-compile the templates into render functions, or use the compiler-included build.
```

这是因为如果你直接写 import Vue from 'vue' 那么调用的是运行时的vue版本，这个版本不带模板解析包。我们之前在index.html页面上引用的也不是运行时版本。所以在这个例子中正确的做法应该是

```
import Vue from 'vue/dist/vue.js'
```

再打包一次，这回我们就能够看到Hello Vue.js了。

## 组件化
如果你在网上看vue的教程，肯定经常会看到component组件这个概念。真正投入市场的产品都是由组件构成的。那么什么是vue的组件呢？

1. 单文件组件(SFC)

每一个vue的组件都采用单文件组件的格式来组织组件文件。英文是  single file component 简称SFC。SFC文件的后缀名是 .vue。简单的说就是一个 .vue文件就是一个组件，它的结构由三部分组成

- html模板
- js脚本
- css样式

2. 新建HelloVue组件

组件放哪都行，不过业界的习惯是把组件放到 src/components文件夹下。所以我们在src下建立component文件夹

然后在src目录下建立一个新的组件HelloVue.vue，然后把之前写在index.html和main.js的代码搬到HelloVue.vue里面


```
<template>
   <p>{{ message }}</p>
</template>
<script>
export default {
  name: 'HelloVue',
  data: function() {
    return {
      message: 'Hello Vue.js!'
    }
  }
}
</script>
```
大家会发现跟之前的代码比起来有几个不同点：

- 不需要执行 new Vue了，只需要用export default {...} 来定义组件即可
- data属性变成一个方法了，该方法可以通过一串预处理过程得出一个json对象。此处我们没有预处理过程，所以直接返回对象咯
- 没有<style>，由于我们的例子很简单，所以就省略了<style>元素

与此同时，index.html中的id="app"的div元素应该简化为以下形式，因为具体的模板已经被移到HelloVue.vue中去了

```
<div id="app"></div>
```

- name属性

有没有看到现在我们的组件有了一个name属性，这个属性很重要。以至于我单独开来一个小节来说它。

vue在启动时会注册各个组件，每个组件的注册名就是name属性定义的值。各个组件之间可以通过name来调用。比如你定义了HelloVue组件，那么在另外一个组件的模板中就可以使用以下方式来调用，比如：


```
<template>
   <h2>Welcome!</h2>
   
   <!-- 3. 使用 -->
   <HelloVue/>
</template>
 
<script>
<!-- 1.引入组件 -->
import HelloVue from './components/HelloVue.vue'
 
export default {
   name: 'Welcome',
   // 2. 定义
   component: {
      HelloVue
   }
}
</script>
```

我们在别的组件中可以直接写上<HelloVue />标签，并使用component组件来对HelloVue标签进行解释，告诉Vue使用HelloVue组件来渲染<HelloVue/>标签，这个标签的名字必须跟HelloVue.vue中的name属性一模一样。请大家记住这个知识点，因为接下来我们马上就会用到。

-  template属性

实际上template节点也不是必须的，你可以通过template属性来定义html模板。如果我们使用template属性来定义模板，上面的例子就可以变为


```
<script>
export default {
  name: 'HelloVue',
  template: "<p>{{ message }}</p>",
  data: function() {
    return {
      message: 'Hello Vue.js!'
    }
  }
}
</script>
```

但是实际工作中大家肯定更愿意使用<template>元素来定义html模板咯？为什么？因为代码分离，可读性更高啊！那么我为什么要提这茬呢？因为我们接下来要讲的部分会用到这个知识点。

3. 加载HelloVue组件

写好了HelloVue.vue组件之后，接下来的问题就是如何将HelloVue组件跟index.html页面连在一起呢？

我相信很多人跟我一样吧。面对.vue文件和.html之间的连接感到一头雾水！完全无法理解为什么.vue文件可以最终被渲染到页面上。

现在我就来给大家解惑。请大家回忆一下之前提到过的两个知识点：

- .vue组件通过name属性注册专属的标签，比如<HelloVue/>
- 模板可以通过Vue对象的template属性定义

index.html是不可能直接使用.vue文件的，因为浏览器不知道.vue文件是什么。所以HelloVue.vue是通过被某个js调用，从而被index.html所感知的。所以它们之间必须要有一个js文件作为连接。我们使用之前建立的main.js来引用HelloVue.vue。刚刚的那团迷雾就变为了main.js



具体步骤，请跟我做，修改src/main.js文件，内容为：

```
import Vue from 'vue/dist/vue.js'
import HelloVue from './components/HelloVue.vue'
 
new Vue({
    el: '#app',
    components: { HelloVue },
    template: "<HelloVue/>"
})
```

我们并没有修改 el: '#app' 属性，除此之外main.js有以下的改动

- 使用import增加了对 HelloVue的导入
- 由于main.js是一个js文件，所以我们肯定不能用<template>标签来定义模板了。这就用到了刚刚介绍的知识点：使用template属性来定义模板
- template中的内容是 <HelloVue/>。这就是用到来刚刚介绍的知识点：使用name属性来注册组件。我们在模板中使用了Hello.vue组件

4. 解析vue文件

光是这样你是不能成功的使用npx webpack命令成功打包js的，因为webpack不认识.vue文件，不知道怎么解析它。所以我们需要加上vue的加载器

```
npm i --save-dev vue-loader vue-template-compiler
```

然后我们需要修改 webpack.config.js文件来使用vue加载器。vue-loader是webpack的一个插件。所以我们在webpack.config.js中先要声明出这个插件

```
const { VueLoaderPlugin } = require('vue-loader')
```

然后使用它

```
plugins: [
    new VueLoaderPlugin()
  ]
```

最后告诉webpack哪些文件需要用这个插件来解析


```
module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
      }
    ]
  },
```

最后，完整的webpack.config.js文件是这样的


```
var path = require('path');
const { VueLoaderPlugin } = require('vue-loader')
 
module.exports = {
  mode: 'development',
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
      }
    ]
  },
  plugins: [
    new VueLoaderPlugin()
  ]
};
```

最后我们终于可以使用npx webpack命令来打包js了。记得刷新浏览器确保你能看到 Hello Vue.js! 字样。

## 用App.vue进行页面布局
现在的vue项目大多都有一个App.vue作为所有vue组件的根组件。我也不知道这是谁定的规则，但是这算是一个业界潜规则吧。使用App.vue的好处就是：如果通过main.js的template属性来定义整个页面的结构，可以想象template属性内的值得有多长，得有多丑陋。如果使用来App.vue你可以把页面的基本结构，写到App.vue的<template>标签中。main.js就单纯负责启动就行了。

那我们就在src目录下建立App.vue文件，内容为


```
<template>
  <div id="app">
    <HelloVue/>
  </div>
</template>
<script>
import HelloVue from './components/HelloVue.vue'
export default {
  name: 'app',
  components: {
    HelloVue
  }
}
</script>
```

然后我们修改main.js文件，使其引用

```
import Vue from 'vue/dist/vue.js'
import App from './App.vue'
 
new Vue({
    el: '#app',
    components: { App },
    template: "<App/>"
})
```

如果你打开一个别人已经写好的vue项目的main.js文件，那么你多半会看到这样的语句


```
new Vue({
  render: h => h(App)
}).$mount('#app')
```

这看起来跟我们写的完全不一样嘛。我第一次看到的时候不知道有多迷茫。现在我们来解释一下其中的语法

- $mount

  $mount是一个vue的全局函数，表示将构建出来的组件挂载到dom对象上。所以 .$mount('#app') 其实就相当于 el: '#app'。关于$mount的其他知识点在此不会详细的介绍。

- render: h => h(App)
这是个什么鬼？！

Vue是建议大家使用template来定义模板的，Vue会帮你进行渲染。但是你也可以不使用template，使用render属性直接渲染页面。

h是createElement函数的缩写形式。

```
render: h => h(App)
```

就是以下语句的缩写形式

```
render: function (createElement) {
    return createElement(App);
}
```

createElement 函数可以创造出一个虚拟dom对象。$mount函数可以把这个虚拟dom对象挂载到id=app的dom对象上。

关于这些函数和表达法的具体知识点暂时可以不需要了解。所以我们的main.js最终形态为

```
import Vue from 'vue/dist/vue.js'
import App from './App.vue'
 
new Vue({
    render: h => h(App)
  }).$mount('#app')
```

## 自动插入脚本
另外，由于我们现在的业务逻辑都在js里面，而浏览器又有缓存机制，所以用户可能没法得到正确的代码版本。业界一直一来都有一个方法来解决这个问题，就是在js的文件名上或者请求参数后增加版本号或者一段随机字符串。但是这个事情自己做比较麻烦。因为你既要改js又要改页面上引用js的代码。现在有了一个更优雅的解决方案：使用html-webpack-plugin帮我们自动插入脚本，并在引用url中增加随机字符串。

html-webpack-plugin就有这么神奇。我们来试试吧。首先肯定是安装插件啦

```
$ npm i --save-dev html-webpack-plugin
```

然后我们在webpack.config.js中增加html-webpack-plugin的配置代码。

首先在webpack.config.js文件头部增加HtmlWebpackPlugin的声明代码

```
const HtmlWebpackPlugin = require('html-webpack-plugin')
```

然后在plugin段落增加HtmlWebpackPlugin的配置


```
plugins: [
    new VueLoaderPlugin(),
   // 以下是HtmlWebpackPlugin的配置
    new HtmlWebpackPlugin({
      template: 'index.html',
      filename: './index.html', // 输出文件【注意：这里的根路径是module.exports.output.path】
      hash: true
    })
  ]
```

我这篇教程只是入门教程，所以不会详细的介绍HtmlWebpackPlugin的所有配置，我只说其中最基础的三个参数：

- template: 模板文件路径，或者可以成为源文件的文件路径
- filename: 输出文件路径，这里的路径是相对于module.exports.output.path的，比如本例中 module.exports.output.path 为path.resolve(__dirname, 'dist')，所以按照我的配置，最终输出的文件路径应该为 learn-vue/dist/index.html
- hash: 这里就是配置是否每次编译都在js引用中增加随机字符串

既然index.html已经成为了源文件，那么我们就要把index.html中对于build.js的引用删除，我们来将其改成一段注释


```
<body>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
```

有了这段注释作为参照，你就更能清楚的分辨出哪些是html-webpack-plugin自动生成的代码了。

配置好后，我们就来运行npx webpack 体验一下html-webpack-plugin所带来的魔法一般的效果吧

成功编译后，我们打开 dist/index.html，内容为


```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>learn-vue</title>
  </head>
  <body>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  <script type="text/javascript" src="bundle.js?d87dda04866c17d61f5f"></script></body>
</html>
```

看build.js已经被自动的插入index.html中了。而且还带上了随机字符串！

最后别忘记再刷新一下页面，确保你还能看到 Hello Vue.js字样。不过现在你应该看不到Hello Vue.js了。因为我们应该访问的是 dist/index.html 文件，而不是index.html。将url修改为访问 http://learn-vue/dist/index.html 后，Hello Vue.js又出现了

（完）