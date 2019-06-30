---
title: 10-Slot使用示例
doc: true
date: 2018-11-15 18:37:07
tags: Vue
---

简单来说，Slot是对组件的扩展，通过slot插槽向组件内部指定位置传递内容

## 入门示例

```
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<title>Insert title here</title>
	<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
	<style type="text/css">
		.bg-error {
			background-color: red;
		}
	</style>
</head>
<body>
	<div id="app">
	    <!-- <2> 使用组件时，传入值会替换模版中的slot -->
		<alert-box>dddddddddd</alert-box>
	</div>

	<script type="text/javascript">
	    // <1> 组件模版中定义slot插槽
		Vue.component("alert-box", {
			template: '<div class="bg-error"> <strong>Error!!!</strong> <slot></slot> </div>'
		});
		
		var vm = new Vue({
			el: "#app",
			data: {
			}
		});
	</script>

</body>
</html>
```

插槽显示的位置在定义模版时决定，显示的内容使用组件时决定。slot写在组件template的哪块，使用组件时传过来的模板内容将来就显示在哪块。

## 单个插槽
单个插槽是vue的官方叫法，但是其实也可以叫它默认插槽，或者与具名插槽相对，我们可以叫它匿名插槽。因为它不用设置name属性。

* ChildSlot.vue
```
<template>
  <section class="bg-child">
    <div>
      <h2>这是子组件</h2>
      <slot>子组件-slot默认值</slot>
    </div>
  </section>
</template>

<script>
export default {
  
}
</script>

<style>
  .bg-child {
    background: #84ccc9
  }
</style>

```

* ParentSlot.vue
```
<template>
  <section class="bg-parent">
    <div>
      <h2>这是父组件</h2>
      <ChildSlot>
        <div class="tmpl">
              <span>菜单1</span>
              <span>菜单2</span>
              <span>菜单3</span>
              <span>菜单4</span>
              <span>菜单5</span>
              <span>菜单6</span>
        </div>
      </ChildSlot>
    </div>
  </section>
</template>

<script>
import ChildSlot from '@/components/ChildSlot.vue'

export default {
  components: {ChildSlot}
}
</script>

<style>
  .bg-parent {
    background: #999999;
    padding: 10px;
  }
</style>

```

## 具名插槽
匿名插槽没有name属性，所以是匿名插槽，那么，插槽加了name属性，就变成了具名插槽。具名插槽可以在一个组件中出现N次。出现在不同的位置。下面的例子，就是一个有两个具名插槽和单个插槽的组件，这三个插槽被父组件用同一套css样式显示了出来，不同的是内容上略有区别。

#### childSlotNamed.vue
```
<template>
  <section class="bg-child">
    <div>
      <!-- 具名插槽 -->
      <slot name="up">子组件slot， name=up的默认值</slot>
      
      <h2>这是子组件</h2>
      
      <!-- 具名插槽 -->
      <slot name="down">子组件slot， name=down的默认值</slot>
      
      <br>
      <!-- 匿名插槽 -->
      <slot>子组件-slot默认值</slot>
    </div>
  </section>
</template>

<script>
export default {
  
}
</script>

<style>
  .bg-child {
    background: #84ccc9
  }
</style>

```

#### parentSlotNamed

```
<template>
  <section class="bg-parent">
    <div>
      <h2>这是父组件</h2>
      <ChildSlotNamed>
        <div slot="up">
              <span>菜单1-up</span>
              <span>菜单2-up</span>
              <span>菜单3-up</span>
              <span>菜单4-up</span>
              <span>菜单5-up</span>
              <span>菜单6-up</span>
        </div>
         <div slot="down">
              <span>菜单1-down</span>
              <span>菜单2-down</span>
              <span>菜单3-down</span>
              <span>菜单4-down</span>
              <span>菜单5-down</span>
              <span>菜单6-down</span>
        </div>
         <div>
              <span>菜单1</span>
              <span>菜单2</span>
              <span>菜单3</span>
              <span>菜单4</span>
              <span>菜单5</span>
              <span>菜单6</span>
        </div>
      </ChildSlotNamed>
    </div>
  </section>
</template>

<script>
import ChildSlotNamed from '@/components/ChildSlotNamed.vue'

export default {
  components: {ChildSlotNamed}
}
</script>

<style>
  .bg-parent {
    background: #999999;
    padding: 10px;
  }
</style>

```

## 作用域插槽 | 带数据的插槽
父组件决定html+css，子组件决定数据。

下面的例子，你就能看到，父组件提供了三种样式(分别是flex、ul、直接显示)，都没有提供数据，数据使用的都是子组件插槽自己绑定的那个人名数组。

#### childSlotScope.vue

```
<template>
  <section class="bg-child">
    <div>
      <h2>这是子组件</h2>
      <slot :data="data"></slot>
    </div>
  </section>
</template>

<script>
export default {
  data: function() {
    return {
      data: ['zhangsan','lisi','wanwu','zhaoliu','tianqi','xiaoba']
    }
  }
}
</script>

<style>
  .bg-child {
    background: #84ccc9
  }
</style>

```

#### parentSlotScope.vue

```
<template>
  <section class="bg-parent">
    <div>
      <h2>这是父组件</h2>
      <ChildSlotScope>
        <template slot-scope="user">
          <div>
            <span class="flex-border" v-for="item in user.data" :key="item">{{item}}</span>
          </div>
        </template>
      </ChildSlotScope>


      <ChildSlotScope>
        <template slot-scope="user">
          <ul>
            <li v-for="item in user.data" :key="item">{{item}}</li>
          </ul>
        </template>
      </ChildSlotScope>

      <child-slot-scope>
        <template slot-scope="user">
          {{user.data}}
        </template>
      </child-slot-scope>

      <child-slot-scope>不使用其提供的数据, 作用域插槽退变成匿名插槽</child-slot-scope>
    </div>
  </section>
</template>

<script>
import ChildSlotScope from '@/components/ChildSlotScope.vue'

export default {
  components: {ChildSlotScope}
}
</script>

<style>
  .bg-parent {
    background: #999999;
    padding: 10px;
  }

  .flex-border {
    padding: 10px;
  }
</style>

```
