---
title: node 之 http-errors
categories:
- node
---
为 Express、Koa 或 Connect 生成相关 error 对象。
<!--more-->
### 一、源码
```js
/**
 * Module dependencies.
 * @private
 */
var deprecate = require('depd')('http-errors')
var setPrototypeOf = require('setprototypeof')
var statuses = require('statuses')
var inherits = require('inherits')

/**
 * Module exports.
 * @public
 */
module.exports = createError
module.exports.HttpError = createHttpErrorConstructor()

// Populate exports for all constructors
populateConstructorExports(module.exports, statuses.codes, module.exports.HttpError)

/**
 * Get the code class of a status code.
 * @private
 */
function codeClass (status) {
  return Number(String(status).charAt(0) + '00')
}

/**
 * Create a new HTTP Error.
 *
 * @returns {Error}
 * @public
 */
function createError () {
  // so much arity going on ~_~
  var err
  var msg
  var status = 500
  var props = {}
  for (var i = 0; i < arguments.length; i++) {
    var arg = arguments[i]
    if (arg instanceof Error) {
      err = arg
      status = err.status || err.statusCode || status
      continue
    }
    switch (typeof arg) {
      case 'string':
        msg = arg
        break
      case 'number':
        status = arg
        if (i !== 0) {
          deprecate('non-first-argument status code; replace with createError(' + arg + ', ...)')
        }
        break
      case 'object':
        props = arg
        break
    }
  }

  if (typeof status === 'number' && (status < 400 || status >= 600)) {
    deprecate('non-error status code; use only 4xx or 5xx status codes')
  }

  if (typeof status !== 'number' ||
    (!statuses[status] && (status < 400 || status >= 600))) {
    status = 500
  }

  // constructor
  var HttpError = createError[status] || createError[codeClass(status)]

  if (!err) {
    // create error
    err = HttpError
      ? new HttpError(msg)
      : new Error(msg || statuses[status])
    Error.captureStackTrace(err, createError)
  }

  if (!HttpError || !(err instanceof HttpError) || err.status !== status) {
    // add properties to generic error
    err.expose = status < 500
    err.status = err.statusCode = status
  }

  for (var key in props) {
    if (key !== 'status' && key !== 'statusCode') {
      err[key] = props[key]
    }
  }

  return err
}

/**
 * Create HTTP error abstract base class.
 * @private
 */
function createHttpErrorConstructor () {
  function HttpError () {
    throw new TypeError('cannot construct abstract class')
  }

  inherits(HttpError, Error)

  return HttpError
}

/**
 * Create a constructor for a client error.
 * @private
 */
function createClientErrorConstructor (HttpError, name, code) {
  var className = name.match(/Error$/) ? name : name + 'Error'

  function ClientError (message) {
    // create the error object
    var msg = message != null ? message : statuses[code]
    var err = new Error(msg)

    // capture a stack trace to the construction point
    Error.captureStackTrace(err, ClientError)

    // adjust the [[Prototype]]
    setPrototypeOf(err, ClientError.prototype)

    // redefine the error message
    Object.defineProperty(err, 'message', {
      enumerable: true,
      configurable: true,
      value: msg,
      writable: true
    })

    // redefine the error name
    Object.defineProperty(err, 'name', {
      enumerable: false,
      configurable: true,
      value: className,
      writable: true
    })

    return err
  }

  inherits(ClientError, HttpError)

  ClientError.prototype.status = code
  ClientError.prototype.statusCode = code
  ClientError.prototype.expose = true

  return ClientError
}

/**
 * Create a constructor for a server error.
 * @private
 */
function createServerErrorConstructor (HttpError, name, code) {
  var className = name.match(/Error$/) ? name : name + 'Error'

  function ServerError (message) {
    // create the error object
    var msg = message != null ? message : statuses[code]
    var err = new Error(msg)

    // capture a stack trace to the construction point
    Error.captureStackTrace(err, ServerError)

    // adjust the [[Prototype]]
    setPrototypeOf(err, ServerError.prototype)

    // redefine the error message
    Object.defineProperty(err, 'message', {
      enumerable: true,
      configurable: true,
      value: msg,
      writable: true
    })

    // redefine the error name
    Object.defineProperty(err, 'name', {
      enumerable: false,
      configurable: true,
      value: className,
      writable: true
    })

    return err
  }

  inherits(ServerError, HttpError)

  ServerError.prototype.status = code
  ServerError.prototype.statusCode = code
  ServerError.prototype.expose = false

  return ServerError
}

/**
 * Populate the exports object with constructors for every error class.
 * @private
 */
function populateConstructorExports (exports, codes, HttpError) {
  codes.forEach(function forEachCode (code) {
    var CodeError
    var name = toIdentifier(statuses[code])

    switch (codeClass(code)) {
      case 400:
        CodeError = createClientErrorConstructor(HttpError, name, code)
        break
      case 500:
        CodeError = createServerErrorConstructor(HttpError, name, code)
        break
    }

    if (CodeError) {
      // export the constructor
      exports[code] = CodeError
      exports[name] = CodeError
    }
  })

  // backwards-compatibility
  exports["I'mateapot"] = deprecate.function(exports.ImATeapot,
    '"I\'mateapot"; use "ImATeapot" instead')
}

/**
 * Convert a string of words to a JavaScript identifier.
 * @private
 */
function toIdentifier (str) {
  return str.split(' ').map(function (token) {
    return token.slice(0, 1).toUpperCase() + token.slice(1)
  }).join('').replace(/[^ _0-9a-z]/gi, '')
}
```
### 二、分析
- 1、执行 populateConstructorExports 方法，遍历常用状态码，具体根据状态码执行 createClientErrorConstructor 或 createServerErrorConstructor，返回自定义错误对象，并赋值给 exports，即 createError。
- 2、导出 createError，根据传入自定义错误对象，允许不同组合：createError(400) | createError(400, 'name required') | createError(400, 'name required', { user: user }) | this.throw(400, new Error('invalid')) 等。
- 3、错误对象的属性 expose = true 时，application 的 onerror 回调对其不输出错误信息，context 的 onerror 回调让 res 发送错误对象的 message 属性，否则 context 发送状态码。

