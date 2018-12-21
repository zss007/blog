---
title: koa 之 response
categories:
- node
---
Koa Response 对象是在 node 的响应对象之上的抽象，提供了诸多对 HTTP 服务器开发有用的功能。
<!--more-->
### 一、源码
```js
const contentDisposition = require('content-disposition');  // 解析 HTTP 的 Content-Disposition
const ensureErrorHandler = require('error-inject'); // 注入失败回调函数到 stream
const getType = require('mime-types').contentType;  // 根据字符串或文件扩展名等获取相应 Content-Type
const onFinish = require('on-finished');  // 当 http 请求关闭、完成或报错时执行回调函数
const isJSON = require('koa-is-json');  // 判断参数是否为 JSON 数据，排除法
const escape = require('escape-html');  // 转换 html 字符串
const typeis = require('type-is').is; // 推断请求的 Content-Type 属性
const statuses = require('statuses'); // 状态码
const destroy = require('destroy'); // 销毁流
const assert = require('assert'); // 断言
const extname = require('path').extname;  // 返回 path 的扩展名
const vary = require('vary'); // 设置 http 的 Vary 头部属性
const only = require('only'); // 返回指定属性的对象

// Response 的原型对象
module.exports = {
  // 获取请求 socket
  get socket() {
    return this.ctx.req.socket;
  },

  // 获取响应标头对象
  get header() {
    const { res } = this;
    return typeof res.getHeaders === 'function'
      ? res.getHeaders()
      : res._headers || {};  // Node < 7.7
  },

  // 获取响应标头对象，别名是 response.header
  get headers() {
    return this.header;
  },

  // 获取响应状态
  get status() {
    return this.res.statusCode;
  },

  // 通过数字代码设置响应状态
  set status(code) {
    if (this.headerSent) return;  // 如果响应头已被发送，则不作任何操作

    assert('number' == typeof code, 'status code must be a number');
    assert(statuses[code], `invalid status code: ${code}`);
    this._explicitStatus = true;
    this.res.statusCode = code;
    // this.req.httpVersionMajor：返回客户端发送的 HTTP 版本
    if (this.req.httpVersionMajor < 2) this.res.statusMessage = statuses[code];
    if (this.body && statuses.empty[code]) this.body = null;
  },

  // 获取响应的状态消息
  get message() {
    return this.res.statusMessage || statuses[this.status];
  },

  // 将响应的状态消息设置为给定值
  set message(msg) {
    this.res.statusMessage = msg;
  },

  // 获取响应主体
  get body() {
    return this._body;
  },

  // 设置响应体（val: String | Buffer| Object | Stream）
  set body(val) {
    const original = this._body;
    this._body = val;

    // no content
    if (null == val) {
      if (!statuses.empty[this.status]) this.status = 204;
      this.remove('Content-Type');
      this.remove('Content-Length');
      this.remove('Transfer-Encoding');
      return;
    }

    // set the status
    if (!this._explicitStatus) this.status = 200;

    // set the content-type only if not yet set
    const setType = !this.header['content-type'];

    // string
    if ('string' == typeof val) {
      if (setType) this.type = /^\s*</.test(val) ? 'html' : 'text';
      this.length = Buffer.byteLength(val);
      return;
    }

    // buffer
    if (Buffer.isBuffer(val)) {
      if (setType) this.type = 'bin';
      this.length = val.length;
      return;
    }

    // stream
    if ('function' == typeof val.pipe) {
      onFinish(this.res, destroy.bind(null, val));
      ensureErrorHandler(val, err => this.ctx.onerror(err));

      // overwriting
      if (null != original && original != val) this.remove('Content-Length');

      if (setType) this.type = 'bin';
      return;
    }

    // json
    this.remove('Content-Length');
    this.type = 'json';
  },

  // 将响应的 Content-Length 设置为给定值
  set length(n) {
    this.set('Content-Length', n);
  },

  // 以数字返回响应的 Content-Length，或者从 ctx.body 推导出来，或者 undefined
  get length() {
    const len = this.header['content-length'];
    const body = this.body;

    if (null == len) {
      if (!body) return;
      if ('string' == typeof body) return Buffer.byteLength(body);
      if (Buffer.isBuffer(body)) return body.length;
      if (isJSON(body)) return Buffer.byteLength(JSON.stringify(body));
      return;
    }

    // 转为数字，并去掉小数部分
    return ~~len;
  },

  // 返回一个布尔值（只读）。如果响应头已被发送则为 true，否则为 false
  get headerSent() {
    return this.res.headersSent;
  },

  // 在 field 上的 Vary
  vary(field) {
    if (this.headerSent) return;

    vary(this.res, field);
  },

  // 执行 302 重定向到 url，如：this.redirect('back'); | this.redirect('back', '/index.html');
  // | this.redirect('/login'); | this.redirect('http://google.com');
  redirect(url, alt) {
    // location
    if ('back' == url) url = this.ctx.get('Referrer') || alt || '/';
    this.set('Location', url);

    // status
    if (!statuses.redirect[this.status]) this.status = 302;

    // html
    if (this.ctx.accepts('html')) {
      url = escape(url);
      this.type = 'text/html; charset=utf-8';
      this.body = `Redirecting to <a href="${url}">${url}</a>.`;
      return;
    }

    // text
    this.type = 'text/plain; charset=utf-8';
    this.body = `Redirecting to ${url}.`;
  },

  // 将 Content-Disposition 设置为 "附件" 以指示客户端提示下载，参数 filename 可选
  attachment(filename) {
    if (filename) this.type = extname(filename);  // 获取文件拓展名
    this.set('Content-Disposition', contentDisposition(filename));
  },

  // 通过 mime 字符串或文件扩展名设置响应 Content-Type
  set type(type) {
    type = getType(type); // 根据字符串或文件扩展名等获取相应 Content-Type，如：.html、html、json、application/json、png
    if (type) {
      this.set('Content-Type', type);
    } else {
      this.remove('Content-Type');
    }
  },

  // 将 Last-Modified 标头设置为适当的 UTC 字符串。参数可以是 Date 或日期字符串
  set lastModified(val) {
    if ('string' == typeof val) val = new Date(val);
    this.set('Last-Modified', val.toUTCString());
  },

  // 如果存在 Last-Modified，则将其返回为 Date, 
  get lastModified() {
    const date = this.get('last-modified');
    if (date) return new Date(date);
  },

  // 设置包含 " 包裹的 ETag 响应
  set etag(val) {
    if (!/^(W\/)?"/.test(val)) val = `"${val}"`;
    this.set('ETag', val);
  },

  // 获取响应对象的 ETag 属性
  get etag() {
    return this.get('ETag');
  },

  // 获取响应 Content-Type 不含参数 "charset"
  get type() {
    const type = this.get('Content-Type');
    if (!type) return '';
    return type.split(';')[0];
  },

  // 非常类似 ctx.request.is()。检查响应类型是否是所提供的类型之一，这对于创建操纵响应的中间件特别有用
  is(types) {
    const type = this.type;
    if (!types) return type || false;
    if (!Array.isArray(types)) types = [].slice.call(arguments);
    return typeis(type, types);
  },

  // 不区分大小写获取响应标头字段值 field，如： this.get('content-type') => "text/plain"
  get(field) {
    return this.header[field.toLowerCase()] || '';
  },

  // 设置响应标头 field 到 value，如：this.set('Foo', ['bar', 'baz']); | this.set('Accept', 'application/json'); | this.set({ Accept: 'text/plain', 'X-API-Key': 'tobi' });
  set(field, val) {
    if (this.headerSent) return;

    if (2 == arguments.length) {
      if (Array.isArray(val)) val = val.map(String);
      else val = String(val);
      this.res.setHeader(field, val);
    } else {
      for (const key in field) {
        this.set(key, field[key]);
      }
    }
  },

  // 用值 val 附加额外的标头 field，比如：this.append('Link', ['<http://localhost/>', '<http://localhost:3000/>']); |
  // this.append('Set-Cookie', 'foo=bar; Path=/; HttpOnly'); | this.append('Warning', '199 Miscellaneous warning');
  append(field, val) {
    const prev = this.get(field);

    if (prev) {
      val = Array.isArray(prev)
        ? prev.concat(val)
        : [prev].concat(val);
    }

    return this.set(field, val);
  },

  // 删除标头 field
  remove(field) {
    if (this.headerSent) return;

    this.res.removeHeader(field);
  },

  // 检查请求是否 writable（需检查 socket 是否存在）
  get writable() {
    // can't write any more after response finished
    if (this.res.finished) return false;  // 表示响应是否已完成，如果已完成则返回 false

    const socket = this.res.socket;
    // There are already pending outgoing res, but still writable
    // https://github.com/nodejs/node/blob/v4.4.7/lib/_http_server.js#L486
    if (!socket) return true; // 如果不存在 socket，则返回 true
    return socket.writable;
  },

  // 为特定属性的对象添加 body 属性
  inspect() {
    if (!this.res) return;
    const o = this.toJSON();
    o.body = this.body;
    return o;
  },

  // 返回只有特定属性的对象
  toJSON() {
    return only(this, [
      'status',
      'message',
      'header'
    ]);
  },

  // 刷新任何设置的标头，并开始主体
  flushHeaders() {
    this.res.flushHeaders();
  }
};
```