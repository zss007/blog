---
title: es6 之 Generator 异步应用
categories:
- note
---
异步编程对 JavaScript 语言太重要。Javascript 语言的执行环境是 "单线程" 的，如果没有异步编程，根本没法用，非卡死不可。现在主要介绍 Generator 函数如何完成异步操作。
<!--more-->
### 一、简介
```
// 异步编程：回调函数
fs.readFile('/etc/passwd', 'utf-8', function (err, data) {
  if (err) throw err;
  console.log(data);
});
  
// 异步编程：Promise
var readFile = require('fs-readfile-promise');
readFile(fileA)
.then(function (data) {
  console.log(data.toString());
})
.then(function () {
  return readFile(fileB);
})
.then(function (data) {
  console.log(data.toString());
})
.catch(function (err) {
  console.log(err);
});
  
// 异步编程：Generator
function* gen(x) {
  var y = yield x + 2;
  return y;
}
var g = gen(1);
g.next() // { value: 3, done: false }
g.next() // { value: undefined, done: true }
  
// 异步任务的封装：异步操作表示得很简洁，但是流程管理却不方便（即何时执行第一阶段、何时执行第二阶段）
var fetch = require('node-fetch');
function* gen(){
  var url = 'https://api.github.com/users/github';
  var result = yield fetch(url);
  console.log(result.bio);
}
var g = gen();
var result = g.next();
result.value.then(function(data){ // 由于 Fetch 模块返回的是一个 Promise 对象，因此要用 then 方法调用下一个 next 方法
  return data.json();
}).then(function(data){
  g.next(data);
});
```
### 二、JavaScript 语言的 Thunk 函数
在 JavaScript 语言中，Thunk 函数替换的是多参数函数，将其替换成一个只接受回调函数作为参数的单参数函数。
```
// 正常版本的readFile（多参数版本）
fs.readFile(fileName, callback);
// Thunk版本的readFile（单参数版本）
var Thunk = function (fileName) {
  return function (callback) {
    return fs.readFile(fileName, callback);
  };
};
var readFileThunk = Thunk(fileName);
readFileThunk(callback);
  
// Generator 函数自动执行，不过不适合异步操作。如果必须保证前一步执行完，才能执行后一步，则下面的自动执行就不可行。
function* gen() {
  // ...
}
var g = gen();
var res = g.next();
while(!res.done){
  console.log(res.value);
  res = g.next();
}
  
// Thunk 函数的自动流程管理
var fs = require('fs');
function run(generator) {
    var it = generator(go);
    function go(err, result) {
        if (err) return it.throw(err);
        it.next(result);
    }
    go();
}
run(function* (done) {
    var firstFile;
    try {
        var dirFiles = yield fs.readdir('NoNoNoNo', done); // No such dir
        firstFile = dirFiles[0];
    } catch (err) {
        firstFile = null;
    }
    console.log(firstFile);
});
```
### 三、co 模块
co 模块是著名程序员 TJ Holowaychuk 于 2013 年 6 月发布的一个小工具，用于 Generator 函数的自动执行。co 模块其实就是将两种自动执行器（Thunk 函数和 Promise 对象），包装成一个模块。使用 co 的前提条件是，Generator 函数的 yield 命令后面，只能是 Thunk 函数或 Promise 对象。如果数组或对象的成员，全部都是 Promise 对象，也可以使用 co。这样当异步操作有了结果就能够用 then 方法交回执行权。
```
// co 模块可以让你不用编写 Generator 函数的执行器，Generator 函数只要传入 co 函数，就会自动执行，并返回一个 Promise 对象
var readFile = function (fileName){
  return new Promise(function (resolve, reject){
    fs.readFile(fileName, function(error, data){
      if (error) return reject(error);
      resolve(data);
    });
  });
};
var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
var co = require('co');
co(gen);
  
// 基于 Promise 对象的自动执行
function getFoo() {
    return new Promise(function (resolve, reject) {
        resolve('foo');
    });
}
function run(generator) {
    var it = generator();
    function go(result) {
        if (result.done) return result.value;
        return result.value.then(function (value) {
            return go(it.next(value));
        }, function (error) {
            return go(it.throw(error));
        });
    }
    go(it.next());
}
run(function* () {
    try {
        var foo = yield getFoo();
        console.log(foo);
    } catch (e) {
        console.log(e);
    }
});
  
// co 源码---------------------------------------------》
var slice = Array.prototype.slice;
module.exports = co['default'] = co.co = co;
// 将生成器函数 fn 封装到函数中，这个函数执行返回 Promise
co.wrap = function (fn) {
  createPromise.__generatorFunction__ = fn;
  return createPromise;
  function createPromise() {
    return co.call(this, fn.apply(this, arguments));
  }
};
// 执行生成器函数或者迭代器，返回 Promise
function co(gen) {
  var ctx = this;
  var args = slice.call(arguments, 1);
  return new Promise(function (resolve, reject) {
    if (typeof gen === 'function') gen = gen.apply(ctx, args);  // 调用生成器函数来获得遍历器
    if (!gen || typeof gen.next !== 'function') return resolve(gen);  // 如果不是遍历器，那么直接将 Promise 状态设置为 resolved
    // 手动执行一次
    onFulfilled();
    // 成功回调函数
    function onFulfilled(res) {
      var ret;
      try {
        ret = gen.next(res);  // 首次启动遍历器遍历，将返回值给 ret 变量
      } catch (e) {
        return reject(e); // 如果执行失败将 Promise 状态设置为 rejected
      }
      next(ret);
      return null;
    }
    // 失败回调函数
    function onRejected(err) {
      var ret;
      try {
        ret = gen.throw(err); // 抛出错误
      } catch (e) {
        return reject(e); // 将 Promise 状态设置为 rejected
      }
      next(ret);
    }
    // 主要是将 onFulfilled、onRejected 函数添加到 ret.value 或者其转化的 Promise 中的回调函数中
    function next(ret) {
      if (ret.done) return resolve(ret.value); // 如果已经遍历完成，那么返回 ret.value
      var value = toPromise.call(ctx, ret.value); // 将 ret.value 转化为 Promise
      if (value && isPromise(value)) return value.then(onFulfilled, onRejected);  // 将 onFulfilled 和 onRejected 添加到 Promise 的回调函数中
      return onRejected(new TypeError('You may only yield a function, promise, generator, array, or object, '
        + 'but the following object was passed: "' + String(ret.value) + '"')); // 如果返回的数据转化失败，那么抛出错误，将 Promise 状态设置为 rejected
    }
  });
}
// 将参数 obj 转为 Promise
function toPromise(obj) {
  if (!obj) return obj;
  if (isPromise(obj)) return obj;
  if (isGeneratorFunction(obj) || isGenerator(obj)) return co.call(this, obj);  // 如果是生成器函数，那么执行 co 返回 Promise
  if ('function' == typeof obj) return thunkToPromise.call(this, obj);
  if (Array.isArray(obj)) return arrayToPromise.call(this, obj);
  if (isObject(obj)) return objectToPromise.call(this, obj);
  return obj;
}
// 将 thunk 函数转换为 Promise
function thunkToPromise(fn) {
  var ctx = this;
  return new Promise(function (resolve, reject) {
    fn.call(ctx, function (err, res) {
      if (err) return reject(err);
      if (arguments.length > 2) res = slice.call(arguments, 1);
      resolve(res);
    });
  });
}
// 将数组转换为 Promise
function arrayToPromise(obj) {
  return Promise.all(obj.map(toPromise, this));
}
// 将 obj 转换为 Promise
function objectToPromise(obj) {
  var results = new obj.constructor(); // 获取一个新的 obj 对象 results
  var keys = Object.keys(obj);  // 获取对象的 keys
  var promises = [];
  for (var i = 0; i < keys.length; i++) {
    var key = keys[i];
    var promise = toPromise.call(this, obj[key]); // 将对象的 values 转为 promise 对象
    if (promise && isPromise(promise)) defer(promise, key); // 如果转换成功执行 defer
    else results[key] = obj[key]; // 如果转换失败则传值给新对象
  }
  return Promise.all(promises).then(function () { // 将所有的 promise 合并为一个新的 promise 并添加回调函数
    return results; // 返回新对象 results
  });
  function defer(promise, key) {
    // predefine the key in the result
    results[key] = undefined;   // 将新对象 results 的对应 value 设置为 undefined
    promises.push(promise.then(function (res) { // 为新 promise 添加回调函数，并将值传入 promises 数组
      results[key] = res; // 回调函数中将新对象 results 的对应 value 设置为原值
    }));
  }
}
// 判断参数 obj 是否是 promise 对象
function isPromise(obj) {
  return 'function' == typeof obj.then;
}
// 判断参数 obj 是否是迭代器
function isGenerator(obj) {
  return 'function' == typeof obj.next && 'function' == typeof obj.throw;
}
// 判断参数 obj 是否是生成器函数或者 generator 生成的迭代器
function isGeneratorFunction(obj) {
  var constructor = obj.constructor;  // ƒ GeneratorFunction() { [native code] }
  if (!constructor) return false;
  // displayName 属性将返回函数的显示名称；name(ES6) 属性返回一个函数声明的名称
  if ('GeneratorFunction' === constructor.name || 'GeneratorFunction' === constructor.displayName) return true;
  return isGenerator(constructor.prototype); // 判断是否是 generator 生成的迭代器，constructor.prototype 返回 Generator 对象
}
// 判断是否为简单对象
function isObject(val) {
  return Object == val.constructor;
}
```
### 四、Generator 应用
```
// 异步操作的同步化表达
function* main() {
  var result = yield request("http://some.url");
  var resp = JSON.parse(result);
    console.log(resp.value);
}
function request(url) {
  makeAjaxCall(url, function(response){
    it.next(response);
  });
}
var it = main();
it.next();
  
// 控制流管理（这里只适合同步操作，异步操作使用 Thunk 或者 co）
scheduler(longRunningTask(initialValue));
function scheduler(task) {
  var taskObj = task.next(task.value);
  // 如果Generator函数未结束，就继续调用
  if (!taskObj.done) {
    task.value = taskObj.value
    scheduler(task);
  }
}
  
// 部署 Iterator 接口
function* iterEntries(obj) {
  let keys = Object.keys(obj);
  for (let i=0; i < keys.length; i++) {
    let key = keys[i];
    yield [key, obj[key]];
  }
}
let myObj = { foo: 3, bar: 7 };
for (let [key, value] of iterEntries(myObj)) {
  console.log(key, value);
}
  
// 作为数据结构（Generator 使得数据或者操作，具备了类似数组的接口）
function* doStuff() {
  yield fs.readFile.bind(null, 'hello.txt');
  yield fs.readFile.bind(null, 'world.txt');
  yield fs.readFile.bind(null, 'and-such.txt');
}
for (task of doStuff()) {
  // task是一个函数，可以像回调函数那样使用它
}
function doStuff() {  // 可以用数组模拟 Generator 的这种用法
  return [
    fs.readFile.bind(null, 'hello.txt'),
    fs.readFile.bind(null, 'world.txt'),
    fs.readFile.bind(null, 'and-such.txt')
  ];
}
```
Generator 可以暂停函数执行，返回任意表达式的值。这种特点使得 Generator 有多种应用场景。