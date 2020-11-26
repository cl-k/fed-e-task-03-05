# Vue 3.0 介绍

Vue.js 3.0 与 2.X 的区别

- 源码组织方式的变化

  全部采用 TS 重写，使用 Monorepo 的方式来组织项目结构，把独立的功能模块都提取到不同的包中

- Composition API

  用来解决 2.x 在开发大型项目时遇到超大组件，使用 options api 不好拆分和重用的问题

- 性能提升

  使用 Proxy 重写了响应式的代码，并且对编译器做了优化，重写了虚拟 DOM，从而让渲染和 update 的性能都有了大幅的提升

- Vite

  开发工具，在开发阶段不需要打包，可以直接运行项目，提升了开发效率

## Vue.js 3.0 源码组织方式

- 源码采用 TypeScript 重写（类型化的语言）

  提升了代码的可维护性，在编码过程中检查类型问题，比如函数传参的类型不匹配提示

- 使用 Monorepo 管理项目结构

  使用一个项目来管理多个包，把不同代码放到不同的 package 管理，每个功能模块划分明确，模块之间的依赖关系也很明确，并且每个功能模块都可单独测试，单独发布，单独使用

### packages 目录结构

```
|-- packages
  |-- compiler-core 和平台无关的编译器
  |-- compiler-dom  浏览器平台下的编译器，依赖于 compiler-core
  |-- compiler-sfc  单文件组件编译器，依赖于 compiler-core 和 compiler-dom
  |-- compiler-ssr  服务端渲染的编译器，依赖于 compiler-dom
  |-- reactivity    数据响应式系统
  |-- runtime-core  和平台无关的运行时
  |-- runtime-dom   针对浏览器的运行时，处理原生 DOM API 和 事件等
  |-- runtime-test  为测试所编写的轻量级运行时，由于它渲染出来的 DOM 树其实是一个 js 对象，所以这个运行时可以运行在所有的 js 环境里，可以用它来测试渲染是否正确，还可以用于序列化 DOM，触发 DOM 事件，以及记录更新中的某次 DOM 操作
  |-- server-renderer  用于服务端渲染
  |-- shared           vue 内部使用的公共 api
  |-- size-check       私有的包，不会发布到 npm,作用是在 tree-sharking 后检查包的大小
  |-- template-explorer  浏览器里运行的实时编译组件，它会输出 render 函数
  |-- vue  用来构建完整版的 vue,依赖于 compiler 和 runtime
```

## Vue.js 3.0 不同构建版本

构建时与 Vue2 类似，都构建了不同的版本，可以在不同的场合下使用。和 Vue2 不同的是 Vue3 中不再构建 UMD 模块化的方式，因为 UMD 模块化的方式会让代码有更多的冗余。Vue3.0 支持多种模块化的方式，Vue 3.0 的构建版本中把 cjs，es module 和自执行函数的方式分别打包到了不同的文件当中

### 构建版本

存放在 packages/vue 中

- cjs（Common JS 的模块化方式）

  - vue.cjs.js
  - vue.cjs.prod.js

- global

  - vue.global.js
  - vue.global.prod.js
  - vue.runtime.global.js
  - vue.runtime.global.prod.js

  这四个文件都可以在浏览器中直接通过 script 标签来导入 ，导入之后会增加一个全局的 vue 对象

- browser

  - vue.esm-browser.js
  - vue.esm-browser.prod.js
  - vue.runtime.esm-browser.js
  - vue.runtime.esm-browser.prod.js

  原生模块化的方式，在浏览器中可以通过 <script type="module"> 的方式来导入

- bundler

  - vue.esm-bundler.js
  - vue.runtime.esm-bundler.js

  这两个文件没有打包所有的代码，需要配合打包工具来使用。使用 es module 的方式，内容通过 import 导入了 runtime-core

## Composition API 设计动机

查看官方提供的 RFC

- RFC(Request For Comments)
  - https://github.com/vuejs/rfcs
