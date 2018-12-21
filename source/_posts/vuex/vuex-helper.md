---
title: vuex 之辅助方法
categories:
- vuex
---
看完 vuex 的 api 后，我们来到最后一节 vuex 的辅助方法。
<!--more-->
### 一、mapState
```
/**
 * Reduce the code which written in Vue.js for getting the state.（减少获取 state 代码的重复和冗余）
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} states # Object's item can be a function which accept state and getters for param, you can do something for state and getters in it.
 * @param {Object}
 */
export const mapState = normalizeNamespace((namespace, states) => {
  const res = {}
  normalizeMap(states).forEach(({ key, val }) => {
    res[key] = function mappedState () {
      let state = this.$store.state
      let getters = this.$store.getters
      // 传入 namespace
      if (namespace) {
        // 获取相应命名空间的模块
        const module = getModuleByNamespace(this.$store, 'mapState', namespace)
        if (!module) {
          return
        }
        // 赋值子模块的 state、getters
        state = module.context.state
        getters = module.context.getters
      }
      return typeof val === 'function'
        ? val.call(this, state, getters)
        : state[val]
    }
    // mark vuex getter for devtools
    res[key].vuex = true
  })
  return res
})
```
可以看到我们调用 normalizeNamespace 方法，其实现如下：
```
/**
 * Return a function expect two param contains namespace and map. it will normalize the namespace and then the param's function will handle the new namespace and the map.
 * @param {Function} fn
 * @return {Function}
 * 命名空间格式化（namespace 表示命名空间，map 表示具体的对象，namespace 可不传）
 */
function normalizeNamespace (fn) {
  return (namespace, map) => {
    if (typeof namespace !== 'string') {
      map = namespace
      namespace = ''
    } else if (namespace.charAt(namespace.length - 1) !== '/') {
      namespace += '/'
    }
    return fn(namespace, map)
  }
}
```
该方法主要是格式化传入参数。继续回到 mapState，可以看到调用了 normalizeMap，其实现如下：
```
/**
 * Normalize the map（map 格式化）
 * normalizeMap([1, 2, 3]) => [ { key: 1, val: 1 }, { key: 2, val: 2 }, { key: 3, val: 3 } ]
 * normalizeMap({a: 1, b: 2, c: 3}) => [ { key: 'a', val: 1 }, { key: 'b', val: 2 }, { key: 'c', val: 3 } ]
 * @param {Array|Object} map
 * @return {Object}
 */
function normalizeMap (map) {
  return Array.isArray(map)
    ? map.map(key => ({ key, val: key }))
    : Object.keys(map).map(key => ({ key, val: map[key] }))
}
```
可以看出我们将传入的第二个参数进行格式化。回到 mapState，调用 forEach 进行遍历。如果没有传入 namespace，则判断 val 是否为 function，如果是，则调用，否则直接返回 state[val]。如果传入了 namespace，则调用 getModuleByNamespace 获取子模块，然后获取子模块的 state、getters。也就是 mapState 返回的是一个对象，然后将这个对象传给 computed：
```
computed: mapState([
  'count'
])
```
那么 this.count 访问的就是 store.state.count 了。继续来看 getModuleByNamespace 实现：
```
/**
 * Search a special module from store by namespace. if module not exist, print error message.
 * @param {Object} store
 * @param {String} helper 工具方法名，如 mapState
 * @param {String} namespace
 * @return {Object}
 */
function getModuleByNamespace (store, helper, namespace) {
  // 查找相应命名空间的 module
  const module = store._modulesNamespaceMap[namespace]
  // 非生产环境下，如果不存在相应命名空间模块，则给出提示信息
  if (process.env.NODE_ENV !== 'production' && !module) {
    console.error(`[vuex] module namespace not found in ${helper}(): ${namespace}`)
  }
  return module
}
```
主要是通过 _modulesNamespaceMap 来获取子模块。
### 二、mapMutations
```
/**
 * Reduce the code which written in Vue.js for committing the mutation（减少获取 mutations 代码的重复和冗余）
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} mutations # Object's item can be a function which accept `commit` function as the first param, it can accept anthor params. You can commit mutation and do any other things in this function. specially, You need to pass anthor params from the mapped function.
 * @return {Object}
 * 将组件中的 methods 映射为 store.commit 调用
 */
export const mapMutations = normalizeNamespace((namespace, mutations) => {
  const res = {}
  normalizeMap(mutations).forEach(({ key, val }) => {
    // 支持传入额外的参数 args，作为提交 mutation 的 payload，如：`this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    res[key] = function mappedMutation (...args) {
      // Get the commit method from store
      let commit = this.$store.commit
      if (namespace) {
        const module = getModuleByNamespace(this.$store, 'mapMutations', namespace)
        if (!module) {
          return
        }
        // 获取子模块的 commit
        commit = module.context.commit
      }
      return typeof val === 'function'
        ? val.apply(this, [commit].concat(args))
        : commit.apply(this.$store, [val].concat(args))
    }
  })
  return res
})
```
遍历传入的 mutations，并支持传入额外的参数 args。然后获取 commit，如果传入了 namespace，则获取子模块的 commit。
### 三、mapGetters
```
/**
 * Reduce the code which written in Vue.js for getting the getters（减少获取 getters 代码的重复和冗余）
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} getters
 * @return {Object}
 */
