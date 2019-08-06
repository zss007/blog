---
title: vuex 之模块初始化
categories:
- source
---
现在我们来看下 vuex 的模块初始化。
<!--more-->
### 一、installModule()
我们接着来看 'src/store.js' 中的构造器：
```
export class Store {
  constructor (options = {}) {
    const state = this._modules.root.state

    // init root module. 初始化根模块
    // this also recursively registers all sub-modules 并递归注册所有子模块
    // and collects all module getters inside this._wrappedGetters
    installModule(this, state, [], this._modules.root)
  }
}
```
经过上节的分析，我们知道 this._modules.root 为根模块，因此从根模块上获取 state 属性。然后执行的是 installModule：
```
// 对模块中的 state、getters、mutations、actions 做初始化工作
// store 表示 root store；state 表示 root state；path 表示模块的访问路径；module 表示当前的模块；hot 表示是否是热更新
function installModule (store, rootState, path, module, hot) {
  // 判断是否是根模块
  const isRoot = !path.length
  // 获取 path 路径下的命名空间
  const namespace = store._modules.getNamespace(path)

  // register in namespace map（把 namespace 对应的模块保存下来，为了方便以后能根据 namespace 查找模块）
  if (module.namespaced) {
    store._modulesNamespaceMap[namespace] = module
  }

  // state 初始化
  if (!isRoot && !hot) {
    const parentState = getNestedState(rootState, path.slice(0, -1))
    const moduleName = path[path.length - 1]
    store._withCommit(() => {
      Vue.set(parentState, moduleName, module.state)
    })
  }

  const local = module.context = makeLocalContext(store, namespace, path)
  ...
}
```
我们一步步来分析下，先看 getNamespace 方法实现，见 'src/module/module-collection.js' 中：
```
  // 获取相应路径下模块的命名空间（从 root module 开始，通过 reduce 方法一层层找子模块，如果发现该模块配置了 namespaced 为 true，则把该模块的 key 拼到 namesapce 中，最终返回完整的 namespace 字符串）
  getNamespace (path) {
    let module = this.root
    return path.reduce((namespace, key) => {
      module = module.getChild(key)
      return namespace + (module.namespaced ? key + '/' : '')
    }, '')
  }
```
this.root 表示根模块，namespaced 实现见 'src/module/module.js' 中：
```
  // 该模块是否带有命名空间
  get namespaced () {
    return !!this._rawModule.namespaced
  }
```
getChild 实现见 'src/module/module.js' 中：
```
  // 获取相应 key 的子模块
  getChild (key) {
    return this._children[key]
  }
```
即如果 vuex 的子模块配置中出现了 namespaced，那么 module.namespaced 返回 true。然后我们使用 _modulesNamespaceMap 存储命名空间的模块，以便后续可根据 namespace 查找模块。
### 二、makeLocalContext()
我们接着来看 installModule 中的 makeLocalContext：
```
/**
 * make localized dispatch, commit, getters and state
 * if there is no namespace, just use root ones
 * store 表示 root store；namespace 表示模块的命名空间，path 表示模块的 path
 */
function makeLocalContext (store, namespace, path) {
  const noNamespace = namespace === ''

  // 如果没有命名空间，则使用 root store 的 dispatch 和 commit 方法，options 为 { root: true } 时，在全局命名空间内分发 action 或提交 mutation
  const local = {
    dispatch: noNamespace ? store.dispatch : (_type, _payload, _options) => {
      const args = unifyObjectStyle(_type, _payload, _options)
      const { payload, options } = args
      let { type } = args

      if (!options || !options.root) {
        // 把 type 自动拼接上 namespace
        type = namespace + type
        // 非生产环境下给出提示信息
        if (process.env.NODE_ENV !== 'production' && !store._actions[type]) {
          console.error(`[vuex] unknown local action type: ${args.type}, global type: ${type}`)
          return
        }
      }

      return store.dispatch(type, payload)
    },

    commit: noNamespace ? store.commit : (_type, _payload, _options) => {
      const args = unifyObjectStyle(_type, _payload, _options)
      const { payload, options } = args
      let { type } = args
      
      if (!options || !options.root) {
        // 把 type 自动拼接上 namespace
        type = namespace + type
        // 非生产环境下给出提示信息
        if (process.env.NODE_ENV !== 'production' && !store._mutations[type]) {
          console.error(`[vuex] unknown local mutation type: ${args.type}, global type: ${type}`)
          return
        }
      }

      store.commit(type, payload, options)
    }
  }

  // getters and state object must be gotten lazily
  // because they will be changed by vm update
  Object.defineProperties(local, {
    getters: {
      get: noNamespace
        ? () => store.getters
        : () => makeLocalGetters(store, namespace)
    },
    state: {
      get: () => getNestedState(store.state, path)
    }
  })

  return local
}
```
可以看到我们主要是为 local 对象的 getters 和 state 属性定义 getter 拦截器。然后让我们继续往下看，然后再回过头来分析这段代码。
### 三、module.forEach***
```
  // 遍历模块下的 mutations，并注册
  module.forEachMutation((mutation, key) => {
    const namespacedType = namespace + key
    registerMutation(store, namespacedType, mutation, local)
  })

  // 遍历模块下的 actions，并注册
  module.forEachAction((action, key) => {
    // root 为 true 时，表示在带命名空间的模块注册全局 action
    const type = action.root ? key : namespace + key
    const handler = action.handler || action
    registerAction(store, type, handler, local)
  })

  // 遍历模块下的 getters，并注册
  module.forEachGetter((getter, key) => {
    const namespacedType = namespace + key
    registerGetter(store, namespacedType, getter, local)
  })

  // 遍历模块中的所有子 modules，递归执行 installModule 方法
  module.forEachChild((child, key) => {
    installModule(store, rootState, path.concat(key), child, hot)
  })
```
可以看到我们调用了 module 的 forEachMutation、forEachAction、forEachGetter、forEachChild 方法，其具体实现代码见 'src/module/module.js' 中：
```
  // 遍历子模块，将键值对传入 fn
  forEachChild (fn) {
    forEachValue(this._children, fn)
  }

  // 遍历当前模块 getters，将键值对传入 fn
  forEachGetter (fn) {
    if (this._rawModule.getters) {
      forEachValue(this._rawModule.getters, fn)
    }
  }

  // 遍历当前模块 actions，将键值对传入 fn
  forEachAction (fn) {
    if (this._rawModule.actions) {
      forEachValue(this._rawModule.actions, fn)
    }
  }

  // 遍历当前模块 mutations，将键值对传入 fn
  forEachMutation (fn) {
    if (this._rawModule.mutations) {
      forEachValue(this._rawModule.mutations, fn)
    }
  }
```
可以看到我们分别遍历当前模块下的 mutations、actions、getters、modules 并注册，然后递归调用 installModule 方法遍历当前模块的子模块。
### 四、registerMutation()
我们继续回到 installModule 方法中：
```
  // 遍历模块下的 mutations，并注册
  module.forEachMutation((mutation, key) => {
    const namespacedType = namespace + key
    registerMutation(store, namespacedType, mutation, local)
  })
```
首先使用当前模块的命名空间 namespace 和模块名 key 拼接，然后调用 registerMutation 方法：
```
// 给 root store 上的 _mutations[types] 添加 wrappedMutationHandler 方法（注意，同一 type 的 _mutations 可以对应多个方法）
function registerMutation (store, type, handler, local) {
  const entry = store._mutations[type] || (store._mutations[type] = [])
  entry.push(function wrappedMutationHandler (payload) {
    handler.call(store, local.state, payload)
  })
}
```
首先获取或初始化 _mutations 相应命名空间下数组，然后我们在 _mutations 中添加 wrappedMutationHandler 方法。注意 handle 即配置中的 mutations 方法，this 绑定为 root store，payload 为载荷，然后我们回到 makeLocalContext 中查看 local.state 拿到的数据：
```
Object.defineProperties(local, {
    ...
    state: {
      get: () => getNestedState(store.state, path)
    }
  })
```
getNestedState 的实现如下：
```
// 从 root state 开始，通过 path.reduce 方法一层层查找子模块 state，最终找到目标模块的 state
function getNestedState (state, path) {
  return path.length
    ? path.reduce((state, key) => state[key], state)
    : state
}
```
看到这里，我们应该理解原来 local.state 获取的就是当前模块的 state。不过还有个问题，就是 state[key] 为什么就是相应 key 子模块的 state，这时候我们就要回到 installModule 方法看这段代码：
```
  // state 初始化
  if (!isRoot && !hot) {
    const parentState = getNestedState(rootState, path.slice(0, -1))
    const moduleName = path[path.length - 1]
    store._withCommit(() => {
      Vue.set(parentState, moduleName, module.state)
    })
  }
```
我们可以看到如果不是根模块而且不是热更新，那么我们先拿到父模块的 state，然后获取当前模块名，最后调用 _withCommit 方法，并在方法参数内调用 Vue.set 为父模块 state 访问子模块建立联系，即 `parentState[moduleName] = module.state`。然后我们来看下 _withCommit 方法的实现：
```
  // 内置提交修改 state，防止被捕获
  _withCommit (fn) {
    const committing = this._committing
    this._committing = true
    fn()
    this._committing = committing
  }
```
我们之所以要这么做是因为要保证对 Vuex 中 state 的修改只能在 mutation 的回调函数中。
### 五、registerAction()
我们继续回到 installModule 方法中：
```
  // 遍历模块下的 actions，并注册
  module.forEachAction((action, key) => {
    // root 为 true 时，表示在带命名空间的模块注册全局 action
    const type = action.root ? key : namespace + key
    const handler = action.handler || action
    registerAction(store, type, handler, local)
  })
```
注意在 vuex 配置中，action 可设置 root 为 true，表示在带命名空间的模块注册全局 action，如下：
```
{
  ...
  modules: {
    foo: {
      namespaced: true,

      actions: {
        someAction: {
          root: true,
          handler (namespacedContext, payload) { ... } // -> 'someAction'
        }
      }
    }
  }
}
```
而 `action.handler || action` 为 action 的两种配置格式，我们接着看 registerAction：
```
// 给 root store 上的 _actions[types] 添加 wrappedActionHandler 方法
function registerAction (store, type, handler, local) {
  const entry = store._actions[type] || (store._actions[type] = [])
  entry.push(function wrappedActionHandler (payload, cb) {
    let res = handler.call(store, {
      dispatch: local.dispatch,
      commit: local.commit,
      getters: local.getters,
      state: local.state,
      rootGetters: store.getters,
      rootState: store.state
    }, payload, cb)
    if (!isPromise(res)) {
      res = Promise.resolve(res)
    }
    if (store._devtoolHook) {
      return res.catch(err => {
        store._devtoolHook.emit('vuex:error', err)
        throw err
      })
    } else {
      return res
    }
  })
}
```
首先获取或初始化 _actions 对应命名空间下的数组，然后添加 wrappedActionHandler 方法。可以看到我们 action 调用时，this 绑定为 root store，context 对象存在 dispatch、commit、getters、state、rootGetters 和 rootState 属性，并且将 action 执行返回结果处理为 Promise 对象。
我们刚才已经讲过 local.state，现在我们来看下 local.dispatch、local.commit 和 local.getters：
```
  // 如果没有命名空间，则使用 root store 的 dispatch 和 commit 方法，options 为 { root: true } 时，在全局命名空间内分发 action 或提交 mutation
  const local = {
    dispatch: noNamespace ? store.dispatch : (_type, _payload, _options) => {
      const args = unifyObjectStyle(_type, _payload, _options)
      const { payload, options } = args
      let { type } = args

      if (!options || !options.root) {
        // 把 type 自动拼接上 namespace
        type = namespace + type
        // 非生产环境下给出提示信息
        if (process.env.NODE_ENV !== 'production' && !store._actions[type]) {
          console.error(`[vuex] unknown local action type: ${args.type}, global type: ${type}`)
          return
        }
      }

      return store.dispatch(type, payload)
    },

    commit: noNamespace ? store.commit : (_type, _payload, _options) => {
      const args = unifyObjectStyle(_type, _payload, _options)
      const { payload, options } = args
      let { type } = args
      
      if (!options || !options.root) {
        // 把 type 自动拼接上 namespace
        type = namespace + type
        // 非生产环境下给出提示信息
        if (process.env.NODE_ENV !== 'production' && !store._mutations[type]) {
          console.error(`[vuex] unknown local mutation type: ${args.type}, global type: ${type}`)
          return
        }
      }

      store.commit(type, payload, options)
    }
  }

  // getters and state object must be gotten lazily
  // because they will be changed by vm update
  Object.defineProperties(local, {
    getters: {
      get: noNamespace
        ? () => store.getters
        : () => makeLocalGetters(store, namespace)
    },
    ...
  })
```
先看 local.getters，我们注意到，如果没有命名空间，那么返回 root store 的 getters，否则执行 makeLocalGetters：
```
// 获取子模块的 getters
function makeLocalGetters (store, namespace) {
  const gettersProxy = {}

  const splitPos = namespace.length
  // 遍历 root store 下的所有 getters
  Object.keys(store.getters).forEach(type => {
    // skip if the target getter is not match this namespace（判断是否匹配命名空间）
    if (type.slice(0, splitPos) !== namespace) return

    // extract local getter type（只有匹配的时候从 namespace 的位置截取后面的字符串得到 localType）
    const localType = type.slice(splitPos)

    // Add a port to the getters proxy.
    // Define as getter property because
    // we do not want to evaluate the getters in this time.
    Object.defineProperty(gettersProxy, localType, {
      get: () => store.getters[type],
      enumerable: true
    })
  })

  return gettersProxy
}
```
再来看下 local.dispatch，如果没有命名空间，则使用 root store 的 dispatch 方法，否则调用封装方法。首先我们来看 unifyObjectStyle 方法实现：
```
// 统一对象风格 store.commit({ type: 'increment', amount: 10 }) | store.commit('increment', { amount: 10 })
function unifyObjectStyle (type, payload, options) {
  if (isObject(type) && type.type) {
    options = payload
    payload = type
    type = type.type
  }

  // 非生产环境下给出提示信息
  if (process.env.NODE_ENV !== 'production') {
    assert(typeof type === 'string', `expects string as the type, but found ${typeof type}.`)
  }

  return { type, payload, options }
}
```
可以看到 unifyObjectStyle 是用来统一对象风格的。然后继续看 local.dispatch 方法，我们判断 options 是否存在而且 root 属性是否为 true。如果 if 语句执行失败，则直接调用 root store 的 dispatch 方法，否则我们为 type 拼接上命名空间，然后调用 dispatch 方法。local.commit 基本同 local.dispatch，我们就不讨论了。最后我们来看这段代码：
```
if (store._devtoolHook) {
    return res.catch(err => {
        store._devtoolHook.emit('vuex:error', err)
        throw err
    })
}
```
如果 store 存在 _devtoolHook，即我们使用 devtool 插件时，我们为 Promise 对象 res 添加 catch 方法捕获错误。
### 六、registerGetter()
我们现在回到 installModule 方法：
```
  // 遍历模块下的 getters，并注册
  module.forEachGetter((getter, key) => {
    const namespacedType = namespace + key
    registerGetter(store, namespacedType, getter, local)
  })
```
我们首先拼接出命名空间路径，然后调用 registerGetter 方法：
```
// root store 上的 _wrappedGetters[key] 指定 wrappedGetter 方法
function registerGetter (store, type, rawGetter, local) {
  // 注意，同一 type 的 _wrappedGetters 只能定义一个，非生产环境下给出提示信息
  if (store._wrappedGetters[type]) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(`[vuex] duplicate getter key: ${type}`)
    }
    return
  }
  store._wrappedGetters[type] = function wrappedGetter (store) {
    return rawGetter(
      local.state, // local state
      local.getters, // local getters
      store.state, // root state
      store.getters // root getters
    )
  }
}
```
我们判断在 _wrappedGetters 是否已经存在相应命名空间路径的 getter，如果存在直接返回并给出提示，即同一 type 的 _wrappedGetters 只能定义一个。如果并未定义，我们将 wrappedGetter 赋值给 store._wrappedGetters[type]。
