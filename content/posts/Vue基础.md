---
title: Vue2 基础
date: 2022-08-11 21:22:49
tags: [vue]
categories: [learn]
description: Vue 基础知识与语法。
---

# Vue2 基础

Vue2：https://v2.cn.vuejs.org/

Vue 是一套用于构建用户界面的渐进式框架。

## Vue 特点

* 采用组件化模式，提高代码复用率，且让代码更好维护；
* 申明式编码，无需直接操作 DOM，提高开发效率；
* 使用虚拟 DOM 和 Diff 算法，尽量复用 DOM。

## Vue 实例

通过 `Vue` 函数创建 Vue 实例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>vue 实例</title>
    <script src="../js/vue.js"></script>
</head>

<body>
<div id="root">
    <h2>{{msg}}</h2>
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            msg: 'Vue 实例'
        }
    })
</script>
</body>
</html>
```

`el`：用于指定当前 Vue 实例为哪个容器服务，通常为 css 选择器字符串。

Vue 实例和容器是一一对应的。

### `el` 和 `data` 的两种写法

```vue

<div id="root">
<h1>Hello {{name}}</h1>
</div>

<script>
new Vue({
  el: '#root',
  data: {
    name: 'Vue'
  }
})
</script>


<div id="root1">
<h1>Hello {{name}}</h1>
</div>

<script>
new Vue({
  data() {
    return {
      name: 'Vue'
    }
  }
}).$mount('#root1')
</script>
```

`el` 的两种写法：

* `new Vue` 是指定配置属性
* 先创建 Vue 实例，通过 `$mount('#root1')` 指定 `el` 值

data 可以使用对象式或函数式写法；

由 Vue 管理的函数，不能使用箭头函数，箭头函数中的 `this` 不是 Vue 实例。

## 模板语法

```vue

<div id="root">
<h1>插值语法</h1>
<h3>Hello,{{name}}</h3>
<hr>
<h1>指令语法</h1>
<a href="https://www.bing.com">Bing</a>
<br>
<a v-bind:href="url">Bing</a>
</div>

<script>
Vue.config.productionTip = false

new Vue({
  el: '#root',
  data: {
    name: 'Vue',
    url: 'https://www.bing.com'
  }
})
</script>
```

### 插值语法

用于解析标签体内容，写法 `{{xx}}`，`xx` 是 js 表达式，且可以读取到 `data` 中的所有属性。

### 指令语法

用于解析标签，包括：标签属性、标签体内容、绑定事件等，`v-bind:href="url"` 简写为 `:href="url"`

## 数据绑定

```vue

<div id="root">
<label>
  单向数据绑定：
  <input type="text" v-bind:value="name">
  <input type="text" :value="name">
</label>
<br>
<label>
  双向数据绑定：
  <input type="text" v-model:value="name">
  <input type="text" v-model="name">
</label>
</div>

<script>
Vue.config.productionTip = false

new Vue({
  el: '#root',
  data: {
    name: 'Vue'
  }
})
</script>
```

Vue 中的数据绑定方式：

* 单向绑定 `v-bind`：数据从 `data` 流向页面
* 双向绑定 `v-model`：它负责监听用户的输入事件以更新数据
    * 双向绑定一般都应用在表单类元素上（如 `input` 、`select`）；
    * `v-model:value` 可以简写为 `v-model`，因为 `v-model` 默认收集 `value` 值。

## MVVM 模型

MVVM 模型：https://zh.wikipedia.org/wiki/MVVM

![MVVMPattern](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202208/172859.png)

![image-20220827172739393](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202208/172739.png)

MVVM 模型代表 M、V 和 VM

M：模型 `Model`：`data` 中的数据

V：视图 `View`：模板代码

VM：视图模型 `ViewModel`：Vue 实例

## 数据代理

### Object.defineProperty()

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty

语法：`Object.defineProperty(obj, prop, descriptor)`

参数：

`obj`：要定义属性的对象。

`prop`：要定义或修改的属性的名称。

`descriptor`：要定义或修改的属性描述符。

`descriptor` 的配置项：

* `value`：该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。 默认为 `undefined`
* `configurable`：配置为 `true` 时属性能从对象上删除，默认为 `false`；
* `enumerable`：配置为 `true` 时属性出现在对象的枚举属性中，默认为 `false`；
* `writable`：配置为 `true` 时，配置的 `value` 才能被赋值运算符改变，默认为 `false`；

### 数据代理

通过一个对象代理对另一个对象中属性的读写操作。

### Vue 中的数据代理

通过 `vm` 对象代理 `data` 对象中数据的读写操作，好处是可以方便的操作 `data` 中的数据。

基本原理：通过 `Object.defineProperty()` 把 `data` 对象中所有属性添加到 `vm` 中，为每一个添加到 `vm`
的属性都指定 `getter/setter` ，在 `getter/setter` 中读写 `data` 中对应的属性。

## Vue 中事件处理

### 监听事件

```vue