- Compositon API RFC
  - https://composition-api.vuejs.org/

### 设计动机

- Options API

  - 包含一个描述组件选项（data、methods、props 等）的对象
  - Options API 开发复杂组件，同一个功能逻辑的代码被拆分到不同选项
  - 使用 Options API 还难以提取组件中可重用的逻辑

  Options API Demo

  ```js
  export default {
    data () {
      return {
        position: {
          x: 0,
          y: 0
        }
      }
    },
    created () {
      window.addEventListener('mousemove', this.handle)
    },
    destroyed () {
      window.removeEventListener('mousemove', this.handle)
    },
    methods: {
      handle (e) {
        this.position.x = e.pageX
        this.position.y = e.pageY
      }
    }
  }
  ```

  

- Composition API

  - Vue.js 3.0 新增的一组 API
  - 一组基于函数的 API
  - 可以更灵活的组织组件的逻辑

  Composition API Demo

  ```js
  import { reactive, onMounted, onUnmounted } from 'vue'
  function useMousePosition () {
      const position = reactive({
          x: 0,
          y: 0
      })
      const update = (e) => {
          position.x = e.pageX
          position.y = e.pageY
      }
     	onMounted(() => {
          window.addEventListener('mousemove', update)
      })
      onUnmounted(() => {
          window.removeEventListener('mousemove', update)
      })
      return position
  }
  
  export default {
      setup () {
          const position = useMousePosition()
          return {
              position
          }
      }
  }
  ```

## Vue.js 3.0 性能提升

- 响应式系统升级
- 编译优化
- 源码体积的优化

### 响应式系统升级

- Vue.js 2.x 中响应式系统的核心 defineProperty
- Vue.js 3.0 中使用 Proxy 对象重写响应式系统
  - 可以监听动态新增的属性
  - 可以监听删除的属性
  - 可以监听数组的索引和 length 属性

### 编译优化

- Vue.js 2.x 中通过标记静态根节点，优化 diff 的过程
- Vue.js 3.0 中标记和提升所有的静态根节点，diff 的时候只需要对比动态节点内容
  - Fragments（升级 vetur 插件）
  
    Vue3 中不在要求模版的跟节点必须是只能有一个节点。跟节点和和 render 函数返回的可以是纯文字、数组、单个节点，如果是数组，会自动转化为 Fragments。
  
  - hoistStatic 静态节点提升
  
    当使用 hoistStatic 时，所有静态的节点都被提升到 render 方法之外。这意味着，他们只会在应用启动的时候被创建一次，而后随着每次的渲染被不停的复用
  
  - Patch flag
  
    在 Vue3.0 中，在这个模版编译时，编译器会在动态标签末尾加上类似 /_ Text _/ PatchFlag。只能带 patchFlag 的 Node 才被认为是动态的元素，会被追踪属性的修改。并且 PatchFlag 会标识动态的属性类型有哪些，比如这里 的 TEXT 表示只有节点中的文字是动态的。
  
    每一个 Block 中的节点，就算很深，也是直接跟 Block 一层绑定的，可以直接跳转到动态节点而不需要逐个逐层遍历。
  
  - 缓存事件处理函数

### 优化打包体积

- Vue.js 3.0 中移除了一些不常用的 API
  
  - 例如：inline-template、filter 等
  
- Tree-shaking

  利用 ES6 模块的静态引用特性，在编译时正确的判断到底加载了哪些代码。对代码全局做一个分析，找到那些没用被用到的模块、函数、变量，并把这些去掉。

## Vite

ES Module

- 现代浏览器都支持 ES Module (IE 不支持)

- 通过下面的方式加载模块

  - <script type="module" src="..."></script>

- 支持模块的 script 默认延迟加载

  - 类似于 script 标签设置 defer
  - 在文档解析完成后，触发 DOMContentLoaded 事件前执行

### Vite as Vue-CLI

