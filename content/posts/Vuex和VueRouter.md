---
title: Vuex 和 Vue Router
date: 2022-12-24 17:13:56
tags: [vue]
categories: [learn]
description: Vuex 和 Vue Router 的使用。
---

# Vuex

与 Vue2 匹配的 Vuex3：https://v3.vuex.vuejs.org/zh/。

与 Vue3 匹配的 Vuex4：https://vuex.vuejs.org/zh/index.html。

## 概念

在 Vue 中实现集中式状态（数据）管理的一个 Vue 插件，对 Vue 应用中多组件的共享状态进行集中式的管理（读/写），也是一种适用于任意组件间通信的方式。

## Vuex 开发环境

安装 Vuex

```bash
npm i vuex@3
```

编写 `/src/store/index.js` 文件，创建 Vuex 中核心的 `store` 

```javascript
import Vuex from 'vuex'
import Vue from "vue";
// 应用 Vuex 插件
Vue.use(Vuex)

const actions = {}
const mutations = {}
const state = {}
// 导出 store
export default new Vuex.Store({
    actions,
    mutations,
    state
})
```

在 `main.js` 中创建 Vue 实例时传入 `store` 配置项

```javascript
import store from "@/store";
new Vue({
    el: '#app',
    store,
    render: h => h(App)
})
```

## 基本使用

使用 Vuex 完成求和功能。

```vue
// Count.vue 组件
<template>
	<div>
  	<h2>当前求和为{{ $store.state.sum }}</h2>
    <button @click="increment">+</button>
  </div>
</template>
<script>
export default {
  name: "Count",
  data() {
    return {
      n: 1
    }
  },
  methods: {
    increment() {
      this.$store.commit('JIA', this.n)
    },
    incrementWait() {
      this.$store.dispatch('jiaWait', this.n)
    },
  }
}
</script>
```

```javascript
// store/index.js
import Vuex from 'vuex'
import Vue from "vue";

Vue.use(Vuex)

const actions = {
    jiaWait(context, value) {
        setTimeout(() => {
            context.commit('JIA', value)
        }, 500)
    },
}
const mutations = {
    JIA(state, value) {
        state.sum += value
    }
}
const state = {
    sum: 0
}
// 导出 store
export default new Vuex.Store({
    actions,
    mutations,
    state
})
```

在组件中读取 Vuex 中的数据：`$store.state.sum`。

在组件中修改 Vuex 中的数据：`$store.dispatch(actionName, value)` 或 `$store.commit(mutationName, value)`。

## getters 的使用

当需要对 `state` 中数据加工后再使用时，在 Vuex 的 `store` 中定义 `getter` （可以认为是 `store` 的计算属性）。

```javascript
const getters = {
    bigSum(state) {
        return state.sum * 10
    }
}
export default new Vuex.Store({
    ...,
    getters
})
```

在组件中访问 `getters` 属性 ：`$store.getters.bigSum`。

## 辅助函数

使用 `mapState` 辅助函数生成计算属性，减少冗余代码。`mapGetters` 辅助函数将 `store` 中的 `getter` 映射为计算属性。

```javascript
import {mapState, mapGetters} from "vuex";

computed: {
  // he() {
  //   return this.$store.state.sum
  // },
  ...mapState({he: 'sum'}),
    // bigSum() {
    //   return this.$store.getters.bigSum
    // },
    ...mapGetters(['bigSum'])
}
```

使用 `mapActions` 辅助函数将组件的 `methods` 映射为 `store.dispatch` 调用，使用 `mapMutations` 辅助函数将组件中的 `methods` 映射为 `store.commit` 调用。

```javascript
methods: {
  // increment() {
  //   this.$store.commit('JIA', this.n)
  // },
  ...mapMutations({'increment': 'JIA'}),
    // decrement() {
    //   this.$store.commit('JIAN', this.n)
    // },
    ...mapMutations(['JIAN']),

    // incrementOdd() {
    //   this.$store.dispatch('jiaOdd', this.n)
    // },
    ...mapActions({'incrementOdd': 'jiaOdd'}),
    // incrementWait() {
    //   this.$store.dispatch('jiaWait', this.n)
    // },
    ...mapActions(['jiaWait'])
}
```

