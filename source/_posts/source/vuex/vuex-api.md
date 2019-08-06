---
title: vuex 之 api
categories:
- source
---
看完 vuex 的构造器，相信大家对 vuex 的原理都有了个基础的认识，现在我们来看下 vuex 对外暴露的 api
<!--more-->
### 一、commit
```
  // 提交 mutation，子模块的 commit 已在 makeLocalContext 中拼装好前缀
  commit (_type, _payload, _options) {
    // check object-style commit
    const {
      type,
      payload,
      options
    } = unifyObjectStyle(_type, _payload, _options)

    const mutation = { type, payload }
    const entry = this._mutations[type]
    // 非生产环境下给出提示信息，不存在相应的 mutation
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown mutation type: ${type}`)
      }
      return
    }
    this._withCommit(() => {
      entry.forEach(function commitIterator (handler) {
        handler(payload)
      })
    })
    // 触发 commit 的订阅回调函数
    this._subscribers.forEach(sub => sub(mutation, this.state))

    // 非生产环境下给出提示信息，silent 选项已被移除
    if (
      process.env.NODE_ENV !== 'production' &&
      options && options.silent
    ) {
      console.warn(
        `[vuex] mutation type: ${type}. Silent option has been removed. ` +
        'Use the filter functionality in the vue-devtools'
      )
    }
  }