- Vite 在开发模式下不需要打包可以直接运行
- Vue-CLI 开发模式下必须对项目打包才可以运行
- Vite 在生产环境下使用 Rollup 打包
  - 基于 ES Module 的方式打包。体积更小
- Vue-CLI 使用 Webpack 打包

### Vite 特点

- 快速冷启动
- 按需编译
- 模块热更新

### Vite 创建项目

- Vite 创建项目

  ```bash
  $ npm init vite-app <project-name>
  $ cd <project-name>
  $ npm install
  $ npm run dev
  ```

- 基于模板创建项目

  ```bash
  $ npm init vite-app --template react
  $ npm init vite-app --template preact
  ```

-------

# Composition API

createApp 函数，用来创建 Vue 对象

setup 函数，它是 composition api 的入口

reactive 函数，用来创建响应式对象

## 生命周期钩子函数

如何在 setup 中使用生命周期钩子函数

| Options API     | Hook inside inside (setup) |
| --------------- | -------------------------- |
| beforeCreate    | Not needed*                |
| created         | Not needed*                |
| beforeMount     | onBeforeMount              |
| mounted         | onMounted                  |
| beforeUpdate    | onBeforeUpdate             |
| updated         | onUpdated                  |
| beforeUnmount   | onBeforeUnmount            |
| unmounted       | onUnmounted                |
| errorCaptured   | onErrorCaptured            |
| renderTracked   | onRenderTracked            |
| renderTriggered | onRenderTriggered          |

unmounted 对应 2.x 的 destroy 

## reactive/toRefs/ref

这三个函数都是创建响应式数据的

reactive 函数，是把一个对象转换为响应式数据

toRefs 函数，可以把一个响应式对象中的所有属性都转换为响应式的

toRefs 要求传入的参数必须是一个代理对象，它内部会创建一个新的对象，然后会遍历所传入的代理对象的所有属性，把所有属性的值都转换为响应式对象，然后再挂载到新创建的对象上，最后把这个新创建的对象返回

ref 函数，作用是把普通数据转化为响应式数据。基本数据类型存储的是值，它不可能是响应式数据。响应式数据需要通过 getter 收集依赖，通过 setter 触发更新。ref 内部会创建一个具有 value 属性的对象，该对象的 value 具有 getter 和 setter

## computed

计算属性，作用是简化模板中的代码，可以缓存计算的结果，当数据变化后，会重新计算。

- 第一种用法

  - watch(() => count.value + 1)

- 第二中用法

  ```js
  const count = ref(1)
  const plusOne = computed({
      get: () => count.value + 1,
      set: val => {
          count.value = val - 1
      }
  })
  ```

## Watch

侦听器，和 computed 类似，监听响应式数据的变化，执行相应的回调函数，可以获取到监听数据的新值和旧值

- Watch 的三个参数
  - 第一个参数：要监听的数据
  - 第二个参数：监听到数据变化后执行的函数，这个函数有两个参数分别是新值和旧值
  - 第三个参数：选项对象，deep（深度监听） 和 immediate（立即执行）
- Watch 的返回值
  - 取消监听的函数

watch 使用起来和过去的 this.$watch 是一样的，不同的是第一个参数不是字符串，而是 ref 或者 reactive 返回的对象

## WatchEffect

- 是 watch 函数的简化版本，也用来监视数据的变化
- 接受一个函数作为参数，监听函数内响应式数据的变化

## ToDoList 功能列表

- 添加待办事项
- 删除待办事项
- 编辑代办事项
  - 双击待办事项，展示编辑文本框
  - 按回车或者编辑文本框失去焦点，修改数据
  - 按 esc 取消编辑
  - 把编辑文本框清空按回车，删除这一项
  - 显示编辑文本框的时候获取焦点
- 切换待办事项
  - 点击 checkbox，改变所有待办项状态
  - All/Active/Completed
  - 其他
    - 显示未完成待办项个数
    - 移除所有完成的项目
    - 如果没有待办项，隐藏 main 和 footer
- 存储待办事项

-------

# Vue.js 3.0 响应式系统原理

