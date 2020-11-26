# Vue.js 3.0 Composition API 及 3.0 原理剖析

##  1、Vue 3.0 性能提升主要是通过哪几方面体现的？

- 响应式系统的升级

  Vue 3.0 中使用了 Proxy 对象重写了响应式系统。可以监听动态新增的属性、删除的属性，监听数组的索引和 length 属性，代理效率提升

- 编译过程优化

  Vue 3.0 中标记和提升所有的静态根节点，diff 的时候只需要对比动态节点内容

- 源码体积优化

  移除了一些不常用的 API。兼容了 Tree-Shaking，按需加载模块，打包体积变小

## 2、Vue 3.0 所采用的 Composition Api 与 Vue 2.x使用的Options Api 有什么区别？

- Options Api 在组织代码时逻辑比较分散，而 Composition API 则解决了这个问题，使得代码易于维护和阅读
- Composition API 有利于提取和封装公共逻辑

## 3、Proxy 相对于 Object.defineProperty 有哪些优点？

- Proxy 可以监听对象新增的属性
- Proxy 可以监听对象删除属性
- Proxy 可以监听数组所以和 length
-  Object.defineProperty 只能遍历所有对象对其进行劫持，导致了初始化的时候会浪费时间。但是Proxy代理的时候只需要在get里面判断一下当前数据是否为对象类型，如果是的话，直接递归调用当前劫持方法就好了
- Proxy 代理的是整个对象，而 Object.defineProperty 代理的时对象的属性
- Proxy 支持 13 种拦截操作，是 Object.defineProperty 不具备的。

## 4、Vue 3.0 在编译方面有哪些优化？

- 支持 Fragments 
- 静态节点提升
- Patch flg
- 缓存事件处理函数

## 5、Vue.js 3.0 响应式系统的实现原理？

响应式原理就是利用 `Proxy` 第二个参数 `handler`，拦截各种取值、赋值操作，依托 `track` 和 `trigger` 两个函数进行依赖收集和派发更新。`track` 用来在读取时收集依赖，`trigger` 用来在更新时触发依赖。