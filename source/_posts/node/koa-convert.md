---
title: koa 之 koa-convert
categories:
- node
---
转换 0.x 和 1.x 版本的 koa generator 中间件到 2.x 版本 promise 包裹的中间件，也可以转换现今版本 promise 包括的中间件回之前的 generator 中间件，这里主要分析 convert。
<!--more-->
### 一、源码
```
const co = require('co')
const compose = require('koa-compose')

module.exports = convert

function convert (mw) {
  if (typeof mw !== 'function') {
    throw new TypeError('middleware must be a function')
  }
  if (mw.constructor.name !== 'GeneratorFunction') {
    // assume it's Promise-based middleware
    return mw
  }
  const converted = function (ctx, next) {
    return co.call(ctx, mw.call(ctx, createGenerator(next)))
  }
  converted._name = mw._name || mw.name
  return converted
}

function * createGenerator (next) {
  return yield next()
}

// convert.compose(mw, mw, mw)
// convert.compose([mw, mw, mw])
convert.compose = function (arr) {
  if (!Array.isArray(arr)) {
    arr = Array.from(arguments)
  }
  return compose(arr.map(convert))
}

convert.back = function (mw) {
  if (typeof mw !== 'function') {
    throw new TypeError('middleware must be a function')
  }
  if (mw.constructor.name === 'GeneratorFunction') {
    // assume it's generator middleware
    return mw
  }
  const converted = function * (next) {
    let ctx = this
    let called = false
    // no need try...catch here, it's ok even `mw()` throw exception
    yield Promise.resolve(mw(ctx, function () {
      if (called) {
        // guard against multiple next() calls
        // https://github.com/koajs/compose/blob/4e3e96baf58b817d71bd44a8c0d78bb42623aa95/index.js#L36
        return Promise.reject(new Error('next() called multiple times'))
      }
      called = true
      return co.call(ctx, next)
    }))
  }
  converted._name = mw._name || mw.name
  return converted
}
```
### 二、分析
##### 1、首先做类型判断，只能转换 Generator 函数。
##### 2、convert 执行后返回 converted 函数，其接受 2.x 版本中间件参数 context & next，并赋值 _name 属性，执行完成后返回 promise。
##### 3、converted 中先调用 createGenerator，返回一个迭代器，然后传入 mw 中，再次执行 mw 返回迭代器作为 co 模块的参数。
##### 4、co 模块先调用 mw 返回的迭代器的 next 方法，然后继续执行直到执行到 yield next 方法，next 会被 co 模块再次调用，然后执行到下一中间件。
### 三、应用
```
const Koa = require('koa') // koa v2.x
const convert = require('koa-convert')
const app = new Koa()

app.use(modernMiddleware)

app.use(convert(legacyMiddleware))

app.use(convert.compose(legacyMiddleware, modernMiddleware))

function * legacyMiddleware (next) {
  // before
  yield next
  // after
}

function modernMiddleware (ctx, next) {
  // before
  return next().then(() => {
    // after
  })
}
```