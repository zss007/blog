---
title: vuex 之模块注册
categories:
- vuex
---
看完初始化和工具方法后，我们来看下 vuex 的模块注册。
<!--more-->
### 一、new Vuex.Store()
```
const store = new Vuex.Store({
  state,
  getters,
  actions,
  mutations
})
```
相信大家对这段代码都非常熟悉，在之前的分析中，我们可得知 Store 来自 'src/store.js' 中，也就是：
```
export class Store {
  constructor (options = {}) {
    // Auto install if it is not done yet and `window` has `Vue`.
    // To allow users to avoid auto-installation in some cases,
    // this code should be placed here. See #731
    // 某些场合自动执行 install，比如 <script> 引入
    if (!Vue && typeof window !== 'undefined' && window.Vue) {
      install(window.Vue)
    }

    // 非生产环境下给出调试信息
    if (process.env.NODE_ENV !== 'production') {
      assert(Vue, `must call Vue.use(Vuex) before creating a store instance.`)
      assert(typeof Promise !== 'undefined', `vuex requires a Promise polyfill in this browser.`)
      assert(this instanceof Store, `store must be called with the new operator.`)
    }

    const {
      plugins = [],
      strict = false
    } = options

    // 标志一个提交状态，作用是保证对 Vuex 中 state 的修改只能在 mutation 的回调函数中，而不能在外部随意修改 state
    this._committing = false
    // 用来存储用户定义的所有的 actions
    this._actions = Object.create(null)
    // 存储 dispatch 的订阅回调函数
    this._actionSubscribers = []
    // 用来存储用户定义所有的 mutatins
    this._mutations = Object.create(null)
    // 用来存储用户定义的所有 getters
    this._wrappedGetters = Object.create(null)
    // 注册模块
    this._modules = new ModuleCollection(options)
    // 存储命名空间的模块
    this._modulesNamespaceMap = Object.create(null)
    // 用来存储所有对 mutation 变化的订阅者
    this._subscribers = []
    // 是一个 Vue 对象的实例，主要是利用 Vue 实例方法 $watch 来观测变化
    this._watcherVM = new Vue()

    // bind commit and dispatch to self
    const store = this
    const { dispatch, commit } = this
    // 提交 action，并且绑定 store
    this.dispatch = function boundDispatch (type, payload) {
      return dispatch.call(store, type, payload)
    }
    // 提交 mutation，并且绑定 store
    this.commit = function boundCommit (type, payload, options) {
      return commit.call(store, type, payload, options)
    }

    // this.strict 表示是否开启严格模式，在严格模式下会观测所有的 state 的变化，建议在开发环境时开启严格模式，线上环境要关闭严格模式，否则会有一定的性能开销
    this.strict = strict
    ...
  }
  ...
}
```
我们在 new Vuex.Store() 执行时运行的是 class Store 的构造器，主要是一些变量的初始化，作用见注释。如果还是不太清楚，也没关系，后面我会详细介绍到。
### 二、new ModuleCollection()
现在让我们看下 `this._modules = new ModuleCollection(options)`，然后由开头的 `import ModuleCollection from './module/module-collection'` 得知，代码在 'src/module/module-collection.js' 中：
```
export default class ModuleCollection {
  constructor (rawRootModule) {
    // 注册根模块 (Vuex.Store options)
    this.register([], rawRootModule, false)
  }
  ...
}
```
我们在初始化 this._modules 变量时执行了 class ModuleCollection 的构造器，里面执行的是 register：
```
  // 注册模块
  register (path, rawModule, runtime = true) {
    // 非生产环境下对选项参数做检测
    if (process.env.NODE_ENV !== 'production') {
      assertRawModule(path, rawModule)
    }

    const newModule = new Module(rawModule, runtime)
    ...
  }
```
assertRawModule 实现如下：
```
// 函数类型判断
const functionAssert = {
  assert: value => typeof value === 'function',
  expected: 'function'
}

// 函数或含 handler 函数属性的对象类型判断
const objectAssert = {
  assert: value => typeof value === 'function' ||
    (typeof value === 'object' && typeof value.handler === 'function'),
  expected: 'function or object with "handler" function'
}

// 对 getters、mutations、actions 分别做类型判断
const assertTypes = {
  getters: functionAssert,
  mutations: functionAssert,
  actions: objectAssert
}

// Vuex.Store options 类型检测
function assertRawModule (path, rawModule) {
  Object.keys(assertTypes).forEach(key => {
    // 如果 options 中没有该选项则跳过
    if (!rawModule[key]) return

    // 获取类型检测对象 functionAssert | objectAssert
    const assertOptions = assertTypes[key]

    // 遍历 options 中的选项进行类型检测
    forEachValue(rawModule[key], (value, type) => {
      assert(
        assertOptions.assert(value),
        makeAssertionMessage(path, key, type, value, assertOptions.expected)
      )
    })
  })
}

// 获取断言信息，用于失败时输出提示信息给开发者
function makeAssertionMessage (path, key, type, value, expected) {
  let buf = `${key} should be ${expected} but "${key}.${type}"`
  if (path.length > 0) {
    buf += ` in module "${path.join('.')}"`
  }
  buf += ` is ${JSON.stringify(value)}.`
  return buf
}
```

我们注意到 `const newModule = new Module(rawModule, runtime)`，所以我们来到 'src/module/module.js' 中：
```
// Base data struct for store's module, package with some attribute and method（store 模块的基础数据结构，含一些属性和方法）
export default class Module {
  constructor (rawModule, runtime) {
    // 表示是否是一个运行时创建的模块
    this.runtime = runtime
    // 存储所有子模块
    this._children = Object.create(null)
    // 模块的配置
    this._rawModule = rawModule
    const rawState = rawModule.state

    // 模块定义的 state
    this.state = (typeof rawState === 'function' ? rawState() : rawState) || {}
  }
}
```
现在让我们回到 register 方法中：
```
  // 注册模块
  register (path, rawModule, runtime = true) {
    ...
    if (path.length === 0) {  // 根模块
      this.root = newModule
    } else {  // 子模块
      // 调用父模块的 addChild 方法建立父子关系
      const parent = this.get(path.slice(0, -1))
      parent.addChild(path[path.length - 1], newModule)
    }

    // register nested modules（递归注册子模块，建立父子关系）
    if (rawModule.modules) {
      forEachValue(rawModule.modules, (rawChildModule, key) => {
        this.register(path.concat(key), rawChildModule, runtime)
      })
    }
  }
```
可以发现，这段代码根据配置中的 modules 递归执行 register 注册子模块，至于 addChild 方法见 'src/module/module.js' 中：
```
  // 添加子模块
  addChild (key, module) {
    this._children[key] = module
  }
```
此时我们发现 new ModuleCollection() 执行后，我们就建立了一个模块树。