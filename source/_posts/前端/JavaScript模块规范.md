---
title: JavaScript模块规范
toc: true
date: 2018-08-05 19:20:54
tags: Web
---

先想一想，为什么模块很重要？
因为有了模块，我们就可以更方便地使用别人的代码，想要什么功能，就加载什么模块。
但是，这样做有一个前提，那就是大家必须以同样的方式编写模块，否则你有你的写法，我有我的写法，岂不是乱了套！
目前，通行的Javascript模块规范共有三种：CommonJs, AMD 和 CMD。

## 服务端用的CommonJS

2009年，美国程序员Ryan Dahl创造了node.js项目，将javascript语言用于服务器端编程。这标志"Javascript模块化编程"正式诞生。因为老实说，在浏览器环境下，没有模块也不是特别大的问题，毕竟网页程序的复杂性有限；但是在服务器端，一定要有模块，与操作系统和其他应用程序互动，否则根本没法编程。

CommonJS官网：http://www.commonjs.org
CommonJS愿景说明：JavaScript是一种功能强大的面向对象的语言，周围有一些最快的动态语言解释器。正式的JavaScript规范为一些对象定义了API，这些API对构建基于浏览器的应用程序很有用。但是，该规范未定义对构建更广泛的应用程序有用的标准库。 CommonJS API将通过定义满足许多常见应用程序需求的API来填补这一空白，最终提供与Python，Ruby和Java一样丰富的标准库。目的是使应用程序开发人员能够使用CommonJS API编写应用程序，然后在不同的JavaScript解释器和主机环境中运行该应用程序。在与CommonJS兼容的系统中，您可以使用JavaScript编写：
- 服务器端JavaScript应用程序
- 命令行工具
- 基于桌面GUI的应用程序 
- 混合应用程序 (Titanium, Adobe AIR)

node.js的模块系统，就是参照CommonJS规范实现的。在CommonJS中，有一个全局性方法require()，用于加载模块。
```
// 假定有一个数学模块math.js，就可以像下面这样加载。
var math = require('math');

// 然后，就可以调用模块提供的方法：
var math = require('math');
math.add(2,3); // 5
```

所以总结来说，CommonJS是服务器端模块的规范，由Node推广使用:

1. 一个单独的文件就是一个模块。每一个模块都是一个单独的作用域，也就是说，在该模块内部定义的变量，无法被其他模块读取，除非定义为global对象的属性。
2. 输出模块变量的最好方法是使用module.exports对象。
3. 加载模块使用require方法，该方法读取一个文件并执行，返回文件内部的module.exports对象

## 浏览器端用的AMD 和 CMD
有了服务器端模块以后，很自然地，大家就想要客户端模块。而且最好两者能够兼容，一个模块不用修改，在服务器和浏览器都可以运行。但是，由于一个重大的局限，使得CommonJS规范不适用于浏览器环境。还是上一节的代码，如果在浏览器中运行，会有一个很大的问题，你能看出来吗？

```
    var math = require('math');

    math.add(2, 3);
```

第二行math.add(2, 3)，在第一行require('math')之后运行，因此必须等math.js加载完成。也就是说，如果加载时间很长，整个应用就会停在那里等。这对服务器端不是一个问题，因为所有的模块都存放在本地硬盘，可以同步加载完成，等待时间就是硬盘的读取时间。但是，对于浏览器，这却是一个大问题，因为模块都放在服务器端，等待时间取决于网速的快慢，可能要等很长时间，浏览器处于"假死"状态。因此，浏览器端的模块，不能采用"同步加载"（synchronous），只能采用"异步加载"（asynchronous）。这就是AMD规范诞生的背景。

AMD是"Asynchronous Module Definition"的缩写，意思就是"异步模块定义"。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

AMD也采用require()语句加载模块，但是不同于CommonJS，它要求两个参数：require([module], callback);

第一个参数[module]，是一个数组，里面的成员就是要加载的模块；第二个参数callback，则是加载成功之后的回调函数。如果将前面的代码改写成AMD形式，就是下面这样：

```
　　require(['math'], function (math) {

　　　　math.add(2, 3);

　　});
```
math.add()与math模块加载不是同步的，浏览器不会发生假死。所以很显然，AMD比较适合浏览器环境。

由于不是JavaScript原生支持，使用AMD规范进行页面开发需要用到对应的库函数，也就是大名鼎鼎require.js，实际上AMD 是 require.js 在推广过程中对模块定义的规范化的产出。另外还有curl.js也实现了AMD规范。

所以总结来说，AMD主要解决两个问题:

1. 多个js文件可能有依赖关系，被依赖的文件需要早于依赖它的文件加载到浏览器
2. js加载的时候浏览器会停止页面渲染，加载文件越多，页面失去响应时间越长

CMD 即Common Module Definition通用模块定义，CMD规范是国内发展出来的，就像AMD有个require.js，CMD有个浏览器的实现sea.js，它要解决的问题和require.js一样，只不过在模块定义方式和模块加载（可以说运行、解析）时机上有所不同。



## ES6 模块
目前 JavaScript 语言，你会发现它有两种格式的模块。一种是 ES6 模块，简称 ESM；另一种是 Node.js 专用的 CommonJS 模块，简称 CJS。这两种模块不兼容。

- 语法上面，CommonJS 模块使用require()加载和module.exports输出，ES6 模块使用import和export。
- 用法上面，require()是同步加载，后面的代码必须等待这个命令执行完，才会执行。import命令则是异步加载，或者更准确地说，ES6 模块有一个独立的静态解析阶段，依赖关系的分析是在那个阶段完成的，最底层的模块第一个执行。
- Node.js 要求 ES6 模块采用.mjs后缀文件名。也就是说，只要脚本文件里面使用import或者export命令，那么就必须采用.mjs后缀名。Node.js 遇到.mjs文件，就认为它是 ES6 模块，默认启用严格模式，不必在每个模块文件顶部指定"use strict"。如果不希望将后缀名改成.mjs，可以在项目的package.json文件中，指定type字段为module。
```
{
   "type": "module"
}
```
一旦设置了以后，该目录里面的 JS 脚本，就被解释用 ES6 模块。
```
# 解释成 ES6 模块
$ node my-app.js
```
如果这时还要使用 CommonJS 模块，那么需要将 CommonJS 脚本的后缀名都改成.cjs。如果没有type字段，或者type字段为commonjs，则.js脚本会被解释成 CommonJS 模块。
总结为一句话：.mjs文件总是以 ES6 模块加载，.cjs文件总是以 CommonJS 模块加载，.js文件的加载取决于package.json里面type字段的设置。
注意，ES6 模块与 CommonJS 模块尽量不要混用。require命令不能加载.mjs文件，会报错，只有import命令才可以加载.mjs文件。反过来，.mjs文件里面也不能使用require命令，必须使用import。

## 参考资料
> - [http://www.ruanyifeng.com/blog/2020/08/how-nodejs-use-es6-module.html](http://www.ruanyifeng.com/blog/2020/08/how-nodejs-use-es6-module.html)