```
首先使用 unifyObjectStyle 格式化对象风格，然后通过 type 获取相应 mutations，注意拿到的是数组，而且 commit 方法执行需要嵌套在 _withCommit 中，原因前面章节有介绍。然后我们遍历执行 mutations，并传入 payload。最后我们遍历 _subscribers，触发 commit 的订阅回调函数。
### 二、dispatch
```
  // 提交 action，子模块的 dispatch 已在 makeLocalContext 中拼装好前缀
  dispatch (_type, _payload) {
    // check object-style dispatch
    const {
      type,
      payload
    } = unifyObjectStyle(_type, _payload)

    const action = { type, payload }
    const entry = this._actions[type]
    // // 非生产环境下给出提示信息，不存在相应的 action
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown action type: ${type}`)
      }
      return
    }

    // 触发 dispatch 的订阅回调函数
    this._actionSubscribers.forEach(sub => sub(action, this.state))

    return entry.length > 1
      ? Promise.all(entry.map(handler => handler(payload)))
      : entry[0](payload)
  }
```
首先使用 unifyObjectStyle 格式化对象风格，然后通过 type 获取相应 actions，注意拿到的是数组，如果数组长度大于 1，则调用 Promise.all。最后我们遍历 _actionSubscribers，触发 dispatch 的订阅回调函数。
### 三、subscribe
```
  // 订阅 store 的 mutation
  subscribe (fn) {
    return genericSubscribe(fn, this._subscribers)
  }
```
直接调用 genericSubscribe 方法，其实现如下：
```
// 通用订阅函数
function genericSubscribe (fn, subs) {
  // 如果是新的回调函数，则添加
  if (subs.indexOf(fn) < 0) {
    subs.push(fn)
  }
  // 返回函数，调用即可停止订阅
  return () => {
    const i = subs.indexOf(fn)
    if (i > -1) {
      subs.splice(i, 1)
    }
  }
}
```
也就是说调用 subscribe 方法时，我们将传入的 fn 添加到 _subscribers 中，然后在每个 mutation 完成后调用。要停止订阅，调用此方法返回的函数即可停止订阅。
### 四、subscribeAction
```
  // 订阅 store 的 action
  subscribeAction (fn) {
    return genericSubscribe(fn, this._actionSubscribers)
  }
```
调用 subscribeAction 方法时，我们将传入的 fn 添加到 _actionSubscribers 中，然后在每个 action 分发的时候调用。要停止订阅，调用此方法返回的函数即可停止订阅。
### 五、watch
```
  // 响应式地侦听 getter 的返回值，当值改变时调用回调函数
  watch (getter, cb, options) {
    if (process.env.NODE_ENV !== 'production') {
      assert(typeof getter === 'function', `store.watch only accepts a function.`)
    }
    // fn 接收 store 的 state 作为第一个参数，其 getter 作为第二个参数。要停止侦听，调用此方法返回的函数即可停止侦听
    return this._watcherVM.$watch(() => getter(this.state, this.getters), cb, options)
  }
```
调用 watch 时，我们将监听 getter 的返回值，如果值发生改变则调用回调函数 cb
### 六、replaceState
```
  replaceState (state) {
    this._withCommit(() => {
      this._vm._data.$$state = state
    })
  }
```
替换 store 的根状态，仅用状态合并或时光旅行调试。
### 七、registerModule
```
  // 模块动态注册
  registerModule (path, rawModule, options = {}) {
    if (typeof path === 'string') path = [path]

    // 非生产环境下给出 path 参数校验提示信息
    if (process.env.NODE_ENV !== 'production') {
      assert(Array.isArray(path), `module path must be a string or an Array.`)
      assert(path.length > 0, 'cannot register the root module by using registerModule.')
    }

    // 注册模块
    this._modules.register(path, rawModule)
    // 初始化模块
    installModule(this, this.state, path, this._modules.get(path), options.preserveState)
    // reset store to update getters...（重新实例化 store._vm，并销毁旧的 store_vm）
    resetStoreVM(this, this.state)
  }
```
首先做参数格式化和校验，然后调用 register 方法注册模块，然后调用 installModule 初始化模块，其中 _modules.get 实现见 'src/module/module-collection.js' 中：
```
  // 获取路径上的子模块
  get (path) {
    return path.reduce((module, key) => {
      return module.getChild(key)
    }, this.root)
  }
```
options 可以包含 preserveState: true 以允许保留之前的 state，也就是说如果设置 options.preserveState，那么 state 保持不变。然后调用 resetStoreVM 方法重新实例化 store._vm，并销毁旧的 store_vm，这样做主要是为了更新 store 的 getters。
### 八、unregisterModule
```
  // 模块动态卸载（只会移除我们运行时动态创建的模块）
  unregisterModule (path) {
    if (typeof path === 'string') path = [path]

    if (process.env.NODE_ENV !== 'production') {
      assert(Array.isArray(path), `module path must be a string or an Array.`)
    }

    // 执行 unregister 方法去修剪我们的模块树
    this._modules.unregister(path)
    // 删除 state 在该路径下的引用
    this._withCommit(() => {
      const parentState = getNestedState(this.state, path.slice(0, -1))
      Vue.delete(parentState, path[path.length - 1])
    })
    // 把 store 下的对应存储的 _actions、_mutations、_wrappedGetters 和 _modulesNamespaceMap 都清空，然后重新执行 installModule 安装所有模块以及 resetStoreVM 重置 store._vm
    resetStore(this)
  }
```
首先进行参数格式化和校验，然后调用 _modules.unregister 方法，具体实现见 'src/module/module-collection.js' 中：
```
  // 卸载模块
  unregister (path) {
    const parent = this.get(path.slice(0, -1))
    const key = path[path.length - 1]
    // 只会移除我们运行时动态创建的模块
    if (!parent.getChild(key).runtime) return

    parent.removeChild(key)
  }
```
注意我们只会移除运行时动态创建的模块。首先获取父模块和当前模块名称，然后调用 removeChild 移除子模块，removeChild 具体实现见 'src/module/module.js' 中：
```
  // 移除子模块
  removeChild (key) {
    delete this._children[key]
  }
```
回到 unregisterModule 中， 我们继续删除 state 在当前模块路径下的引用，并调用 resetStore 方法，其实现如下：
```
// 重置 _actions、_mutations、_wrappedGetters、_modulesNamespaceMap，并重新执行 installModule、resetStoreVM
function resetStore (store, hot) {
  store._actions = Object.create(null)
  store._mutations = Object.create(null)
  store._wrappedGetters = Object.create(null)
  store._modulesNamespaceMap = Object.create(null)
  const state = store.state
  // init all modules
  installModule(store, state, [], store._modules.root, true)
  // reset vm
  resetStoreVM(store, state, hot)
}
```
该方法主要是清空 _actions、_mutations、_wrappedGetters、_modulesNamespaceMap，并重新初始化模块和重置 store._vm
### 九、hotUpdate
```
  // 热替换新的 action 和 mutation
  hotUpdate (newOptions) {
    this._modules.update(newOptions)
    // 把 store 下的对应存储的 _actions、_mutations、_wrappedGetters 和 _modulesNamespaceMap 都清空，然后重新执行 installModule 安装所有模块以及 resetStoreVM 重置 store._vm
    resetStore(this, true)
  }
```
首先看到调用了 _modules.update 方法，其实现见 'src/module/module-collection.js' 中：
```
  // 更新模块，执行 update (path, targetModule, newModule)
  update (rawRootModule) {
    // 更新根模块
    update([], this.root, rawRootModule)
  }
```
调用了该文件自身的 update 方法：
```
// 更新模块具体实现
function update (path, targetModule, newModule) {
  // 非生产环境下做类型检测
  if (process.env.NODE_ENV !== 'production') {
    assertRawModule(path, newModule)
  }

  // update target module（更新 targetModule 的 namespaced、actions、mutations、getters）
  targetModule.update(newModule)

  // update nested modules
  if (newModule.modules) {
    for (const key in newModule.modules) {
      // 如果 newModule 存在 targetModule 没有的子模块，则给出 reload 提示信息
      if (!targetModule.getChild(key)) {
        if (process.env.NODE_ENV !== 'production') {
          console.warn(
            `[vuex] trying to add a new module '${key}' on hot reloading, ` +
            'manual reload is needed'
          )
        }
        return
      }
      // 递归更新子模块
      update(
        path.concat(key),
        targetModule.getChild(key),
        newModule.modules[key]
      )
    }
  }
}
```
然后 targetModule.update 方法见 'src/module/module.js' 中：
```
  update (rawModule) {
    this._rawModule.namespaced = rawModule.namespaced
    if (rawModule.actions) {
      this._rawModule.actions = rawModule.actions
    }
    if (rawModule.mutations) {
      this._rawModule.mutations = rawModule.mutations
    }
    if (rawModule.getters) {
      this._rawModule.getters = rawModule.getters
    }
  }
```
主要是更新当前 module 的 namespaced、actions、mutations、getters