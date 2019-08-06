---
title: vuex 之初始化
categories:
- source
---
最近终于挤出一段时间把 vuex 源码看完了，现在开始总结一波，先从初始化开始逐行分析。
<!--more-->
### 一、入口
访问 https://github.com/vuejs/vuex ，然后 clone 下工程。拿到项目后，我们先来看下 package.json。
```
{
  "name": "vuex",
  "version": "3.0.1",
  "description": "state management for Vue.js",
  "main": "dist/vuex.common.js",
  "module": "dist/vuex.esm.js",
  "unpkg": "dist/vuex.js",
  ...
}
```
我们重点观察入口文件 "dist/vuex.common.js"、"dist/vuex.esm.js"、"dist/vuex.js"，分别用于 node、es6、browser 环境。注意这些文件都是编译后的文件，而不是源代码。这个时候我们来看 configs.js 部分代码：
```
...
const configs = {
  umdDev: {
    input: resolve('src/index.js'),
    file: resolve('dist/vuex.js'),
    format: 'umd',
    env: 'development'
  },
  umdProd: {
    input: resolve('src/index.js'),
    file: resolve('dist/vuex.min.js'),
    format: 'umd',
    env: 'production'
  },
  commonjs: {
    input: resolve('src/index.js'),
    file: resolve('dist/vuex.common.js'),
    format: 'cjs'
  },
  esm: {
    input: resolve('src/index.esm.js'),
    file: resolve('dist/vuex.esm.js'),
    format: 'es'
  }
}
...
```
可以能清晰的看到配置文件中，不同配置分别对应不同环境，但是入口文件都是 'src/index.js' 或 'src/index.esm.js'，我们就从 'src/index.js' 开始。
### 二、Vue.use(Vuex)
相信用过 vuex 的人对 'Vue.use(Vuex)' 这段代码非常熟悉，我们就不绕圈子了，直接说明下他的作用。先来到 vue 的源码片段：
```
  Vue.use = function (plugin: Function | Object) {
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // additional parameters
    const args = toArray(arguments, 1)
    args.unshift(this)
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    installedPlugins.push(plugin)
    return this
  }
```
可以看到在 vue 源码中先获取 _installedPlugins 变量，然后判断是否已经存在。后面是获取 Vue.use 方法其他参数，然后如果存在 install 方法的话，调用 install 方法，并传入其他参数，如果不存在 install，则直接调用 plugin。

那么我们现在回到 Vuex 源码中，来看下 'src/index.js' 代码：
```
import { Store, install } from './store'
import { mapState, mapMutations, mapGetters, mapActions, createNamespacedHelpers } from './helpers'

export default {
  Store,
  install,
  version: '__VERSION__',
  mapState,
  mapMutations,
  mapGetters,
  mapActions,
  createNamespacedHelpers
}
```
可以发现 install 方法存在于 'src/store.js' 中，我们来看下 install 方法实现：
```
import applyMixin from './mixin'
...
export function install (_Vue) {
  // 保证反复调用只执行一次
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
```
我们发现传入参数 _Vue 被赋值给 Vue 变量，这样我们就能够在插件中使用 Vue 而不用 import 了。你可能好奇 _Vue 什么时候被传入的，可以往上翻一下 `args.unshift(this)` 这里将 this 也就是 Vue 放入 args 中，然后在 `plugin.install.apply(plugin, args)` 中传入。

我们继续往下看，来到 'src/mixin.js' 中：
```
export default function (Vue) {
  const version = Number(Vue.version.split('.')[0])

  if (version >= 2) {
    // 全局混入 beforeCreate 钩子函数
    Vue.mixin({ beforeCreate: vuexInit })
  } else {
    // override init and inject vuex init procedure
    // for 1.x backwards compatibility.
    const _init = Vue.prototype._init
    Vue.prototype._init = function (options = {}) {
      options.init = options.init
        ? [vuexInit].concat(options.init)
        : vuexInit
      _init.call(this, options)
    }
  }
}
```
可以看到，我们在 mixin.js 中主要就是注入 before 钩子函数。如果 vue 版本 >=2 的话，就全局混入 beforeCreate 钩子函数，1.x 的版本我们就不研究了。

然后我们来看 vuexInit 函数，看下在 beforeCreate 的时候到底做啥了：
```
  /**
   * Vuex init hook, injected into each instances init hooks list.
   * 给 Vue 的实例注入一个 $store 的属性
   */

  function vuexInit () {
    const options = this.$options
    // 把 options.store 保存在所有组件的 this.$store 中
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) { // 如果 $options 没有 store，则向其 parent 向上查找并赋值
      this.$store = options.parent.$store
    }
  }
```
可以看到，我们通过 options.store 获取 vuex 的实例对象，并赋值给 this.$store。而且注意，如果是子组件，我们则在父组件上找，将父组件上的 $store 赋值给子组件。这样我们所有的 vue 组件实例就都能够通过 this.$store 拿到 vuex 的实例对象，而且他们是共享的。