简单说，响应式原理就是利用 `Proxy` 第二个参数 `handler` 也就是陷阱操作符中，拦截各种取值、赋值操作，依托 `track` 和 `trigger` 两个函数进行依赖收集和派发更新。

`track` 用来在读取时收集依赖。

`trigger` 用来在更新时触发依赖。

## 介绍

Vue.js 3 响应式回顾

- Proxy 对象实现属性监听
- 多层属性嵌套，在访问属性过程处理下一级属性
- 默认监听动态添加的属性
- 默认监听属性的删除操作
- 默认监听数组索引和 length 属性
- 可以作为单独的模块使用

核心方法

- reactive/ref/toRefs/computed
- effect
- track
- trigger

## Proxy 对象回顾

```js
'use strict'
// 问题1： set 和 deleteProperty 中需要返回布尔类型的值
// 在严格模式下，如果返回 false 的话会出现 Type Error 的异常
const target = {
  foo: 'xxx',
  bar: 'yyy'
}
// Reflect.getPrototypeOf()
// Object.getPrototypeOf()
const proxy = new Proxy(target, {
  get (target, key, receiver) {
    // return target[key]
    return Reflect.get(target, key, receiver)
  },
  set (target, key, value, receiver) {
    // target[key] = value
    return Reflect.set(target, key, value, receiver)
  },
  deleteProperty (target, key) {
    // delete target[key]
    return Reflect.deleteProperty(target, key)
  }
})
proxy.foo = 'zzz'
// delete proxy.foo
```

```js
// 问题2：Proxy 和 Reflect 中使用的 receiver
// Proxy 中 receiver：Proxy 或者继承 Proxy 的对象
// Reflect 中 receiver：如果 target 对象中设置了 getter，getter 中的 this 指向 receiver

const obj = {
  get foo() {
    console.log(this)
    return this.bar
  }
}

const proxy = new Proxy(obj, {
  get (target, key, receiver) {
    if (key === 'bar') { 
      return 'value - bar' 
    }
    return Reflect.get(target, key, receiver)
  }
})
console.log(proxy.foo)
```

## reactive

- 接收一个参数，判断这个参数是否是对象
- 创建拦截器对象 handler, 设置 get/set/deleteProperty

```
get
    收集依赖（track）
    返回当前 key 的值。
    如果当前 key 的值是对象，则为当前 key 的对象创建拦截器 handler, 设置 get/set/deleteProperty
    如果当前的 key 的值不是对象，则返回当前 key 的值
    
set
    设置的新值和老值不相等时，更新为新值，并触发更新（trigger）
    
deleteProperty
    当前对象有这个 key 的时候，删除这个 key 并触发更新（trigger）
```

- 返回 Proxy 对象

## 收集依赖

## effect&&track

effect 接收一个函数作为参数。作用是：访问响应式对象属性时去收集依赖

track

- 接收两个参数：target 和 key
- 如果没有 activeEffect，则说明没有创建 effect 依赖
- 如果有 activeEffect，则去判断 WeakMap 集合中是否有 target 属性，

```
WeakMap 集合中没有 target 属性，则 set(target, (depsMap = new Map()))
WeakMap 集合中有 target 属性，则判断 target 属性的 map 值的 depsMap 中是否有 key 属性
depsMap 中没有 key 属性，则 set(key, (dep = new Set()))
depsMap 中有 key 属性，则添加这个 activeEffect
```

trigger

- 判断 WeakMap 中是否有 target 属性

```
WeakMap 中没有 target 属性，则没有 target 相应的依赖
WeakMap 中有 target 属性，则判断 target 属性的 map 值中是否有 key 属性，有的话循环触发收集的 effect()
```

## ref

reactive vs ref

- ref 可以把基本数据类型数据，转成响应式对象
- ref 返回的对象，重新赋值成对象也是响应式的
- reactive 返回的对象，重新赋值丢失响应式
- reactive 返回的对象不可以解构

## toRefs

## computed

-------