<div id="root">
<h1>Hello, {{msg}}</h1>
<!--    <button v-on:click="showInfo">点击提示信息</button>-->
<button @click="showInfo1">点击提示信息1</button>

<button @click="showInfo2(66,$event)">点击提示信息2</button>
</div>

<script>
new Vue({
  el: '#root',
  data: {
    msg: 'Vue'
  },
  methods: {
    showInfo1(event) {
      // console.log(event.target)
      // console.log(this)
      alert('点击')
    },
    showInfo2(number, e) {
      console.log(number, e)
    }
  }
})
</script>
```

使用 `v-on:xxx` 或 `@xxx` 绑定事件，事件回调配置在 `methods` 中。可以用特殊变量 `$event` 访问原始的 DOM 事件对象。

### 事件修饰符

```vue

<div id="root">
<!-- 阻止单击事件继续传播 -->
<button v-on:click.stop="doThis">点击</button>
<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit">
  <input type="submit" value="提交">
</form>
<!-- 点击事件将只会触发一次 -->
<a v-on:click.once="doThis"></a>
</div>

<script>
new Vue({
  el: '#root',
  methods: {
    doThis() {
      console.log('doThis')
    },
    onSubmit() {
      console.log('onSubmit')
    }
  }
})
</script>
```

`prevent`：同 `event.preventDefault()` ，阻止默认事件。

`stop`：同 `event.stopPropagation()` ，阻止事件冒泡。

`once`：事件只会触发一次。

事件修饰符可以连写：`@click.stop.prevent`

### 键盘事件

```vue

<div id="root">
<input type="text" placeholder="按下回车提示输入" @keyup="showInfo">
<input type="text" placeholder="制表符" @keydown.tab="showKeyCode">
</div>

<script>
Vue.config.productionTip = false

new Vue({
  el: '#root',
  methods: {
    showInfo(e) {
      if (e.keyCode === 13) {
        console.log(e.target.value)
      }
    },
    showKeyCode(e) {
      console.log(e.key, e.keyCode)
    }
  }
})
</script>
```

Vue 中常用的按键别名：

回车 => `enter` => `13`

删除 => `delete` （同时捕获“删除`8`”和“退格`46`”）

退出 => `esc` => `27`

空格 => `space` => `32`

制表符 => `tab` => `9`

上、下、左、右 => `up down left right`

Vue 未提供按键别名的按键，可以使用按键原始的 key 值绑定，但要转换为 kebab-case（短横线命名）。

系统修饰键：`ctrl`、`alt`、`shift` 和 `meta` 使用 `keydown` 正常触发事件，使用 `keyup` 需要再按下其它键，随后释放才触发事件。

## 计算属性与监视

### 计算属性

```vue

<div id="root">
<label>
  姓：
  <input type="text" v-model="firstName">
</label> <br><br>
<label>
  名：
  <input type="text" v-model="lastName">
</label> <br><br>
全名：<span>{{fullName}}</span>
</div>

<script>
Vue.config.productionTip = false

new Vue({
  el: '#root',
  data: {
    firstName: '张',
    lastName: '三'
  },
  computed: {
    fullName: {
      get() {
        return this.firstName + '-' + this.lastName
      },
      set(val) {
        let arr = val.split('-')
        this.firstName = arr[0]
        this.lastName = arr[1]
      }
    }
  }
})
</script>
```

通过已有的属性计算出新属性称为计算属性，底层借助了 `Object.defineproperty` 方法提供 `getter` 和 `setter`。

计算属性会缓存结果，只有在其依赖的属性变化时，计算属性才会重新计算。

### 监视属性

```vue

<div id="root">
<h2>今天天气很{{info}}</h2>
<button @click="changeWeather">切换天气</button>
</div>

<script>
Vue.config.productionTip = false