## 模块化和命名空间

Vuex 支持将 `store` 拆分成多个模块（module），每个模块拥有自己的 `state` 、`mutation`、`action`、`getter`，通过添加 `namespaced:true` 使模块成为带有命名空间的模块。

修改 `store.js` 配置，以开启命名空间

```javascript
import Vuex from 'vuex'
import Vue from "vue";
Vue.use(Vuex)
const countOptions = {
    namespaced: true,
    actions: {
        jiaOdd(context, value) {
            if (context.state.sum % 2) {
                context.commit('JIA', value)
            }
        },
        jiaWait(context, value) {
            setTimeout(() => {
                context.commit('JIA', value)
            }, 500)
        },
    },
    mutations: {
        JIA(state, value) {
            state.sum += value
        },
        JIAN(state, value) {
            state.sum -= value
        }
    },
    state: {
        sum: 0,
    },
    getters: {
        bigSum(state) {
            return state.sum * 10
        }
    }
}
const personOptions = {
    namespaced: true,
    actions: {},
    mutations: {
        ADD_PERSON(state, value) {
            state.personList.push(value)
        }
    },
    state: {
        personList: [
            {id: '001', name: '张三'}
        ]
    },
    getters: {}
}
// 导出 store
export default new Vuex.Store({
    modules: {
        countAbout: countOptions,
        personAbout: personOptions
    }
})
```

通过辅助函数使用时传入 `namespace` 参数

```javascript
  computed: {
    ...mapState('countAbout', {he: 'sum'}),
    ...mapState('personAbout', ['personList']),
    ...mapGetters('countAbout', ['bigSum'])
  },
  methods: {
    ...mapMutations('countAbout', {'increment': 'JIA'}),
    ...mapMutations('countAbout', ['JIAN']),

    ...mapActions('countAbout', {'incrementOdd': 'jiaOdd'}),
    ...mapActions('countAbout', ['jiaWait'])
  }
```

不通过辅助函数时读取 `state` 数据

```javascript
computed: {
  sum() {
    return this.$store.state.countAbout.sum
  }
}
```

不通过辅助函数访问 `getters`、`actions`、`mutations`

```javascript
computed: {
  bigSum(){
    return this.$store.getters["countAbout/bigSum"]
  }
}

this.$store.dispatch('countAbout/jiaOdd',value)
this.$store.commit('personAbout/ADD_PERSON', value)
```

# Vue Router

与 Vue2 匹配的 Vue Router3 ：https://v3.router.vuejs.org/zh/。

与 Vue3 匹配的 Vue Router4：https://router.vuejs.org/zh/。

## 基础理解

一个路由（`route`）就是一组映射关系（`key-value`），多个路由需要路由器（`router`）进行管理，前端路由中：`key` 是路径，`value` 是组件。

## 基本使用

安装 `vue-router`

```bash
npm i vue-router@3
```

在 `/src/route/index.js` 中创建路由器，编写 `router` 配置项

```javascript
import VueRouter from "vue-router";
import About from "@/components/About.vue";
import Home from "@/components/Home.vue";
// 创建一个路由器
export default new VueRouter({
    routes: [
        {
            path: '/about',
            component: About
        },
        {
            path: '/home',
            component: Home
        }
    ]
})
```

应用 `VueRouter` 插件，创建 Vue 实例时传入 `router` 配置项

```javascript
import Vue from 'vue'
import App from "@/App.vue";
import VueRouter from "vue-router";
import router from "@/router";

Vue.use(VueRouter)
new Vue({
    el: '#app',
    router,
    render: h => h(App)
})
```

使用 `<router-link>` 实现路由切换，使用 `<vue-view/>` 指定展示位置

