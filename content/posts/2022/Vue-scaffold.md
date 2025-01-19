---
title: Vue2 组件和脚手架
date: 2022-12-07 20:14:54
tags: [vue, web]
categories: [learn]
description: Vue 组件语法与脚手架使用。
summary: Vue 组件语法与脚手架使用。
---

组件是实现应用中局部功能代码和资源的集合，主要作用是复用代码，简化项目编码，提高运行效率。

## 非单文件组件

一个文件中包含多个组件。

```vue

<div id="root">
  {{msg}}
  <hr>
  <!-- 使用组件 -->
  <school></school>
  <hr>
  <student></student>
</div>

<script>
  Vue.config.productionTip = false

  // 定义组件
  const school = Vue.extend({
    template: `
          <div>
          <h1>学校名称：{{ schoolName }}</h1>
          <h1>学校地址：{{ address }}</h1>
          <button @click="showName">显示学校名称</button>
          </div>
        `,
    data() {
      return {
        schoolName: '学校',
        address: '地址'
      }
    },
    methods: {
      showName() {
        alert(this.schoolName)
      }
    }
  })
  const student = Vue.extend({
    template: `
          <div>
          <h2>学生姓名：{{ studentName }}</h2>
          <h2>学生年龄：{{ age }}</h2>
          </div>
        `,
    data() {
      return {
        studentName: '姓名',
        age: 18
      }
    }
  })

  // 注册组件
  new Vue({
    el: '#root',
    components: {
      school,
      student
    },
    data: {
      msg: 'Hello Vue!'
    }
  })
</script>
```

### 使用组件的步骤：

* 定义组件：使用 `Vue.extend(options)` 创建组件，使用 `template` 配置组件结构。
* 注册组件：使用 `Vue.component(componentName, component)` 全局注册组件，使用 `components` 选项局部注册组件。
* 使用组件：通过编写组件标签的方式使用组件。

### VueComponent

* 组件本质是一个 `Vue.extend` 生成的名为 `VueComponent` 的构造函数。
* Vue 在解析模版时会创建组件的实例对象，即调用 `new VueComponent(options)`。
* 每次调用 `Vue.extend` 都会产生一个全新的 `VueComponent`。
* `VueComponent.prototype.__proto__ === Vue.prototype` ，让组件实例对象可以访问到 Vue 原型上的属性和方法。

## 单文件组件

一个文件中只包含一个组件。

```vue

<template>
  <!--结构-->
</template>

<script>
// 交互
</script>

<style>
/*样式*/
</style>
```

## ref 属性

```vue

<template>
  <div>
    <h1 ref="h1">{{ msg }}</h1>
    <button @click="showDOM" ref="btn">展示 DOM 元素</button>
    <school ref="sch"></school>
  </div>
</template>

<script>
import School from "@/components/School";

export default {
  name: "App",
  components: {School},
  data() {
    return {
      msg: 'Vue'
    }
  },
  methods: {
    showDOM() {
      console.log(this.$refs.h1)
      console.log(this.$refs.btn)
      console.log(this.$refs.sch)
    }
  }
}
</script>
```

`ref` 用来给元素或子组件注册引用信息，使用在 HTML 元素上，引用指向 DOM 元素，使用在子组件上，引用指向组件实例。

注册方式：

```vue
<h1 ref="h1">{{ msg }}</h1>
<school ref="sch"></school>
```

使用方式：`this.$refs[refName]`。

## 配置项 `props`

```vue

<template>
  <div>
    <h1>{{ msg }}</h1>
    <h2>姓名：{{ name }}</h2>
    <h2>年龄：{{ myAge }}</h2>
    <h2>性别：{{ sex }}</h2>
    <button @click="updateAge">修改年龄</button>
  </div>
</template>

<script>
export default {
  name: "School",
  data() {
    return {
      msg: '学生信息',
      myAge: this.age
    }
  },
  methods: {
    updateAge() {
      this.myAge++
    }
  },
  // props: ['name', 'age', 'sex']

  // props: {
  //   name: String,
  //   age: Number,
  //   sex: String
  // }

  props: {
    name: {
      type: String,
      required: true
    },
    age: {
      type: Number,
      required: false,
      default: 18
    },
    sex: {
      type: String,
      required: true,
      // 自定义验证函数
      validator: (value) => {
        return '男' === value || '女' === value
      }
    }
  }
}
</script>
```

功能：让组件接收父组件传递的数据。
传递数据：

```vue
<demo name="xxx"/>
```

接收数据：

* 简单声明接收
  ```vue
  props:['name']
  ```

* 限制类型
  ```vue
  props:{
    name:String
  }
  ```

* 类型检测、自定义验证和默认值
  ```vue
  props:{
    name:{
      type: String,
      required: false,
      default: '老王'
    }
  }
  ```

