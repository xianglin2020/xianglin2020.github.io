---
title: Vue3 基础
date: 2022-12-28 22:14:54
tags: [vue]
categories: [learn]
description: Vue3 基础知识与语法。
---

# Vue3 基础

Vue3：https://cn.vuejs.org/。

## 创建 Vue3 工程

### 使用 vue-cli 创建

使用如下命令

```bash
vue create hello-vue3-cli
```

选择 `[Vue 3]` 即可

![image-20221231114751200](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202301/01215907.png)

### 使用 vite 创建

官网：https://cn.vitejs.dev/。

vite 优势：

* 开发环境中，无需打包操作，可快速冷启动。
* 轻量快速的热重载（HMR）。
* 真正的按需编译，不再等待整个应用编译完成，

使用如下命令（https://cn.vuejs.org/guide/quick-start.html#creating-a-vue-application）

```bash
npm init vue@latest
```

按命令行提示填入项目名称和选择可选功能即可。

也可以使用如下命令（https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project）

```bash
npm create vite@latest vue-vite -- --template vue
```

## 常用 `Composition API`

## `setup`

`setup` 是 Vue3 中一个新配置项，值为一个函数。

组件中所用到的数据方法等均要配置在 `setup` 中。

`setup` 函数的两种返回值：

* 返回对象，则对象中的属性方法在模板中可以直接使用。

  ```vue
  <template>
    <h1>一个人的信息</h1>
    <h2>姓名：{{ name }}</h2>
    <h2>年龄：{{ age }}</h2>
    <button @click="sayHello">说话</button>
  </template>
  
  <script>
  export default {
    name: 'App',
    components: {},
    setup() {
      // 数据
      let name = '张三'
      let age = 18
  
      // 方法
      function sayHello() {
        alert(`姓名：${name}，年龄：${age}`)
      }
  
      return {
        name,
        age,
        sayHello,
      }
    }
  }
  </script>
  ```

  

* 返回一个渲染函数，则可以自定义展示内容。

  ```javascript
  import {h} from 'vue'
  return () => {
    return h('h1', '渲染函数')
  }
  ```

`setup` 的执行时机，在 `beforeCreate` 之前执行，此时 `this` 的值是 `undifined`。

`setup` 的参数

* `props`：值为对象，包含组件外部传递，且组件内使用 `props` 配置项声明接收的属性。
* `context`：上下文对象：
  * `attrs`：值对对象，包含组件外部传递，但没有在 `props` 配置中声明接收的属性。
  * `slots`：收到的插槽内容。
  * `emit`：触发自定义事件的函数，相当于 `this.$emit`。

### `ref` 函数

作用：定义一个响应式数据。

语法：`const xxx = ref(initValue)`

* 创建一个包含响应式数据的引用对象。
* JavaScript 中操作数据：`xxx.value`。
* 模版中读取数据不需要 `.value`，直接使用 `<div>{{ xxx }}</div>`

备注：

* 接收的参数可以是基本类型，也可以是对象类型。
* 基本类型的数据实现响应式依然借助 `Object.defineProperty()` 的 `get` 与 `set` 完成。
* 对象类型的数据实现响应式借助 Vue3 中的 `reactive` 函数。

### `reactive` 函数

作用：定义一个对象类型的响应式数据。

语法：`const obj = reactive(srcObj)` 接收一个对象或数组，返回一个代理对象（`Proxy` 对象）。

`reactive` 定义的响应式数据是“深层次的”。

内部基于 ES6 的 `Proxy` 实现，通过代理对象操作源对象的内部数据。

### Vue3 响应式原理

通过 `Proxy` 拦截对象中任意属性的变化，包括属性值的读写、属性的添加删除等。

通过 `Reflect` 对源对象的属性进行操作。

```javascript
let person = {
  name: '张三',
  age: 18
}
const p = new Proxy(person, {
  get(target, propName) {
    console.log(`读取 p 的 ${propName} 属性`)
    return Reflect.get(target, propName)
  },
  set(target, propName, newValue) {
    console.log(`修改或增加 p 的 ${propName} 属性`)
    return Reflect.set(target, propName, newValue)
  },
  deleteProperty(target, propName) {
    console.log(`删除 p 的 ${propName} 属性`)
    return Reflect.deleteProperty(target, propName)
  }
})
```

### 计算属性与监视

#### `computed` 方法