```vue
<div class="row">
  <div class="col-xs-2 col-xs-offset-2">
    <div class="list-group">
      <!-- Vue 中使用 router-link 实现路由切换 -->
      <router-link to="/about" active-class="active" class="list-group-item">About</router-link>
      <router-link to="/home" active-class="active" class="list-group-item">Home</router-link>
    </div>
  </div>
  <div class="col-xs-6">
    <div class="panel">
      <div class="panel-body">
        <!-- 指定组件的呈现位置 -->
        <router-view/>
      </div>
    </div>
  </div>
</div>
```

### 注意点

* 路由组件通常存放在 `pages` 文件夹，一般组件通常存放在 `components` 文件夹。
* 路由切换时，组件默认被销毁，需要时重新挂载。
* 每个组件都有自己的 `$route` 属性，保存着自身的路由信息。
* 整个应用只有一个 `router`，通过 `$router` 属性获取。

## 多级路由

路由配置中使用 `children` 配置项

```javascript
import VueRouter from "vue-router";
import Home from "@/pages/Home.vue";
import Message from "@/pages/Message.vue";
import MessageDetail from "@/pages/MessageDetail.vue";
// 创建一个路由器
export default new VueRouter({
    routes: [
        {
            path: '/home',
            component: Home,
            children: [
                {
                    path: 'message',
                    component: Message,
                    children: [
                        {
                            path: 'detail',
                            component: MessageDetail
                        }
                    ]
                }
            ]
        }
    ]
})
```

`<router-link>` 中使用全路径名称

```vue
<router-link to="/home/news">News</router-link>
```

## 路由传参

### query 参数

字符串写法

```vue
<router-link :to="`/home/message/detail?id=${message.id}&title=${message.title}`">
  {{ message.title }}
</router-link>
```

对象写法

```vue
<router-link :to="{
  path: '/home/message/detail',
  query: {
    id: message.id,
    title: message.title
  }
}">
  {{ message.title }}
</router-link>
```

接收参数

```vue
<li>消息编号：{{ $route.query.id }}</li>
<li>消息标题：{{ $route.query.title}}</li>
```

### params 参数

配置路由时，申明接收 `params` 参数

```javascript
import VueRouter from "vue-router";
import Home from "@/pages/Home.vue";
import Message from "@/pages/Message.vue";
import MessageDetail from "@/pages/MessageDetail.vue";
// 创建一个路由器
export default new VueRouter({
    routes: [
        {
            path: '/home',
            component: Home,
            children: [
                {
                    path: 'message',
                    component: Message,
                    children: [
                        {
                            name: 'xq',
                            path: 'detail',
                            component: MessageDetail
                        }
                    ]
                }
            ]
        }
    ]
})
```

传递参数

```vue
<router-link :to="`/home/message/detail/${message.id}/${message.title}`">
     {{ message.title }}
</router-link>

<router-link :to="{
          name: 'xq',
          params: {
            id: message.id,
            title: message.title
          }
        }">
  {{ message.title }}
</router-link>
```

接收参数

```vue
<li>消息编号：{{ $route.params.id }}</li>
<li>消息标题：{{ $route.params.title}}</li>
```

## 命名路由

命名路由用于简化路由配置，使用 `name` 配置项指定路由名称

````javascript
import VueRouter from "vue-router";
import Home from "@/pages/Home.vue";
import Message from "@/pages/Message.vue";
import MessageDetail from "@/pages/MessageDetail.vue";
// 创建一个路由器
export default new VueRouter({
    routes: [
        {
            path: '/home',
            component: Home,
            children: [
                {
                    path: 'message',
                    component: Message,
                    children: [
                        {
                            name: 'xq',
                            path: 'detail',
                            component: MessageDetail
                        }
                    ]
                }
            ]
        }
    ]
})
````

在 `<router-link>` 中使用 `name` 配置项代替 `path` 

```vue
<router-link :to="{
  name: 'xq',
  query: {
    id: message.id,
    title: message.title
  }
}">
  {{ message.title }}
</router-link>
```

## 路由的 props 配置项

将组件和路由解耦，让组件更方便接收参数。