### 三、拓展
#### 1、调用栈的工作机制
函数被调用时，就会被加入到调用栈顶部，执行结束之后，就会从调用栈顶部移除该函数，这种数据结构的关键在于后进先出，即大家所熟知的 LIFO。比如，当我们在函数 y 内部调用函数 x 的时候，调用栈从下往上的顺序就是 y -> x 。
```js
function c() {
    console.log('c');
}

function b() {
    console.log('b');
    c();
    console.trace(); // 使用 console.trace 来把当前的调用栈输出到 console 中
}

function a() {
    console.log('a');
    b();
}

a();

// 得到如下的结果：调用函数的时候，会被推到调用栈的顶部，而执行完毕之后，就会从调用栈移除。
Trace
    at b (repl:4:9)
    at a (repl:3:1)
    at repl:1:1  // <-- 从这行往下的内容可以忽略，因为这些都是 Node 内部的东西
    at realRunInThisContextScript (vm.js:22:35)
    at sigintHandlersWrap (vm.js:98:12)
    at ContextifyScript.Script.runInThisContext (vm.js:24:12)
    at REPLServer.defaultEval (repl.js:313:29)
    at bound (domain.js:280:14)
    at REPLServer.runBound [as eval] (domain.js:293:12)
    at REPLServer.onLine (repl.js:513:10)
```
#### 2、Error 对象及错误处理
当代码中发生错误时，我们通常会抛出一个 Error 对象。Error 对象可以作为扩展和创建自定义错误类型的原型。Error 对象的 prototype 具有以下属性：constructor 负责该实例的原型构造函数；message 错误信息；name 错误的名字。
上面都是标准属性，有些 JS 运行环境还提供了标准属性之外的属性，如 Node.js、Firefox、Chrome、Edge、IE 10、Opera 和 Safari 6+ 中会有 stack 属性，它包含了错误代码的调用栈，接下来我们简称错误堆栈。错误堆栈包含了产生该错误时完整的调用栈信息。
抛出错误时，你必须使用 throw 关键字。为了捕获抛出的错误，则必须使用 try catch 语句把可能出错的代码块包起来，catch 的时候可以接收一个参数，该参数就是被抛出的错误。与 Java 中类似，JS 中也可以在 try catch 语句之后有 finally，不论前面代码是否抛出错误 finally 里面的代码都会执行，这种语言的常见用途有：在 finally 中做些清理的工作。此外，你可以使用没有 catch 的 try 语句，但是后面必须跟上 finally，这意味着我们可以使用三种不同形式的 try 语句：try … catch、try … finally、try … catch … finally
```js
try {
    console.log('The try block is running...');
} finally {
    try {
        // 强烈建议抛出 Error 对象，否则在需要处理错误的调用栈信息和其他有意义的元数据，抛出非 Error 对象的错误会让你的处境很尴尬
        throw new Error('Error inside finally.');
    } catch (err) {
        console.log('Caught an error inside the finally block.');
    }
}
```
#### 3、错误堆栈的裁剪
Node.js 才支持这个特性，通过 Error.captureStackTrace 来实现，Error.captureStackTrace 接收一个 object 作为第 1 个参数，以及可选的 function 作为第 2 个参数。其作用是捕获当前的调用栈并对其进行裁剪，捕获到的调用栈会记录在第 1 个参数的 stack 属性上，裁剪的参照点是第 2 个参数，也就是说，此函数之前的调用会被记录到调用栈上面，而之后的不会。
```js
// 让我们用代码来说明，首先，把当前的调用栈捕获并放到 myObj 上：
const myObj = {};

function c() {
}

function b() {
    // 把当前调用栈写到 myObj 上
    Error.captureStackTrace(myObj);
    c();
}

function a() {
    b();
}

// 调用函数 a
a();

// 打印 myObj.stack
console.log(myObj.stack);

// 输出会是这样
//    at b (repl:3:7) <-- Since it was called inside B, the B call is the last entry in the stack
//    at a (repl:2:1)
//    at repl:1:1 <-- Node internals below this line
//    at realRunInThisContextScript (vm.js:22:35)
//    at sigintHandlersWrap (vm.js:98:12)
//    at ContextifyScript.Script.runInThisContext (vm.js:24:12)
//    at REPLServer.defaultEval (repl.js:313:29)
//    at bound (domain.js:280:14)
//    at REPLServer.runBound [as eval] (domain.js:293:12)
//    at REPLServer.onLine (repl.js:513:10)
```
上面的调用栈中只有 a -> b，因为我们在 b 调用 c 之前就捕获了调用栈。现在对上面的代码稍作修改，然后看看会发生什么：
```js
const myObj = {};

function d() {
    // 我们把当前调用栈存储到 myObj 上，但是会去掉 b 和 b 之后的部分
    Error.captureStackTrace(myObj, b);
}

function c() {
    d();
}

function b() {
    c();
}

function a() {
    b();
}

// 执行代码
a();

// 打印 myObj.stack
console.log(myObj.stack);

// 输出如下
//    at a (repl:2:1) <-- As you can see here we only get frames before `b` was called
//    at repl:1:1 <-- Node internals below this line
//    at realRunInThisContextScript (vm.js:22:35)
//    at sigintHandlersWrap (vm.js:98:12)
//    at ContextifyScript.Script.runInThisContext (vm.js:24:12)
//    at REPLServer.defaultEval (repl.js:313:29)
//    at bound (domain.js:280:14)
//    at REPLServer.runBound [as eval] (domain.js:293:12)
//    at REPLServer.onLine (repl.js:513:10)
//    at emitOne (events.js:101:20)
```
在这段代码里面，因为我们在调用 Error.captureStackTrace 的时候传入了 b，这样 b 之后的调用栈都会被隐藏。可以在想对用户隐藏跟他业务无关的错误堆栈（比如某个库的内部实现）试用这个技巧。