const vm = new Vue({
  el: '#root',
  data: {
    isHot: true
  },
  computed: {
    info() {
      return this.isHot ? '炎热' : '凉爽'
    }
  },
  watch: {
    isHot: {
      immediate: true,
      handler(newValue, oldValue) {
        console.log('isHot 被修改了', newValue, oldValue)
      }
    }
  },
  methods: {
    changeWeather() {
      this.isHot = !this.isHot
    }
  }
})
// 监视属性API
vm.$watch('isHot', {
  immediate: true,
  handler(newValue, oldValue) {
    console.log('isHot 被修改了', newValue, oldValue)
  }
})
</script>
```

当被监视的属性变化时，回调函数自动调用，或执行异步或耗时较长的操作。监视属性使用 `watch` 配置或使用 `vm.$watch` 配置监视。

`immediate`：立即触发回调函数

`deep`：深度监视，监视对象内部值的变化

监视对象的回调函数中 `newValue` 和 `oldValue` 是一致的，可以配合计算属性，序列化生成新对象避免此问题。

```vue

<div id="root">
<h2>numbers 的 a 是 {{numbers.a}}</h2>
<button @click="numbers.a++">点击 a + 1</button>
</div>

<script>
Vue.config.productionTip = false

const vm = new Vue({
  el: '#root',
  data: {
    numbers: {
      a: 1,
      b: 2
    }
  },
  computed: {
    newNumbers() {
      return JSON.parse(JSON.stringify(this.numbers))
    }
  },
  watch: {
    numbers: {
      handler(newValue, oldValue) {
        console.log('numbers 被改变了', newValue, oldValue)
      },
      deep: true
    },
    newNumbers(newVal, oldVal) {
      console.log('newNumbers 被改变了', newVal, oldVal)
    }
  },
})
</script>
```

## 绑定样式

### 绑定 class 样式

```vue

<style>
.basic {
}

.normal {
}

.happy {
}

.sad {
}

.s1 {
}

.s2 {
}

.s3 {
}
</style>

<div id="root">
<!-- 绑定 class 样式——字符串写法 -->
<div class="basic" :class="mood" id="demo" @click="changeMood">{{name}}</div>
<!-- 绑定 class 样式——数组写法 -->
<div class="basic" :class="classArr">{{name}}</div>
<!-- 绑定 class 样式——对象写法 -->
<div class="basic" :class="classObj">{{name}}</div>
</div>

<script type="text/javascript">
Vue.config.productionTip = false

new Vue({
  el: '#root',
  data: {
    name: 'Vue',
    mood: 'normal',
    classArr: ['s1', 's2', 's3'],
    classObj: {
      s1: true,
      s2: true,
      s3: true,
    }
  },
  methods: {
    changeMood() {
      this.mood = 'happy'
    }
  }
})
</script>
```

### 绑定 style 样式

```vue

<div id="root">
<!-- 绑定 style 样式——对象写法 -->
<div class="basic" :style="styleObj">{{name}}</div>
</div>

<script>
Vue.config.productionTip = false

const vm = new Vue({
  el: '#root',
  data: {
    styleObj: {
      fontSize: '40px'
    }
  }
})
</script>
```

## 条件渲染

```vue

<div id="root">
<h1 v-show="show">Hello,{{name}}</h1>

<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>

<template v-if="show">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
</div>

<script>
Vue.config.productionTip = false

new Vue({
  el: '#root',
  data: {
    name: 'Vue',
    show: true,
    type: 'D'
  }
})
</script>
```

* `v-if`：可以与 `v-else-if` 和 `v-else` 配合使用，可以与 `template` 配合使用渲染分组；
* `v-show`：元素被渲染，通过 `display` 样式切换，适用于非常频繁的切换。

## 列表渲染

```vue

<div id="root">
<!-- 遍历数组 -->
<h2>人员列表</h2>
<ul>
  <li v-for="(p, index) in persons" :key="p.id">
    {{index}}-{{p.name}}-{{p.age}}
  </li>
</ul>
<!-- 遍历对象 -->
<h2>汽车信息</h2>
<ul>
  <li v-for="(v, k) of car" :key="k">
    {{k}}: {{v}}
  </li>
</ul>
</div>

<script>
Vue.config.productionTip = false

