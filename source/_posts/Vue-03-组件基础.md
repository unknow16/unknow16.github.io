---
title: Vue-03-组件基础
date: 2018-08-05 19:22:49
tags: Web
---

## data 必须是一个函数
```
<div id = "app">
	<button-counter></button-counter>
	<button-counter></button-counter>
	<button-counter></button-counter>
</div>

<script type="text/javascript">
	Vue.component("button-counter", {
		data: function() {
			return {
				count: 0
			}
		},
		template: '<button @click="count += 1">你点击了我{{count}}次</button>'
	});
	
	var vm = new Vue({
		el: '#app'
	});
</script>
```
因为组件是可复用的 Vue 实例，所以它们与 new Vue 接收相同的选项，例如 data、computed、watch、methods 以及生命周期钩子等。仅有的例外是像 el 这样根实例特有的选项。

当我们定义这个 <button-counter> 组件时，你可能会发现它的 data 并不是像这样直接提供一个对象：


```
data: {
  count: 0
}
```

取而代之的是，一个组件的 data 选项必须是一个函数，因此每个实例可以维护一份被返回对象的独立的拷贝：


```
data: function () {
  return {
    count: 0
  }
}
```

如果 Vue 没有这条规则，点击一个按钮就可能会影响到其它所有实例。

## 通过 Prop 向子组件传递数据
```
Vue.component("blog-post",{
	props: ['title'],
	template: '<h1>{{title}}</h1>'
});

<blog-post title="blog-title"></blog-post>
<blog-post title="bbbbb"></blog-post>
<blog-post title="cccccc"></blog-post>
```

## 每个组件必须只有一个根元素
你可以将模板的内容包裹在一个父元素内，来修复这个问题。
```
<div id="app">
	<blog-post
		v-for="post in posts"
		v-bind:key="post.key"
		v-bind:post="post">
	</blog-post>
</div>

<script type="text/javascript">
	Vue.component("blog-post", {
		props: ['post'],
		template: '<div><h1>{{post.title}}</h1><p v-html="post.content"></p></div>'
	});
	
	var vm = new Vue({
		el: "#app",
		data: {
			posts: [
				{key: 1, title: 't1', content: '<span style="color:red">content1</span>'},
				{key: 2, title: 't2', content: '<span style="color:red">content2</span>'},
				{key: 3, title: 't3', content: '<span style="color:red">content3</span>'}
			]
		}
	});
</script>
```
## 通过事件向父级组件发送消息
1. 在组件模版中声明了一个按钮
2. 点击该按钮时，通过$emit(\'enlarge-text\')发射一个enlarge-text事件
3. 组件的父级监听2中发出的enlarge-text，并相应事件，增大字体。
```
<div id="app" :style="{ fontSize : postFontSize + 'em' }">
	<blog-post
		v-on:enlarge-text="postFontSize += 0.1"
		v-for="post in posts"
		v-bind:key="post.key"
		v-bind:post="post">
	</blog-post>
</div>

<script type="text/javascript">
	Vue.component("blog-post", {
		props: ['post'],
		template: '<div> <h1>{{post.title}}</h1> <p><button @click="$emit(\'enlarge-text\')">enlarge-text</button></p> <p v-html="post.content"></p> </div>'
	});
	
	var vm = new Vue({
		el: "#app",
		data: {
			postFontSize: 1,
			posts: [
				{key: 1, title: 't1', content: '<span style="color:red">content1</span>'},
				{key: 2, title: 't2', content: '<span style="color:red">content2</span>'},
				{key: 3, title: 't3', content: '<span style="color:red">content3</span>'}
			]
		}
	});
</script>
```

## 使用事件抛出一个值
有的时候用一个事件来抛出一个特定的值是非常有用的。例如我们可能想让 <blog-post> 组件决定它的文本要放大多少。这时可以使用 $emit 的第二个参数来提供这个值

```
<button v-on:click="$emit('enlarge-text', 0.1)">
  Enlarge text
</button>
```

然后当在父级组件监听这个事件的时候，我们可以通过 $event 访问到被抛出的这个值：


```
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"
></blog-post>
```

或者，如果这个事件处理函数是一个方法：


```
<blog-post
  ...
  v-on:enlarge-text="onEnlargeText"
></blog-post>
```

那么这个值将会作为第一个参数传入这个方法：


```
methods: {
  onEnlargeText: function (enlargeAmount) {
    this.postFontSize += enlargeAmount
  }
}
```

## 在组件上使用 v-model
你可以用 v-model 指令在表单 <input>、<textarea> 及 <select> 元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。

尽管有些神奇，但 v-model 本质上不过是语法糖。它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。


```
<input v-model="searchText">

等价于：

<input
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>
```
为了让它正常工作，这个组件内的 <input> 必须：

- 将其 value 特性绑定到一个名叫 value 的 prop 上
- 在其 input 事件被触发时，将新的值通过自定义的 input 事件抛出
```
<div id="app">
	<p>{{searchText}}</p>
	<custom-input v-model="searchText" >
</div>

<script type="text/javascript">
	Vue.component("custom-input", {
		props: ['value'],
		template: '<input v-bind:value="value" v-on:input=\'$emit(\"input\", $event.target.value)\' >'
	});
	
	var vm = new Vue({
		el: "#app",
		data: {
			searchText: 'aaa'
		}
	});
</script>
```

## 通过插槽分发内容
<slot></slot>用来自定义组件标签之间内容的占位符
```
<style type="text/css">
	.bg-error {
		background-color: red;
	}
</style>

<div id="app">
	<alert-box>dddddddddd</alert-box>
</div>

<script type="text/javascript">
	Vue.component("alert-box", {
		template: '<div class="bg-error"> <strong>Error!!!</strong> <slot></slot> </div>'
	});
	
	var vm = new Vue({
		el: "#app",
		data: {
		}
	});
</script>
```

## 动态组件

## 解析 DOM 模板时的注意事项
有些 HTML 元素，诸如 <ul>、<ol>、<table> 和 <select>，对于哪些元素可以出现在其内部是有严格限制的。而有些元素，诸如 <li>、<tr> 和 <option>，只能出现在其它某些特定的元素内部。

这会导致我们使用这些有约束条件的元素时遇到一些问题。例如：
```
<table>
  <blog-post-row></blog-post-row>
</table>
```
这个自定义组件 <blog-post-row> 会被作为无效的内容提升到外部，并导致最终渲染结果出错。幸好这个特殊的 is 特性给了我们一个变通的办法：
```
<table>
  <tr is="blog-post-row"></tr>
</table>
```
