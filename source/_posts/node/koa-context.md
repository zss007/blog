---
title: koa 之 context
categories:
- node
---
Koa Context 将 node 的 request 和 response 对象封装到单个对象中，为编写 Web 应用程序和 API 提供了许多有用的方法。每个请求都将创建一个 Context，并在中间件中作为接收器引用，或者 ctx 标识符。
<!--more-->
### 一、源码
```js
const util = require('util');
const createError = require('http-errors'); // 对错误的处理
const httpAssert = require('http-assert');  // 断言
const delegate = require('delegates');  // 委托
const statuses = require('statuses'); // 状态码

// Context 的原型对象
const proto = module.exports = {
  // util.inspect 实现
  inspect() {
    if (this === proto) return this;  // 如果是原型对象调用，则返回
    return this.toJSON(); // 如果是实例调用，则调用 toJSON 方法
  },

  // 返回 JSON 对象
  toJSON() {
    return {
      request: this.request.toJSON(),
      response: this.response.toJSON(),
      app: this.app.toJSON(),
      originalUrl: this.originalUrl,
      req: '<original node req>',
      res: '<original node res>',
      socket: '<original node socket>'
    };
  },

  // 添加断言，如果不满足条件，则抛出封装的 Error 对象，比如：this.assert(this.user, 401, 'Please login!');
  assert: httpAssert,

  // 抛出带有状态码和报错信息的 Error 对象：this.throw(403) | this.throw('name required', 400) | this.throw(400, 'name required') | this.throw('something exploded') | this.throw(new Error('invalid'), 400) | this.throw(400, new Error('invalid'));
  throw(...args) {
    throw createError(...args); // 根据参数生成相应的 Error 对象
  },

  // 默认错误处理函数（私有方法）
  onerror(err) {
    // 如果没有 error，则不做任何处理（可作为 node 风格的回调函数）
    if (null == err) return;

    // 如果不是 Error 实例，则抛出相应提示信息的 Error 对象
    if (!(err instanceof Error)) err = new Error(util.format('non-error thrown: %j', err));

    let headerSent = false;
    // this.headerSent：来自 response 模块，如果响应头已被发送则为 true，否则为 false
    // this.writable：来自 response 模块，当传递数据时允许写入 socket，否则忽略。默认 false
    if (this.headerSent || !this.writable) {
      headerSent = err.headerSent = true;
    }

    // 将 context 的错误处理委托到 application 上
    this.app.emit('error', err, this);

    // 如果响应头已被发送，那么只能委托到 app 层面处理并记录日志
    if (headerSent) {
      return;
    }

    const { res } = this;

    // 首先移除所有响应头
    if (typeof res.getHeaderNames === 'function') { // res.getHeaderNames：返回一个包含当前响应唯一名称的 http 头信息名称数组. 名称均为小写
      res.getHeaderNames().forEach(name => res.removeHeader(name)); // 移除一个已经在 headers 对象里面的 header
    } else {
      res._headers = {}; // Node < 7.7
    }

    // 然后将错误对象的 headers 属性设置为 response 相应属性
    this.set(err.headers);

    // 强制响应头 Content-Type 为 text/plain
    this.type = 'text';

    // ENOENT 类型支持，一般是 'No such file or directory'
    if ('ENOENT' == err.code) err.status = 404;

    // 默认设置状态码为 500
    if ('number' != typeof err.status || !statuses[err.status]) err.status = 500;

    // respond
    const code = statuses[err.status];
    const msg = err.expose ? err.message : code;  // 如果 expose 为 true，那么显示 err.message 错误描述信息，否则显示状态码描述信息
    this.status = err.status; // 设置 response 的状态码
    this.length = Buffer.byteLength(msg); // 设置响应头 Content-Length 为 msg 字节长度
    this.res.end(msg);  // 发送数据并告知服务器所有数据都已被发送
  }
};

// 响应委托
delegate(proto, 'response')
  .method('attachment')
  .method('redirect')
  .method('remove')
  .method('vary')
  .method('set')
  .method('append')
  .method('flushHeaders')
  .access('status')
  .access('message')
  .access('body')
  .access('length')
  .access('type')
  .access('lastModified')
  .access('etag')
  .getter('headerSent')
  .getter('writable');

// 请求委托
delegate(proto, 'request')
  .method('acceptsLanguages')
  .method('acceptsEncodings')
  .method('acceptsCharsets')
  .method('accepts')
  .method('get')
  .method('is')
  .access('querystring')
  .access('idempotent')
  .access('socket')
  .access('search')
  .access('method')
  .access('query')
  .access('path')
  .access('url')
  .getter('origin')
  .getter('href')
  .getter('subdomains')
  .getter('protocol')
  .getter('host')
  .getter('hostname')
  .getter('URL')
  .getter('header')
  .getter('headers')
  .getter('secure')
  .getter('stale')
  .getter('fresh')
  .getter('ips')
  .getter('ip');
```
### 二、分析
##### 1、为方便起见许多上下文的访问器和方法使用 delegates 直接委托给它们的 ctx.request 或 ctx.response，不然的话它们是相同的。例如 ctx.type 和 ctx.length 委托给 response 对象，ctx.path 和 ctx.method 委托给 request。
##### 2、绕过 Koa 的 response 处理是不被支持的，除非显式设置 ctx.respond = false，从而使用原生 res api：res.statusCode、res.writeHead()、res.write()、res.end()，代码实现于 application.js。
##### 3、ctx.state 是推荐的命名空间，用于通过中间件传递信息。
##### 4、ctx.throw 抛出根据参数自定义的错误对象。