new Vue({
  el: '#root',
  data: {
    persons: [
      {id: '001', name: '张三', age: 18},
      {id: '002', name: '李四', age: 19},
      {id: '003', name: '王五', age: 20},
    ],
    car: {
      name: 'A8',
      price: '70W',
      color: '黑色'
    }
  }
})
</script>
```

`v-for` 可以用于遍历数组、对象、字符串以及次数。

### key 的原理

`key` 是虚拟 DOM 对象的标识，当数据发生变化时，Vue 会根据新数据生成新的虚拟 DOM，随后 Vue 进行新旧虚拟 DOM 的差异对比。
虚拟 DOM 对比规则：

* 旧虚拟 DOM 中存在与新虚拟 DOM `key` 相同的元素，如果虚拟 DOM 中内容没变，直接复用之前的真实 DOM。如果虚拟 DOM
  中内容发生变化，生成新的真实 DOM。
* 旧虚拟 DOM 中不存在与新虚拟 DIM `key` 相同的元素，生成新的真实 DOM，渲染到页面中。

使用 `index` 作为 `key` 时对数据进行破环顺序的操作，会产生没有必要的真实 DOM 更新，效率较低。如果结构中还包含输入类的元素，会产生错误的
DOM 更新。

## Vue 监视数据的原理

* Vue 会监视 `data` 中所有层次的数据。
* 通过 `setter` 监视对象中的数据，且要在 `new Vue` 时传入监视的数据。对象中后追加的属性，Vue 默认不做响应式处理，可以使用如下
  API 为添加的属性做响应式：
  ```vue
  Vue.set(target, propertyName/index, value)
  vm.$set(target, propertyName/index, value)
  ```
* 通过包裹数组更新方法监视数组中的数据，即调用数组原生方法更新数据，然后重新解析模板。这些方法包括：`push`、`pop`、`shift`
  、`unshift`、`splice`、`sort` 和 `reverse`。
* `Vue.set()` 和 `vm.$set()` 不能给 `vm` 或 `vm` 的根数据对象(`_data`) 添加属性。

## 过滤器

```vue

<div id="root">
<h1>当前时间</h1>
<h2>时间戳：{{time}}</h2>
<h2>格式化：{{fmtTime}}</h2>
<h2>格式化：{{getFmtTime()}}</h2>
<h2>格式化：{{time | timeFormatter('YYYY年MM月DD日')}}</h2>
<h2>首字母大写：{{'hello' | capitalize}}</h2>
</div>

<script>
Vue.config.productionTip = false
Vue.filter('capitalize', function (val) {
  if (!val) {
    return ''
  }
  val = val.toString()
  return val.charAt(0).toUpperCase() + val.slice(1)
})

new Vue({
  el: '#root',
  data: {
    time: 1669816105394
  },
  computed: {
    fmtTime() {
      return dayjs(this.time).format('YYYY-MM-DD HH:mm:ss')
    }
  },
  methods: {
    getFmtTime() {
      return dayjs(this.time).format('YYYY-MM-DD HH:mm:ss')
    }
  },
  filters: {
    timeFormatter(value, pattern = 'YYYY-MM-DD HH:mm:ss') {
      return dayjs(value).format(pattern)
    }
  }
})
</script>
```

过滤器可用于一些简单的操作，比如文本格式化。过滤器可以用在 `{{}}` 插值和 `v-bind` 表达式中，过滤器跟在管道符号 `|` 后面。

使用过滤器的两种方式：

* 使用 `filters` 配置项在组件中定义本地过滤器。
* 在创建 Vue 实例前使用 `Vue.filter()` 定义全局过滤器。

过滤器默认接受管道符号前表达式的值作为第一个参数。

## 自定义指令

```vue

<div id="root">
<h2>当前的 n：<span v-text="n"></span></h2>
<h2>放大的 n：<span v-big-number="n"></span></h2>
<button @click="n++">点击+1</button>
<br>
<br>
<input type="text" v-fbind="n">
</div>

<script>
Vue.config.productionTip = false
// bind 和 update 触发相同行为
Vue.directive('big-number', function (element, binding) {
  element.innerText = binding.value * 10
})

new Vue({
  el: '#root',
  data: {
    n: 0
  },
  directives: {
    fbind: {
      bind(element, binding) {
        element.value = binding.value
      },
      inserted(element) {
        element.focus()
      },
      update(element, binding) {
        element.value = binding.value
      }
    }
  }
})
</script>
```

### 指令的两种注册方式

* 使用 `Vue.directive()` 注册全局指令。
* 使用 `directives` 配置项注册局部指令。

### 钩子函数

* `bind`：只调用一次，指令第一次绑定到元素时调用，做初始化设置。
* `inserted`：被绑定元素插入父节点时调用。
* `update`：所在组件的 VNode 更新时调用。

钩子函数的参数：

* `el`：指令所绑定的真实 DOM。
* `binding`：对象，包含一下属性：
    * `name`：指令名，不包括 `v-` 前缀。
    * `value`：指令绑定的值。
* ……

## Vue 生命周期

常用的生命周期钩子：

* `mounted`：发送请求、启动定时器、绑定自定义事件、订阅消息等初始化操作。
* `beforeDestroy`：清除定时器、解绑自定义事件、取消订阅消息等收尾工作。

关于销毁 Vue 实例：

* 销毁后自定义事件会失效、但原生 DOM 事件依然有效。
* 一般不会在 `beforeDestory` 中操作数据，即使操作了数据，也不会触发更新流程。
