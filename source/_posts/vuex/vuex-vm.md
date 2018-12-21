---
title: vuex 之初始化 store._vm
categories:
- vuex
---
模块初始化看完后，我们继续来看初始化 store._vm
<!--more-->
### 一、resetStoreVM
我们回到 Store 的构造器函数，看到在执行 installModule 后，我们紧接着执行了：
```
// initialize the store vm, which is responsible for the reactivity
// (also registers _wrappedGetters as computed properties)
resetStoreVM(this, state)
```
resetStoreVM 定义如下：
```
// 建立 getters 和 state 的联系
function resetStoreVM (store, state, hot) {
  const oldVm = store._vm

  // bind store public getters
  store.getters = {}
  const wrappedGetters = store._wrappedGetters
  const computed = {}
  forEachValue(wrappedGetters, (fn, key) => {
    // use computed to leverage its lazy-caching mechanism（使用 computed 的懒加载机制）
    // 根据 key 访问 store.getters 的某一个 getter 的时候，实际上就是访问了 store._vm[key]，也就是 computed[key]
    // 在执行 computed[key] 对应的函数的时候，会执行 rawGetter(local.state,...) 方法，那么就会访问到 store.state
    // 进而访问到 store._vm_data.$$state，这样就建立了一个依赖关系。当 store.state 发生变化的时候，下一次再访问 store.getters 的时候会重新计算。
    computed[key] = () => fn(store)
    Object.defineProperty(store.getters, key, {
      get: () => store._vm[key],
      enumerable: true // for local getters
    })
  })

  // use a Vue instance to store the state tree
  // suppress warnings just in case the user has added
  // some funky global mixins
  const silent = Vue.config.silent
  Vue.config.silent = true
  store._vm = new Vue({
    data: {
      $$state: state
    },
    computed
  })
  Vue.config.silent = silent

  // enable strict mode for new vm
  if (store.strict) {
    enableStrictMode(store)
  }

  if (oldVm) {
    if (hot) {
      // dispatch changes in all subscribed watchers
      // to force getter re-evaluation for hot reloading.
      store._withCommit(() => {
        oldVm._data.$$state = null
      })
    }
    // 销毁旧的 vue 实例
    Vue.nextTick(() => oldVm.$destroy())
  }
}
```
首先我们获取 root store 上的 _vm 和 _wrappedGetters 属性，上节我们讲到 _wrappedGetters 存储用户定义的所有 getters。接着我们遍历 _wrappedGetters，为 computed 的 key 属性赋值，并为 store.getters 的 key 属性设置拦截器。

继续往下看，我们修改 Vue.config.silent，然后为 store._vm 赋值，并将 Vue.config.silent 改回原先值。这里 Vue.config.silent 的修改主要是为了防止用户配置了全局的 mixins 而打印出警告信息。

我们注意到如果访问 `store.getters[key]`，实际上访问的是 `store._vm[key]`，也就是 computed[key]，在执行 computed[key] 对应的函数的时候，会执行 rawGetter(local.state,...) 方法，那么就会访问到 store.state，进而访问到 store._vm._data.$$state，这样就建立了一个依赖关系。当 store.state 发生变化的时候，下一次再访问 store.getters 的时候会重新计算。这样做的主要目的是为了使用计算属性的懒加载机制。

注意，我们访问 store.state 时其实访问的是：
```
  // 访问 store.state 的时候，实际上会访问 Store 类上定义的 state 的 get 方法
  get state () {
    return this._vm._data.$$state
  }

  // 给出提示信息，不能设置 state 值
  set state (v) {
    if (process.env.NODE_ENV !== 'production') {
      assert(false, `use store.replaceState() to explicit replace store state.`)
    }
  }
```
回到 resetStoreVM 方法，如果存在 oldVm，那么需要调用 $destroy 销毁这个实例。如果是热更新的话，同时设置 oldVm._data.$$state 为 null，来促使所有的监听函数重新计算。
### 二、enableStrictMode()
我们注意到 resetStoreVM 有这样一段代码：
```
  // enable strict mode for new vm
  if (store.strict) {
    enableStrictMode(store)
  }
```
如果传入的 options.strict 为 true，那么执行 enableStrictMode 方法。enableStrictMode 的实现如下：
```
// 开启严格模式（在严格模式下，无论何时发生了状态变更且不是由 mutation 函数引起的，将会抛出错误，这能保证所有的状态变更都能被调试工具跟踪到）
function enableStrictMode (store) {
  // store._vm 添加一个 wathcer 来观测 this._data.$$state 的变化
  store._vm.$watch(function () { return this._data.$$state }, () => {
    // 非生产环境下，当 store.state 被修改的时候, store._committing 必须为 true，否则给出提示信息
    if (process.env.NODE_ENV !== 'production') {
      assert(store._committing, `do not mutate vuex store state outside mutation handlers.`)
    }
  }, { deep: true, sync: true })
}
```
可以看到我们在 store._vm 添加了一个监听器，在 this._data.$$state 发生改变时触发回调函数。回调函数中判断 state 改变时，_committing 是否为 true，如果不是，则给出提示信息。
### 三、plugins
我们回到构造器函数中继续往下看：
```
// apply plugins
plugins.forEach(plugin => plugin(this))

if (Vue.config.devtools) {
    devtoolPlugin(this)
}
```
我们首先遍历配置中的 plugins，并传入 store 实例执行 plugin 方法，所以我们传入的 plugins 选项能够在 new Vuex.Store() 执行时得到 store 实例并执行。然后判断 Vue.config.devtools（是否允许 vue-devtools 检查代码）是否为 true，如果是执行 devtoolPlugin 方法，devtoolPlugin 代码见 'src/plugins/devtool.js' 中：
```
const devtoolHook =
  typeof window !== 'undefined' &&
  window.__VUE_DEVTOOLS_GLOBAL_HOOK__

export default function devtoolPlugin (store) {
  if (!devtoolHook) return

  store._devtoolHook = devtoolHook

  devtoolHook.emit('vuex:init', store)

  devtoolHook.on('vuex:travel-to-state', targetState => {
    store.replaceState(targetState)
  })

  store.subscribe((mutation, state) => {
    devtoolHook.emit('vuex:mutation', mutation, state)
  })
}
```
可以看到我们在 store 上添加了 _devtoolHook 属性，这就是上节我们说到的 store._devtoolHook 来源。