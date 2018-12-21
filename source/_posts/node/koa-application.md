---
title: koa 之 application
categories:
- node
---
打开 koa 的源码，核心文件共四个在 lib 目录下：application.js、context.js、request.js、response.js，我们先来看看 application.js。
<!--more-->
### 一、模块依赖
首先看看依赖的模块和文件。
```js
const isGeneratorFunction = require('is-generator-function'); // 判断当前传入函数是否为 generator 函数
const debug = require('debug')('koa:application');  // 轻量级 js debug 调试工具
const onFinished = require('on-finished');  // 事件监听，监听 http 完成或者关闭
const response = require('./response'); // response 模块
const compose = require('koa-compose'); // 使用 koa-compose 将多个中间件 "组合" 成一个单一的中间件，便于重用或导出
const isJSON = require('koa-is-json');  // 判断数据为否为 json 格式
const context = require('./context'); // context 模块
const request = require('./request'); // request 模块
const statuses = require('statuses'); // http 状态码
const Cookies = require('cookies'); // 记录用户信息
const accepts = require('accepts'); // 内容协商
const Emitter = require('events');  // 事件模块
const assert = require('assert'); // 断言，判断是否符合预期
const Stream = require('stream'); // 流
const http = require('http'); // http 模块，nodejs 核心模块
const only = require('only'); // 返回指定属性的对象
const convert = require('koa-convert'); // 兼容旧的 koa，转换 generator 中间件
const deprecate = require('depd')('koa'); // 判断 api 是否过期
```
### 二、源码
```js
// 暴露 Application 类，继承 Emitter
module.exports = class Application extends Emitter {
  constructor() {
    super();

    this.proxy = false;
    this.middleware = []; // 中间件数组
    this.subdomainOffset = 2; // 子域返回偏移量，用于 request 中的 get subdomains()
    this.env = process.env.NODE_ENV || 'development';
    this.context = Object.create(context);
    this.request = Object.create(request);
    this.response = Object.create(response);
  }

  // 监听函数，全写：http.createServer(app.callback()).listen(...)
  listen(...args) {
    debug('listen');
    const server = http.createServer(this.callback());
    return server.listen(...args);
  }

  // 只输出可能含 subdomainOffset、proxy、env 属性的对象，属性值对应 this 相应属性值
  toJSON() {
    return only(this, [
      'subdomainOffset',
      'proxy',
      'env'
    ]);
  }

  // 调用 toJSON 方法
  inspect() {
    return this.toJSON();
  }

  // 处理给出的中间件 fn，旧格式的中间件将会被转换，返回实例对象
  use(fn) {
    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
    if (isGeneratorFunction(fn)) {  // 判断中间件是否是 generator 函数
      deprecate('Support for generators will be removed in v3. ' +
                'See the documentation for examples of how to convert old middleware ' +
                'https://github.com/koajs/koa/blob/master/docs/migration.md');
      fn = convert(fn);
    }
    debug('use %s', fn._name || fn.name || '-');
    this.middleware.push(fn);
    return this;
  }

  // 返回请求处理回调
  callback() {
    const fn = compose(this.middleware);  // this.middleware 必须是函数数组

    if (!this.listeners('error').length) this.on('error', this.onerror);  // 添加错误默认处理函数（this.listeners('error'): 返回名为 error 的事件的监听器数组的副本）

    const handleRequest = (req, res) => { // 返回 requestListener 函数
      const ctx = this.createContext(req, res);
      return this.handleRequest(ctx, fn);
    };

    return handleRequest;
  }

  // 处理请求（私有方法）
  handleRequest(ctx, fnMiddleware) {
    const res = ctx.res;
    res.statusCode = 404;
    const onerror = err => ctx.onerror(err);
    const handleResponse = () => respond(ctx);
    onFinished(res, onerror);
    return fnMiddleware(ctx).then(handleResponse).catch(onerror);
  }

  // 初始化一个新的 context 对象（私有方法）
  createContext(req, res) {
    const context = Object.create(this.context);
    const request = context.request = Object.create(this.request);
    const response = context.response = Object.create(this.response);
    context.app = request.app = response.app = this;
    context.req = request.req = response.req = req;
    context.res = request.res = response.res = res;
    request.ctx = response.ctx = context;
    request.response = response;
    response.request = request;
    context.originalUrl = request.originalUrl = req.url;
    context.cookies = new Cookies(req, res, {
      keys: this.keys,
      secure: request.secure
    });
    request.ip = request.ips[0] || req.socket.remoteAddress || '';
    context.accept = request.accept = accepts(req);
    context.state = {};
    return context;
  }

  // 默认的错误处理函数
  onerror(err) {
    assert(err instanceof Error, `non-error thrown: ${err}`); // 类型检查，确认 err 是 Error 对象

    if (404 == err.status || err.expose) return;  // 如果 err.status 是 404 或者 err.expose 是 true，则不输出错误信息
    if (this.silent) return;  // 如果 silent 是 true，则不输出错误信息

    const msg = err.stack || err.toString();
    console.error();
    console.error(msg.replace(/^/gm, '  '));
    console.error();
  }
};

// 响应方法
function respond(ctx) {
  // 显式设置 ctx.respond = false 来绕过 koa response 封装，使用原生 res api
  if (false === ctx.respond) return;

  const res = ctx.res;
  if (!ctx.writable) return;

  let body = ctx.body;
  const code = ctx.status;

  // 处理空状态码
  if (statuses.empty[code]) {
    // strip headers
    ctx.body = null;
    return res.end();
  }

  // HEAD 方法与 GET 类似，但是 HEAD 并不返回消息体
  if ('HEAD' == ctx.method) {
    if (!res.headersSent && isJSON(body)) {
      ctx.length = Buffer.byteLength(JSON.stringify(body));
    }
    return res.end();
  }

  // 处理空回复
  if (null == body) {
    body = ctx.message || String(code);
    if (!res.headersSent) {
      ctx.type = 'text';
      ctx.length = Buffer.byteLength(body);
    }
    return res.end(body);
  }

  // 三种处理  buffer 字符串 流
  if (Buffer.isBuffer(body)) return res.end(body);
  if ('string' == typeof body) return res.end(body);
  if (body instanceof Stream) return body.pipe(res);

  // body: json（字符串序列化）
  body = JSON.stringify(body);
  if (!res.headersSent) {
    ctx.length = Buffer.byteLength(body);
  }
  res.end(body);
}
```
### 三、分析
#### 1、constructor
koa 核心代码在 lib 目录下的四个文件中，其中 application.js 为入口文件。其暴露 Application 类，这个类继承 Emitter。当引入 koa 且执行 new 操作时初始化 Application 实例并在实例上初始化一些配置，并挂载 context、request、response 实例。
#### 2、use
往 Application 实例上的 middleware 数组上添加中间件，generator 函数会被转换并警告。
#### 3、listen
调用 callback 方法获取返回函数作为 createServer 的参数，并监听相应端口。
#### 4、callback
调用 compose 将 middleware 数组组合在一起，添加错误默认处理函数，调用 createContext、handleRequest，返回 handleRequest。
#### 5、createContext
初始化一个新的 context 对象，并在对象上挂载 Application 实例、req & res 原生请求对象、request & response 类实例及一些属性。
#### 6、handleRequest
为请求添加失败回调函数，并使用中间件处理请求，后执行成功回调。