```vue
<template>
  <h1>一个人的信息</h1>
  姓：<input type="text" v-model="person.firstName">
  名：<input type="text" v-model="person.lastName">
  <br>
  <input type="text" v-model="person.fullName">
</template>

<script>
import {reactive, computed} from "vue";

export default {
  name: 'Demo',
  setup() {
    let person = reactive({
      firstName: '张',
      lastName: '三'
    })

    // 计算属性（简写）
    // person.fullName = computed(() => {
    //   return person.firstName + '-' + person.lastName
    // })
    person.fullName = computed({
      get() {
        return person.firstName + '-' + person.lastName
      },
      set(value) {
        let arr = value.split('-');
        person.firstName = arr[0]
        person.lastName = arr[1]
      }
    })

    return {
      person,
    }
  }
}
</script>

```

#### `watch` 方法

```javascript
import {reactive, ref, watch} from "vue";

export default {
  name: 'Demo',
  setup() {
    let sum = ref(0)
    let msg = ref('你好')

    // 监视 ref 定义的一个响应式数据
    watch(sum, (newVal, oldVal) => {
      console.log('sum 改变', newVal, oldVal)
    })
    // 监视 ref 定义的多个响应式数据
    watch([sum, msg], (newVal, oldVal) => {
      console.log('sum/msg 改变', newVal, oldVal)
    })

    let person = reactive({
      name: '张三',
      age: 18,
      job: {
        j1: {
          salary: 20
        }
      }
    })
    // 监视 reactive 定义的响应式数据（深层监视），此处 newVal 和 oldVal 是相同的，因为它们是同一个对象
    watch(person, (newVal, oldVal) => {
      console.log('person 改变', newVal, oldVal)
    })
    // 监视 reactive 定时响应式数据中的某些属性
    watch(() => person.job, (newVal, oldVal) => {
      console.log('person job 改变', newVal, oldVal)
    }, {
      deep: true
    })

    return {
      sum,
      msg,
      person
    }
  }
}
```

### `watchEffect` 函数

`watchEffect` 会立即执行一遍回调函数，如果这时函数产生了副作用，Vue 会自动追踪副作用的依赖关系，自动分析出响应源。

```javascript
// 当 value 或 salary 变化时，再次执行回调
watchEffect(() => {
  const x1 = sum.value
  const x2 = person.job.j1.salary

  console.log('watchEffect', x1, x2)
})
```

### 生命周期

配置式生命周期钩子

`beforeDestroy` --> `beforeUnmount`、`destroyed` --> `Unmounted`。

组合式生命周期钩子

```javascript
import {
  ref, onBeforeMount, onMounted, onBeforeUpdate,
  onUpdated, onBeforeUnmount, onUnmounted
} from "vue";

export default {
  name: 'Demo',
  setup() {
    let sum = ref(0)

    // Composition API 生命周期
    onBeforeMount(() => {
      console.log('onBeforeMount')
    })
    onMounted(() => {
      console.log('onMounted')
    })
    onBeforeUpdate(() => {
      console.log('onBeforeUpdate')
    })
    onUpdated(() => {
      console.log('onUpdated')
    })
    onBeforeUnmount(() => {
      console.log('onBeforeUnmount')
    })
    onUnmounted(() => {
      console.log('onUnmounted')
    })

    return {
      sum
    }
  },
  // beforeCreate() {
  //   console.log('beforeCreate')
  // },
  // created() {
  //   console.log('created')
  // },
  // beforeMount() {
  //   console.log('beforeMount')
  // },
  // mounted() {
  //   console.log('mounted')
  // },
  // beforeUpdate() {
  //   console.log('beforeUpdate')
  // },
  // updated() {
  //   console.log('updated')
  // },
  // beforeUnmount() {
  //   console.log('beforeUnmount')
  // },
  // unmounted() {
  //   console.log('unmounted')
  // }
}
```

### 自定义 `hook` 函数

本质是一个函数，对 `setup` 函数中使用的 Composition API 进行封装。类似于 Vue2 中的 `mixin`，作用是复用代码，让 `setup` 中的逻辑更清晰。

在 `/src/hooks/usePoint.js` 中定义一个 `hook` 函数

```javascript
import {onBeforeUnmount, onMounted, reactive} from "vue";

export default function () {
    let point = reactive({
        x: 0,
        y: 0
    })
    const savePoint = (e) => {
        point.x = e.pageX
        point.y = e.pageY
    };
    onMounted(() => {
        window.addEventListener('click', savePoint)
    })
    onBeforeUnmount(() => {
        window.removeEventListener('click', savePoint)
    })

    return point
}

```

在组件中使用 `hook` 

