---
title: vuex 之工具函数
categories:
- vuex
---
我们按照先易后难的原则，先把容易看懂的部分研究完，这节我们来看 'src/util.js'。
<!--more-->
### 一、find
获取 list 执行 f 过滤后的第一个元素
```
/**
 * Get the first item that pass the test
 * by second argument function
 * @param {Array} list
 * @param {Function} f
 * @return {*}
 */
export function find (list, f) {
  return list.filter(f)[0]
}
```
### 二、deepCopy
递归调用深拷贝对象，cache 数组会缓存所有拷贝对象和嵌套对象，避免无限循环
```
/**
 * Deep copy the given object considering circular structure.
 * This function caches all nested objects and its copies.
 * If it detects circular structure, use cached copy to avoid infinite loop.
 * @param {*} obj
 * @param {Array<Object>} cache
 * @return {*}
 */
export function deepCopy (obj, cache = []) {
  // 如果 obj 是 null 或者基本数据类型，则直接返回 obj
  if (obj === null || typeof obj !== 'object') {
    return obj
  }

  // if obj is hit, it is in circular(递归) structure
  const hit = find(cache, c => c.original === obj)
  if (hit) {
    return hit.copy
  }

  const copy = Array.isArray(obj) ? [] : {}
  // put the copy into cache at first
  // because we want to refer it in recursive deepCopy
  cache.push({
    original: obj,
    copy
  })

  // 递归调用
  Object.keys(obj).forEach(key => {
    copy[key] = deepCopy(obj[key], cache)
  })

  return copy
}
```
### 三、forEachValue
遍历 obj，将键值对传入 fn 调用
```
/**
 * forEach for object
 */
export function forEachValue (obj, fn) {
  Object.keys(obj).forEach(key => fn(obj[key], key))
}
```
### 四、isObject
判断是否为 Object
```
export function isObject (obj) {
  return obj !== null && typeof obj === 'object'
}
```
### 五、isPromise
判断是否为 isPromise
```
export function isPromise (val) {
  return val && typeof val.then === 'function'
}
```
### 六、assert
断言，如果不满足 condition，则抛出 `[vuex] ${msg}` 错误

```
export function assert (condition, msg) {
  if (!condition) throw new Error(`[vuex] ${msg}`)
}
```