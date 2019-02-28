---
title: koa 之 koa-compose
categories:
- node
---
koa-compose 模块可以将多个中间件合成为一个。
<!--more-->
### 一、源码
```
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */
  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, function next () {
          return dispatch(i + 1)
        }))
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```
### 二、分析
- 1、首先进行类型判断，middleware 参数必须是函数数组。
- 2、index 为上次调用 middleware 数组的索引值，dispatch 采用递归方法调用。
- 3、如果 i <= index，那么 next() 被调用多次，返回 Promise.reject。
- 4、如果 i === middleware.length，说明是最后一个中间件，则将 next 参数赋值给 fn，如果 next 不存在，返回 Promise.resolve，否则继续调用 next，将结果返回。
- 5、Promise.resolve(Promise.resolve(Promise.resolve('resolved'))).then(success, error) 执行 success，Promise.resolve(Promise.resolve(Promise.reject('rejected'))).then(success, error) 执行 error。
- 6、这就是洋葱模型的来由了：
<img src="/assets/node/koa-diagram.png">

### 三、应用
```
// test.js（将多个中间件合成为一个）
const compose = require('koa-compose');

const logger = (ctx, next) => {
  console.log(`${Date.now()} ${ctx.request.method} ${ctx.request.url}`);
  next();
}

const main = ctx => {
  ctx.response.body = 'Hello World';
};

const middlewares = compose([logger, main]);
app.use(middlewares);

$ node test.js
```
访问 http://127.0.0.1:3000 ，就可以在命令行窗口看到日志信息。