```javascript
import {ref} from "vue";
import usePoint from "@/hooks/usePoint";

export default {
  name: 'Demo',
  setup() {
    let sum = ref(0)

    let point = usePoint();

    return {
      sum,
      point
    }
  },
}
```

### `toRef` 函数

作用：创建一个 `ref` 对象（`ObjectRefImpl`），其 `value` 值执行另一个对象中的属性。

语法：`let salary = toRef(person.job.j1, 'salary')`或者 `let x = toRefs(person)`。

用于将响应式对象中的某些属性单独提供给外部使用。

```vue
<template>
  <h4>{{ person }}</h4>
  <h2>姓名：{{ name }}</h2>
  <h2>年龄：{{ age }}</h2>
  <h2>薪资：{{ job.j1.salary }}</h2>
  <button @click="name += '-'">修改姓名</button>
  <button @click="age++">增长年龄</button>
  <button @click="salary++">涨薪</button>
</template>

<script>
import {reactive, toRef, toRefs} from "vue";

export default {
  name: 'Demo',
  setup() {
    let person = reactive({
      name: '张三',
      age: 18,
      job: {
        j1: {
          salary: 20
        }
      }
    })

    // ObjectRefImpl
    let salary = toRef(person.job.j1, 'salary')
    let x = toRefs(person)
    return {
      person,
      salary,
      ...x
    }
  }
}
</script>
```

## 其它 `Composition API`

`shallowReactive()`：`reactive()` 的浅层作用形式，只有根级别的属性是响应式的。

`shallowRef()`：`ref()`  的浅层作用形式，不会被深层递归地转为响应式只有对 `.value` 的访问是响应式的。

`readonly()`：接收一个对象或者 `ref`，返回一个原值的只读代理。

`shallowReadonly()`：与 `readonly()` 相比，只有根层级的属性变为只读。

`toRaw()`：根据一个 Vue 创建的代理返回其原始对象，可以用于临时读取而不引起代理访问/跟踪开销，或是写入而不触发更改的特殊方法。

`markRaw()`：将一个对象标记为不可被转为代理。

### `customRef`

作用：创建一个自定义的 `ref`，显式声明对其依赖追踪和更新触发的控制方式。

文档地址：https://cn.vuejs.org/api/reactivity-advanced.html#customref。

```javascript
// 创建一个防抖 ref
function myRef(value, delay) {
  let timer
  return customRef((track, trigger) => {
    return {
      get() {
        track()
        return value
      },
      set(newVal) {
        clearTimeout(timer)
        timer = setTimeout(() => {
          value = newVal
          trigger()
        }, delay)
      }
    }
  })
}

let keyword = myRef('hello', 500)
```

### 响应式数据判断

`isRef()`：检查某个值是否为 `ref`。

`isReactive()`：检查一个对象是否是由 `reactive()` 或 `shallowReactive()` 创建的代理。

`isReadonly()`：检查传入的值是否为只读对象，通过 `readonly()` 和 `shallowReadonly()` 创建的代理都是只读的。

`isProxy()`：检查一个对象是否是由 `reactive()`、`readonly()`、`shallowReactive()` 或 `shallowReadonly()` 创建的代理。

### `provide` 和 `inject`

作用：实现祖和后代组件间通信。

父组件使用 `provide` 提供数据，后代组件中使用 `inject` 注入数据。

```javascript
// 父组件
let car = reactive({
  name: '奔驰',
  price: '40W'
})
provide('car', car)

// 后代组件
let car = inject('car')
```

## 组件

### `Fragment`

在 Vue2 中组件必须有一个根标签，在 Vue3 中，组件可以没有根标签，内部会将多个标签包含在 `Fragment` 虚拟元素中。

### `Teleport`

可以将一个组件内部的一部分模版“传送”到该组件的 DOM 结构外层的位置去。

`Teleport` 接收一个 `to` 选项来指定传送的目标。

```vue
<button @click="open = true">Open Modal</button>

<Teleport to="body">
  <div v-if="open" class="modal">
    <p>Hello from the modal!</p>
    <button @click="open = false">Close</button>
  </div>
</Teleport>

```

### `Suspense`

作用：等待一部组件渲染时提供一些默认内容，让应用有更好的体验。

异步引入组件

```javascript
import {defineAsyncComponent} from 'vue'
const Child = defineAsyncComponent(() => import('./components/Child.vue'))

```

使用 `Suspense` 包裹组件，并配置好 `default` 和 `fallback` 插槽的内容

```vue
<Suspense>
  <template v-slot:default>
<Child/>
  </template>
  <template v-slot:fallback>
<h3>加载中……</h3>
  </template>
</Suspense>
```