# Vite 实现原理

## Vite

- Vite 是一个面向现代浏览器的一个更轻、更快的 Web 应用开发工具
- 它基于 ECMAScript 标准原生模块系统（ES Modules）实现
- 它的出现是为了解决 webpack 在开发阶段使用 dev-server 冷启动阶段时间过长以及 webpack HMR 热更新反应慢的问题

Vite 项目依赖

- Vite
- @vue/compiler-sfc

基础使用

- vite serve 开启一个用于开发的 web 服务器，在启动服务器的时候不需要编译所有的代码文件，启动速度快
- vite build

HMR

- Vite HMR
  - 立即编译当前所修改的文件，所以响应速度快
- Webpack HMR
  - 会自动以这个文件为入口重写 build 一次，所有涉及到的依赖也都会被加载一遍

Build

- vite build
  - Rollup
  - Dynamic import
    - Ployfill

打包 or 不打包

- 使用 Webpack 打包的两个原因：
  - 浏览器环境并不支持模块化
  - 零散的模块文件会产生大量的 HTTP 请求

Vite 开箱即用

- TypeScript - 内置支持
- less/sass/styles/postcss - 内置支持（需要单独安装）
- JSX
- Web Assembly

Vite 特性

Vite 优势在于提升开发体验

- 快速冷启动
- 模块热更新
- 按需编译
- 开箱即用

## Vite 实现原理 - 静态 Web 服务器

Vite 核心功能

- 静态 Web 服务器
- 编译单文件组件
  - 拦截浏览器不识别的模块，并处理
- HMR

当启动 Vite 的时候，首先会将当前项目目录作为静态文件服务器的根目录。静态文件服务器会拦截部分请求，例如，当请求单文件组建的时候会实时编译，以及处理其他浏览器不能识别的模块。通过 web socket 实现 HMR

```js
#!/usr/bin/env node
const Koa = require('koa')
const send = require('koa-send')

const app = new Koa()

// 1. 开启静态文件服务器
app.use(async (ctx, next) => {
  await send(ctx, ctx.path, { root: process.cwd(), index: 'index.html' })
  await next()
})

app.listen(3000)
console.log('server running @ http://localhost:3000')
```

测试之前先使用 npm link 链接到npm 安装目录里面

## Vite 实现原理 - 修改第三方模块的路径

需要创建两个中间件，一个中间件是把加载第三方模块的 import 中的路径改为加载 /@modules/<module-name>，另一个中间件是当请求过来之后，判断请求路径中是由有 /@modules/<module-name>，如果有的话，去 node_modulus 目录中加载对应的模块

```js
// 2. 修改第三方模块的路径
app.use(async (ctx, next) => {
  if (ctx.type === 'application/javascript') {
    const contents = await streamToString(ctx.body)
    // import vue from 'vue' 需要处理这种情况
    // import App from './App.vue'
    ctx.body = contents.replace(/(from\s+['"])(?!\.\/)/g, '$1/@modules/')
  }
})
```

## Vite 实现原理 - 加载第三方模块

判断请求路径中是由有 /@modules/<module-name>，如果有的话，去 node_modulus 目录中加载对应的模块

```js
// 3. 加载第三方模块
app.use(async (ctx, next) => {
  // ctx.path --> /@modules/vue
  if (ctx.path.startsWith('/@modules/')) {
    const moduleName = ctx.path.substr(10)
    const pkgPath = path.join(process.cwd(), 'node_modules', moduleName, 'package.json')
    const pkg = require(pkgPath)
    ctx.path = path.join('/node_modules', moduleName, pkg.module)
  }
  await next()
})

```

## Vite 实现原理 - 编译单文件组件

浏览器无法处理在 main.js 中使用 import 加载的单文件组件模块和样式模块，浏览器只能处理 js 模块。所以通过 import 加载的其他模块都需要在服务器端处理。当请求单文件组件的时候，需要在服务器上把单文件组件编译成 js 模块，然后返回给浏览器