export const mapGetters = normalizeNamespace((namespace, getters) => {
  const res = {}
  normalizeMap(getters).forEach(({ key, val }) => {
    // thie namespace has been mutate by normalizeNamespace（namespace 已经被 normalizeNamespace 方法处理好了）
    val = namespace + val
    res[key] = function mappedGetter () {
      // 如果存在命名空间，但是没能在该命名空间下找到相应 module，则返回
      if (namespace && !getModuleByNamespace(this.$store, 'mapGetters', namespace)) {
        return
      }
      // 非生产环境下，如果没有相应 getter，给出提示信息
      if (process.env.NODE_ENV !== 'production' && !(val in this.$store.getters)) {
        console.error(`[vuex] unknown getter: ${val}`)
        return
      }
      return this.$store.getters[val]
    }
    // mark vuex getter for devtools
    res[key].vuex = true
  })
  return res
})
```
遍历 getters，拼接上 namespace，然后从 this.$store.getters 取值返回。
### 四、mapActions
```
/**
 * Reduce the code which written in Vue.js for dispatch the action（减少获取 actions 代码的重复和冗余）
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} actions # Object's item can be a function which accept `dispatch` function as the first param, it can accept anthor params. You can dispatch action and do any other things in this function. specially, You need to pass anthor params from the mapped function.
 * @return {Object}
 */
export const mapActions = normalizeNamespace((namespace, actions) => {
  const res = {}
  normalizeMap(actions).forEach(({ key, val }) => {
    // 支持传入额外的参数 args，作为提交 action 的 payload，如：`this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    res[key] = function mappedAction (...args) {
      // get dispatch function from store
      let dispatch = this.$store.dispatch
      // 带命名空间参数
      if (namespace) {
        // 获取相应命名空间下的 moudle
        const module = getModuleByNamespace(this.$store, 'mapActions', namespace)
        if (!module) {
          return
        }
        // 获取子模块的 dispatch 函数
        dispatch = module.context.dispatch
      }
      return typeof val === 'function'
        ? val.apply(this, [dispatch].concat(args))
        : dispatch.apply(this.$store, [val].concat(args))
    }
  })
  return res
})
```
遍历 actions，如果传入命名空间则获取子模块下的 dispatch 函数。
### 五、createNamespacedHelpers
```
/**
 * Rebinding namespace param for mapXXX function in special scoped, and return them by simple object
 * 创建基于某个命名空间辅助函数，返回一个对象，对象里有新的绑定在给定命名空间值上的组件绑定辅助函数
 * @param {String} namespace
 * @return {Object}
 */
export const createNamespacedHelpers = (namespace) => ({
  mapState: mapState.bind(null, namespace),
  mapGetters: mapGetters.bind(null, namespace),
  mapMutations: mapMutations.bind(null, namespace),
  mapActions: mapActions.bind(null, namespace)
})
```
返回一个对象，其 mapState、mapGetters、mapMutations、mapActions 属性都已经绑定在了给定的命名空间上。