`props` 是只读的，对 `props` 修改会触发警告，可以通过复制 `props` 内容到 `data` 中修改。

## mixin（混入）

```vue

<script>
import {mixin} from "@/mixin";

export default {
  name: "School",
  data() {
    return {
      name: '学生',
    }
  },
  mixins: [mixin]
}
</script>
```

```javascript
import {mixin2} from "@/mixin";

Vue.mixin(mixin2)
```

可以把多个组件公用的配置提取成一个混入对象。

定义混合对象：

```javascript
export const mixin = {
    methods: {
        showName() {
            alert(this.name)
        }
    },
    data() {
        return {
            x: 1,
            y: 2
        }
    },
}

export const mixin2 = {
    mounted() {
        console.log('mixin mounted')
    }
}
```

使用混入：

* 全局配置：`Vue.mixin()`
* 局部配置：`mixins:[]`

## 插件

用于增强 Vue，本质是包含 `install` 方法的一个对象。`install` 方法第一个参数是 `Vue`，后续参数是插件使用者传递的数据。

定义插件：

```javascript
{
    install(Vue, options)
    {
        Vue.filter()
        Vue.prototype.hello = () => {
        }
    }
}
```

使用插件：

```javascript
Vue.use(plugins)
```

## scoped 属性

当 `style` 标签有 `scoped` 属性时，它的 CSS 只作用于当前组件中的元素。

```css
<style scoped>
.demo {
  background-color: pink;
}
</style>
```

添加 `scoped` Vue 编译时会添加一个唯一的 `data` 属性（`data-v-22321ebb`）为组件内 CSS 指定作用域，`.demo` 会被编译为 ` .demo[data-v-22321ebb] {  background-color: pink; }`。

scoped 原理：

* 给 HTML 的 DOM 节点加一个不重复的 `data` 属性标识唯一性。
* 给每句 `css` 选择器的末尾加一个当前组件的 `data` 属性选择器来私有化样式。
* 如果组件内包含其它组件，只会给组件最外层标签加上当前组件的 `data` 属性。

scoped 穿透

如果需要父组件的样式覆盖子组件的样式有两种方式：

1. 使用两个 `style`，一个用于私有样式，一个用于共有样式。

   ```css
   <style scoped>
   .basic {
     background-color: blue;
   }
   </style>
   
   <style>
   .title {
     font-size: 40px;
   }
   </style>
   ```

2. 深度作用选择器（`>>>` 、 `/deep/` 或 `::v-deep`）

   ```css
   <style scoped>
   .basic {
     background-color: blue;
   }
   >>> .title {
     font-size: 40px;
   }
   </style>
   ```

   上述代码会编译成：

   ```css
   <style type="text/css">
   .basic[data-v-7ba5bd90] {
     background-color: blue;
   }
   [data-v-7ba5bd90] .title {
     font-size: 40px;
   }
   </style>
   ```


## 自定义事件

自定义事件是一种组件间通信方式，适用于子组件向父组件传递数据。

绑定自定义事件的方式：

* 在父组件中通过 `v-on:` 绑定事件

  ```vue
  <Student @showStudentName="showStudentName"/>
  ```

* 在父组件中通过 `$on` 函数绑定事件

  ```javascript
  this.$refs.student.$on('showStudentName', this.showStudentName)
  // 使用箭头函数保证 this 指向当前实例对象
  this.$refs.student.$once('method', (value) => {this.name = value})
  ```

* 如果希望自定义事件只能触发一次，可以使用 `once` 修饰符，或者使用 `$once` 方法。

触发自定义事件的方式：

```javascript
this.$emit('showStudentName', this.name)
```

解绑自定义事件的方式：

```javascript
this.$off('showStudentName')
this.$off(['m1', 'm2'])
```

组件上可以绑定原生 DOM 事件，但需要使用 `native` 修饰符。

## 全局事件总线

适用于任意组件间通信，使用前需安装全局事件总线：

```javascript
new Vue({
    beforeCreate() {
        Vue.prototype.$bus = this
    }
})
```

使用方式：

* 组件 A 希望接收数据，则在组件 A 中给 `$bus` 绑定自定义事件，事件回调写在组件 A 中。

  ```javascript
  methods(){
    handle(data){}
  },
  mounted(){
    this.$bus.$on('xxx', this.handle)
  }
  ```

* 其它组件调用组件 A 绑定的自定义事件提供数据。

  ```javascript
  this.$bus.$emit('xxx', data)
  ```

最好在组件的 `beforeDestroy` 生命周期钩子中，使用 `$off` 函数解绑当前组件绑定的自定义事件。

## 消息订阅与发布（pubsub-js）

PubSubJS 是一个用 JavaScript 编写的基于主题的发布订阅库。

安装 PubSubJS：