```json
{
  name: 'xq',
  path: 'detail/:id/:title',
  component: MessageDetail,
  // props: true,

  // props: {
  //     id: 'id',
  //     title: 'title'
  // },

  props: function (route) {
    return {
      id: route.params.id,
      title: route.params.title
    }
  }
}
```

### 布尔模式

如果 `props` 被设置为 `true` ，把路由收到的 `params` 参数以 `props` 形式传递给组件。

### 对象模式

如果 `props` 是对象，该对象被按原样设置为组件属性。

### 函数模式

函数返回值作为 `props` 传递给组件。

## 编程式路由导航

使用 `router` 实例方法代替 `<router-link>` 创建 `a` 标签来定义导航链接。

| 声明式                            | 编程式                |
| --------------------------------- | --------------------- |
| `<router-link :to="...">`         | `router.push(...)`    |
| `<router-link :to="..." replace>` | `router.replace(...)` |

```javascript
pushShow(message) {
  this.$router.push({
    name: 'xq',
    params: {
      id: message.id,
      title: message.title
    }
  })
},
replaceShow(message) {
  this.$router.replace({
    name: 'xq',
    params: {
      id: message.id,
      title: message.title
    }
  })
}
```

解决 `push` 和 `replace` 多次跳转到统一位置报错（`NavigationDuplicated`）

```javascript
import VueRouter from "vue-router";
const originalPush = VueRouter.prototype.push;
const originalReplace = VueRouter.prototype.replace;
VueRouter.prototype.push = function push(location) {
    return originalPush.call(this, location).catch(err => {
        if (err.name !== 'NavigationDuplicated') throw err
    });
}
VueRouter.prototype.replace = function replace(location) {
    return originalReplace.call(this, location).catch(err => {
        if (err.name !== 'NavigationDuplicated') throw err
    });
}
```

## 缓存路由组件

让不展示的路由组件保持挂载，不被销毁。

```vue
<keep-alive include="News">
  <router-view/>
</keep-alive>
// 缓存多个路由组件
<keep-alive :include="['News', 'Message']">
  <router-view/>
</keep-alive>
```

## Vue Router 生命周期钩子

用于捕获路由组件的激活状态

* `activated`：组件被激活时触发。
* `deactivated`：组件失活是触发。

## 路由守卫

### 全局守卫

```javascript
// 注册一个全局前置守卫
router.beforeEach((to, from, next) => {
    console.log('前置守卫', to, from)
    if (to.meta.isAuth) {
        if (localStorage.getItem('school') === 'school') {
            next();
        }
    } else {
        next();
    }
})
// 注册全局后置钩子
router.afterEach((to, from) => {
    console.log('后置守卫', to, from)
    document.title = to.meta.title
})
```

当路由切换时，全局前置守卫按照创建顺序调用。

### 独享守卫

```json
const router = new VueRouter({
    routes: [
         {
           name: 'xinwen',
           path: 'news',
           component: News,
           meta: {
             isAuth: true,
             title: '新闻'
           },
           beforeEnter(to, from, next) {
             console.log('独享前置守卫', to, from)
             if (localStorage.getItem('school') === 'school') {
               next();
             }
           }
         }
    ]
})

export default router
```

在路由配置上直接定义 `beforeEnter` 守卫。

### 组件内守卫

```javascript
  // 通过路由规则，进入该组件时调用
  beforeRouteEnter(to, from, next) {
    console.log('beforeRouteEnter')
    next()
  },
  // 通过路由规则，离开该组件时调用
  beforeRouteLeave(to, from, next) {
    console.log('beforeRouteLeave');
    next()
  }
```

### 完整流程

1. 导航被触发。
2. 在失活的组件里调用 `beforeRouteLeave` 守卫。
3. 调用全局的 `beforeEach` 守卫。
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫 。
5. 在路由配置里调用 `beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的组件里调用 `beforeRouteEnter`。
8. 调用全局的 `beforeResolve` 守卫。
9. 导航被确认。
10. 调用全局的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数，创建好的组件实例会作为回调函数的参数传入。
