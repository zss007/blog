---
title: es6 之 promise
categories:
- note
---
异步编程传统做法是使用回调函数，而 promise 提供了一种更加合理、更加强大的解决方案，避免出现地狱回调。在前面的文章介绍 zepto 源码时有讲到 promise 实现方案，而现在 ES6 将其写进了语言标准，原生提供了 promise 对象。
<!--more-->
### 一、resolve/reject
ES6 规定，promise 对象是一个构造函数，用来生成 promise 实例，该构造函数接受一个函数作为参数，该函数的两个参数分别是 resolve 和 reject。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。
```
// 300ms 后打印出 "FULFILLED"
var promise = new Promise((fulfill, reject) => {
    setTimeout(() => {
        fulfill('FULFILLED!');
    }, 300);
});

promise.then((val) => {
    console.log(val);
});
   
// 300ms 后打印出 "REJECTED"
var promise = new Promise(function (fulfill, reject) {
    setTimeout(() => {
        reject(new Error('REJECTED!'));
    }, 300);
});

function onReject(error) {
    console.log(error.message);
}

promise.then(null, onReject);
```
### 二、state
resolve 函数的作用是，将 Promise 对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；reject 函数的作用是，将 Promise 对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去；一旦状态改变，就不会再变。
```
// 打印出 "I FIRED"
var promise = new Promise((fulfill, reject) => {
    fulfill('I FIRED');
    reject(new Error('I DID NOT FIRE'));
});

function onReject(error) {
    console.log(error.message);
}

promise.then(console.log, onReject);
```
### 三、asynchronously
promise 可以将异步操作改成同步写法，避免地狱回调，但是其实 promise 在执行中还是异步的。
```
var promise = new Promise((fulfill, reject) => {
    fulfill('PROMISE VALUE');
});

promise.then(console.log);

console.log('MAIN PROGRAM');
```
### 四、shortcuts
Promise.resolve、Promise.reject 是 promise 构造函数语法糖，可用于快速得到 promise 对象。
#### 4.1、Promise.resolve 参数
- 1、参数是一个 Promise 实例
如果参数是 Promise 实例，那么 Promise.resolve 将不做任何修改、原封不动地返回这个实例。
- 2、参数是一个 thenable 对象
thenable对象指的是具有then方法的对象，Promise.resolve 方法会将这个对象转为 Promise 对象，然后就立即执行 thenable 对象的 then 方法。
- 3、参数不是具有 then 方法的对象，或根本就不是对象
如果参数是一个原始值，或者是一个不具有 then 方法的对象，则 Promise.resolve 方法返回一个新的 Promise 对象，状态为 resolved。
- 4、不带有任何参数
Promise.resolve 方法允许调用时不带参数，直接返回一个 resolved 状态的 Promise 对象。

#### 4.2、Promise.reject 参数
Promise.reject(reason) 方法也会返回一个新的 Promise 实例，该实例的状态为 rejected。注意，Promise.reject() 方法的参数，会原封不动地作为 reject 的理由，变成后续方法的参数。这一点与 Promise.resolve 方法不一致。
```
Promise.resolve('SECRET VALUE').then(val => {
    console.log(val);
});
   
Promise.reject(new Error('SECRET VALUE')).then(err => {
    console.log(err.message);
});

// 等价于 var promise = Promise.reject(new Error('SECRET VALUE'));
// var promise = new Promise(function (fulfill, reject) {
//   reject(new Error('SECRET VALUE'));
// });
```
### 五、catch
catch 方法是 then(null, reject) 的别名，用于指定发生错误时的回调函数。如果异步操作抛出错误，状态就会变为 rejected，就会调用catch 方法指定的回调函数，处理这个错误。另外，then 方法指定的回调函数，如果运行中抛出错误，也会被 catch 方法捕获。
```
function alwaysThrows() {
    throw new Error('OH NOES');
}

function iterate(num) {
    console.log(num);
    return num + 1;
}

// 打印 "1, 2, 3, 4, 5, OH NOES"
Promise.resolve(iterate(1))
    .then(iterate)
    .then(iterate)
    .then(iterate)
    .then(iterate)
    .then(alwaysThrows)
    .then(iterate)
    .then(iterate)
    .then(iterate)
    .then(iterate)
    .then(iterate)
    .catch(err => {
        console.log(err.message);
    });
```
### 六、all
Promise.all 方法接受一个数组作为参数，数组中都是 Promise 实例，如果不是，就会先调用 Promise.resolve 方法，将参数转为 Promise 实例，再进一步处理；数组 Promise 实例状态都变成 fulfilled，返回的 Promise 的状态才会变成 fulfilled，有一个被 rejected，返回的就是 rejected；注意如果作为参数的 Promise 实例，自己定义了 catch 方法，那么它一旦被 rejected，并不会触发 Promise.all() 的 catch 方法。
```
const promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
   
// 自定义了 catch 方法
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
})
.then(result => result)
.catch(e => e);

const p2 = new Promise((resolve, reject) => {
  throw new Error('报错了');
})
.then(result => result)
.catch(e => e);

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// ["hello", Error: 报错了]
   
// 精简版 Promise.all 实现
function all(a, b) {
    return new Promise(function (fulfill, reject) {
        var counter = 0;
        var out = [];

        a.then(function (val) {
            out[0] = val;
            counter++;

            if (counter >= 2) {
                fulfill(out);
            }
        });

        b.then(function (val) {
            out[1] = val;
            counter++;

            if (counter >= 2) {
                fulfill(out);
            }
        });
    });
}

all(getPromise1(), getPromise2())
    .then(console.log);
```
### 七、race
```
// 如果 5 秒之内 fetch 方法无法返回结果，变量 p 的状态就会变为 rejected，从而触发 catch 方法指定的回调函数
const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
]);
p.then(response => console.log(response));
p.catch(error => console.log(error));
```
Promise.race 方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例，参数与 Promise.all 方法一样，如果不是 Promise 实例，就会先调用 Promise.resolve 方法，将参数转为 Promise 实例，再进一步处理。只要数组参数中有一个实例率先改变状态，返回 promise 对象的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给返回 promise 对象的回调函数。