```bash
npm i pubsub-js
```

导入模块：

```javascript
import PubSub from 'pubsub-js'
```

使用 PubSubJS 发布与订阅：

```javascript
// 发布消息
PubSub.publish('removeTodo', this.todo.id)
// 订阅消息
this.pubId = PubSub.subscribe('removeTodo', this.removeTodo)
// 取消订阅
PubSub.unsubscribe(this.pubId)
```

## $nextTick

```javascript
this.myTodo.edit = true
this.$nextTick(function () {
  this.$refs.input.focus()
})
```

语法：`this.$nextTick(function(){})`。

作用：在下一次 DOM 更新结束后执行回调操作，用于数据改变后，要基于更新后的 DOM 进行操作时。

## slot 插槽

一种组件间通信方式，让父组件向子组件指定位置插入 HTML 结构，分为：默认插槽、具名插槽、作用域插槽。

### 默认插槽

```vue
// 子组件中
<template>
  <div class="category">
    <h2>{{ title }}</h2>
    <slot>插槽默认值</slot>
  </div>
</template>
// 父组件中
<Category title="美食分类">
  <img alt="美食" src="">
</Category>
```

### 具名插槽

```vue
// 子组件中
<template>
  <div class="category">
    <h2>{{ title }}</h2>
    <slot name="slot1">插槽 1 默认值</slot>
    <slot name="slot2">插槽 2 默认值</slot>
  </div>
</template>
// 父组件中
<Category title="美食分类">
  <img slot="slot1" alt="美食" src="">
  <template v-slot:slot2>
		<ol>
  		<li v-for="(f, index) in foods" :key="index">{{ f }}</li>
    </ol>
  </template>
</Category>
```

`v-slot` 只能添加在 `template` 上，插槽的默认名称是 `default`，`v-slot:default` 的缩写是 `#default`。

### 作用域插槽

数据在组件自身，但根据数据生成的结构需要父组件决定时使用作用域插槽。

```vue
// 子组件
<template>
  <div class="category">
    <h2>{{ title }}</h2>
    <slot :games="games">插槽默认值</slot>
  </div>
</template>
// 父组件
<Category title="游戏分类">
  <template v-slot:default="{games}">
		<ol>
  		<li v-for="(g, index) in games" :key="index">{{ g }}</li>
    </ol>
  </template>
</Category>
```

# Vue 脚手架

官网地址：https://cli.vuejs.org/zh/

Vue 脚手架是 Vue 官方提供的标准化开发工具。

## 创建脚手架项目

### 安装 Vue CLI

```bash
npm install -g @vue/cli
```

### 创建项目

```bash
vue create hello-vue
```

## Vue 项目结构

```bash
├── babel.config.js babel 的配置文件
├── jsconfig.json
├── node_modules
├── package-lock.json 包版本控制文件
├── package.json 应用包配置文件
├── public
|		|—— favicon.ico 页签图标
|		|—— index.html 主页面
├── src
|		|—— assets 存放静态资源
|		|		|—— logo.png
|		|—— components 存放组件
|		|—— App.vue
|		|—— main.js 入口文件
└── vue.config.js
```

### Render 函数

```javascript
new Vue({
    // render: h => h(App),
    render(createElement) {
        return createElement(App)
    }
}).$mount('#app')
```

* `vue.js` 是完整版的 Vue，包含核心功能和模版解析器。
* `vue.runtime.xxx.js` 是运行版 Vue，只包含核心功能。
* 由于运行版 Vue 没有模版解析器，所以在 `new Vue({})` 中不能使用 `template` 配置项，需要使用 `render`
  函数接收到的 `createElement` 函数去指定内容。

## 修改默认配置

使用 `vue inspect > output.js` 可以查看 Vue 脚手架的默认配置。

`vue.config.js` 是一个可选的配置文件，如果项目的根目录中存在这个文件，那么它会被 `@vue/cli-service` 自动加载。

## 配置代理

### 简单配置

在 `vue.config.js` 中添加如下配置：

```javascript
module.exports = {
    devServer: {
        proxy: 'http://localhost:9509'
    }
}
```

请求了前端不存在的资源时，会将请求转发给代理服务器。

### 完整配置

在 `vue.config.js` 中添加如下配置：

```javascript
module.exports = {
    devServer: {
        proxy: {
            '/api1': {
                target: 'http://localhost:9509',
                pathRewrite: {'^api1': ''},
                ws: true,
                changeOrigin: true
            }, '/api2': {
                target: 'http://localhost:9000',
                pathRewrite: {'^api2': ''},
            }
        }
    }
}
```

可以配置多个代理。

`target`：代理目标的 `basePath`。

`ws`：是否支持 websocket。

`changeOrigin`：修改请求头中 `host` 字段与目标服务器一致。
