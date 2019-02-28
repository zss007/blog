---
title: koa 之 request
categories:
- node
---
Koa Request 对象是在 node 的请求对象之上的抽象，提供了诸多对 HTTP 服务器开发有用的功能。
<!--more-->
### 一、源码
```js
const URL = require('url').URL; // 获取 URL 构造函数
const net = require('net'); // 网络
const contentType = require('content-type');  // 根据 RFC 协议解析 HTTP 的 Content-Type 头部
const stringify = require('url').format;  // url 的格式化方法
const parse = require('parseurl');  // 基本上跟 url.parse 相同，不过使用同一个 req 对象解析时，直接用缓存中取，不再解析
const qs = require('querystring');  // 查询字符串
const typeis = require('type-is');  // 推断请求的 Content-Type 属性
const fresh = require('fresh'); // 根据请求头和响应头判断是否缓存
const only = require('only'); // 筛选对象的属性

// Request 的原型对象
module.exports = {
  // 返回请求头
  get header() {
    return this.req.headers;
  },

  // 设置请求头
  set header(val) {
    this.req.headers = val;
  },

  // 返回请求头，别名为 request.header
  get headers() {
    return this.req.headers;
  },

  // 设置请求头，别名为 request.header
  set headers(val) {
    this.req.headers = val;
  },

  // 获取请求的 URL
  get url() {
    return this.req.url;
  },

  // 设置请求的 URL
  set url(val) {
    this.req.url = val;
  },

  // 获取请求原始 URL，调用自身的 get protocol() 和 get host() 方法
  get origin() {
    return `${this.protocol}://${this.host}`;
  },

  // 获取完整的请求URL，包括 protocol，host 和 url
  get href() {
    // this.originalUrl 在 application.js 中 被赋值为 req.url，即请求的 URL 字符串
    if (/^https?:\/\//i.test(this.originalUrl)) return this.originalUrl;  // 如果 originalUrl 含有协议和主机信息，那么直接返回，如 http://example.com/foo
    return this.origin + this.originalUrl;  // 否则加上请求原始 URL
  },

  // 获取请求方法
  get method() {
    return this.req.method;
  },

  // 设置请求方法
  set method(val) {
    this.req.method = val;
  },

  // 获取请求路径名
  get path() {
    return parse(this.req).pathname;
  },

  // 设置请求路径名 pathname，并在存在时保留查询字符串
  set path(path) {
    const url = parse(this.req);
    if (url.pathname === path) return;

    url.pathname = path;
    url.path = null;

    this.url = stringify(url);
  },

  // 获取解析的查询字符串（有缓存处理）
  get query() {
    const str = this.querystring;
    const c = this._querycache = this._querycache || {};
    return c[str] || (c[str] = qs.parse(str));
  },

  // 将查询字符串设置为给定对象
  set query(obj) {
    this.querystring = qs.stringify(obj);
  },

  // 获取原始查询字符串
  get querystring() {
    if (!this.req) return '';
    return parse(this.req).query || '';
  },

  // 设置原始查询字符串
  set querystring(str) {
    const url = parse(this.req);
    if (url.search === `?${str}`) return;

    url.search = str;
    url.path = null;

    this.url = stringify(url);
  },

  // 使用 ? 获取原始查询字符串
  get search() {
    if (!this.querystring) return '';
    return `?${this.querystring}`;
  },

  // 设置原始查询字符串
  set search(str) {
    this.querystring = str;
  },

  // 获取主机（host）部分
  get host() {
    const proxy = this.app.proxy; // 当代理时，使用 X-Forwarded-Host
    // 反向代理（如负载均衡服务器、CDN 等）的域名或端口号可能会与处理请求的源头服务器有所不同，在这种情况下，X-Forwarded-Host 可以用来确定哪一个域名是最初被用来访问的
    let host = proxy && this.get('X-Forwarded-Host');
    host = host || this.get('Host');  // 从请求的 headers 中拿到 host
    if (!host) return '';
    return host.split(/\s*,\s*/)[0];
  },

  // 获取主机名，不含端口号
  get hostname() {
    const host = this.host;
    if (!host) return '';
    if ('[' == host[0]) return this.URL.hostname || ''; // IPv6
    return host.split(':')[0];
  },

  // 获取 WHATWG 解析的 URL 对象
  get URL() {
    if (!this.memoizedURL) {
      const protocol = this.protocol;
      const host = this.host;
      const originalUrl = this.originalUrl || ''; // avoid undefined in template string
      try {
        this.memoizedURL = new URL(`${protocol}://${host}${originalUrl}`);
      } catch (err) {
        this.memoizedURL = Object.create(null);
      }
    }
    return this.memoizedURL;
  },

  // 检查请求缓存是否 "新鲜"，也就是内容没有改变，用于 If-None-Match / ETag, 和 If-Modified-Since 和 Last-Modified 之间的缓存协商
  get fresh() {
    const method = this.method;
    const s = this.ctx.status;

    // GET or HEAD for weak freshness validation only
    if ('GET' != method && 'HEAD' != method) return false;

    // 2xx or 304 as per rfc2616 14.26
    if ((s >= 200 && s < 300) || 304 == s) {
      return fresh(this.header, this.ctx.response.header);
    }

    return false;
  },

  // 与 request.fresh 相反
  get stale() {
    return !this.fresh;
  },

  // 判断方法是否在 methods 中
  get idempotent() {
    const methods = ['GET', 'HEAD', 'PUT', 'DELETE', 'OPTIONS', 'TRACE'];
    return !!~methods.indexOf(this.method);
  },

  // 返回与连接关联的 net.Socket 对象
  get socket() {
    return this.req.socket;
  },

  // 在存在时获取请求字符集，或者 undefined
  get charset() {
    let type = this.get('Content-Type');
    if (!type) return '';

    try {
      type = contentType.parse(type);
    } catch (e) {
      return '';
    }

    return type.parameters.charset || '';
  },

  // 返回以数字返回请求的 Content-Length，或 undefined
  get length() {
    const len = this.get('Content-Length');
    if (len == '') return;
    return ~~len; // ~~：利用符号进行的类型转换，转换成数字类型
  },

  // 获取请求协议：当用 TLS 请求时返回协议字符串 http 或者 https，当 proxy 为 true 时返回 X-Forwarded-Proto 头部属性对应的值
  get protocol() {
    const proxy = this.app.proxy;
    if (this.socket.encrypted) return 'https';
    if (!proxy) return 'http';
    const proto = this.get('X-Forwarded-Proto') || 'http';
    return proto.split(/\s*,\s*/)[0];
  },

  // this.protocol == 'https' 的简称
  get secure() {
    return 'https' == this.protocol;
  },

  // 当 X-Forwarded-For 存在并且 app.proxy 被启用时，这些 ips 的数组被返回。禁用时返回一个空数组
  get ips() {
    const proxy = this.app.proxy;
    // X-Forwarded-For（XFF）在客户端访问服务器的过程中如果需要经过 HTTP 代理或者负载均衡服务器，可以被用来获取最初发起请求的客户端的 IP 地址
    // 例如值为 "client, proxy1, proxy2"，返回 ["client", "proxy1", "proxy2"]，从上游 -> 下游排序
    const val = this.get('X-Forwarded-For');
    return proxy && val
      ? val.split(/\s*,\s*/)
      : [];
  },

  // 将子域返回为数组
  get subdomains() {
    const offset = this.app.subdomainOffset;
    const hostname = this.hostname;
    if (net.isIP(hostname)) return [];  // 如果主机名是 ip 地址，则返回空数组
    // tobi.ferrets.example.com => subdomainOffset 为 2 时，返回 ["ferrets", "tobi"]；subdomainOffset 为 3 时，返回 ["tobi"]
    return hostname
      .split('.')
      .reverse()
      .slice(offset);
  },

  // Koa 的 request 对象包括由 accepts 和 negotiator 提供的有用的内容协商实体
  // 检查给定的 type(s) 是否可以接受，如果 true，返回最佳匹配，否则为 false
  /**
   *     // Accept: text/html
   *     this.accepts('html');
   *     // => "html"
   *
   *     // Accept: text/*, application/json
   *     this.accepts('html');
   *     // => "html"
   *     this.accepts('text/html');
   *     // => "text/html"
   *     this.accepts('json', 'text');
   *     // => "json"
   *     this.accepts('application/json');
   *     // => "application/json"
   *
   *     // Accept: text/*, application/json
   *     this.accepts('image/png');
   *     this.accepts('png');
   *     // => false
   *
   *     // Accept: text/*;q=.5, application/json
   *     this.accepts(['html', 'json']);
   *     this.accepts('html', 'json');
   *     // => "json"
   */
  accepts(...args) {
    return this.accept.types(...args);
  },

  // 检查 encodings 是否可以接受，返回最佳匹配为 true，否则为 false。Accept-Encoding: gzip, deflate -> ['gzip', 'deflate']
  acceptsEncodings(...args) {
    return this.accept.encodings(...args);
  },

  // 检查 charsets 是否可以接受，在 true 时返回最佳匹配，否则为 false。Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5 -> ['utf-8', 'utf-7', 'iso-8859-1']
  acceptsCharsets(...args) {
    return this.accept.charsets(...args);
  },

  // 检查 langs 是否可以接受，如果为 true，返回最佳匹配，否则为 false。Accept-Language: en;q=0.8, es, pt -> ['es', 'pt', 'en']
  acceptsLanguages(...args) {
    return this.accept.languages(...args);
  },

  // 检查传入请求是否包含 Content-Type 头字段， 并且包含任意的 mime type。 如果没有请求主体，返回 null。 如果没有内容类型，或者匹配失败，则返回 false。 反之则返回匹配的 content-type
  /**
   *     // With Content-Type: text/html; charset=utf-8
   *     this.is('html'); // => 'html'
   *     this.is('text/html'); // => 'text/html'
   *     this.is('text/*', 'application/json'); // => 'text/html'
   *
   *     // When Content-Type is application/json
   *     this.is('json', 'urlencoded'); // => 'json'
   *     this.is('application/json'); // => 'application/json'
   *     this.is('html', 'application/*'); // => 'application/json'
   *
   *     this.is('html'); // => false
   */
  is(types) {
    if (!types) return typeis(this.req);
    if (!Array.isArray(types)) types = [].slice.call(arguments);
    return typeis(this.req, types);
  },

  // 返回请求的 Content-Type，不含参数 charset
  get type() {
    const type = this.get('Content-Type');
    if (!type) return '';
    return type.split(';')[0];
  },

  // 返回请求头部，如：this.get('Content-Type'); => "text/plain" | this.get('content-type'); => "text/plain" | this.get('Something'); => undefined
  // Referrer 头部属性是特例，Referrer 和 Referer 可互换，历史遗留问题，http 协议命令为 Referer，实际上是 Referrer（主要用于重定向）
  get(field) {
    const req = this.req;
    switch (field = field.toLowerCase()) {
      case 'referer':
      case 'referrer':
        return req.headers.referrer || req.headers.referer || '';
      default:
        return req.headers[field] || '';
    }
  },

  // 监听 toJSON 实现
  inspect() {
    if (!this.req) return;
    return this.toJSON();
  },

  // 返回只有特定属性的对象
  toJSON() {
    return only(this, [
      'method',
      'url',
      'header'
    ]);
  }
};
```
### 二、分析
<img src="/assets/node/url.jpg">
这里主要讲下node 对 URL 的处理：一个是 Node.js 遗留的特有的 API，另一个则是通常使用在 web 浏览器中 实现了 WHATWG URL Standard 的 API。虽然 Node.js 遗留的特有的 API 并没有被弃用，但是保留的目的是用于向后兼容已有应用程序。因此新的应用程序请使用 WHATWG API。
WHATWG 与Node.js 遗留的特有的 API 的比较如上。网址`http://user:pass@sub.host.com:8080/p/a/t/h?query=string#hash`上方是由遗留的 url.parse() 返回的对象属性，下方的则是 WHATWG URL 对象的属性。
```js
// 利用 WHATWG API 解析一个 URL 字符串
const { URL } = require('url');
const myURL = new URL('https://user:pass@sub.host.com:8080/p/a/t/h?query=string#hash');

// 通过 Node.js 提供的 API 解析一个 URL
const url = require('url');
const myURL = url.parse('https://user:pass@sub.host.com:8080/p/a/t/h?query=string#hash');
```
在浏览器中，WHATWG URL 在全局总是可用的，而在 Node.js 中，任何情况下打开或使用一个链接都必须事先引用 'url' 模块：require('url').URL。