---
title: es6 之 async
categories:
- es6
---
ES2017 标准引入了 async 函数，使得异步操作变得更加方便。async 函数是什么？一句话，它就是 Generator 函数的语法糖。
<!--more-->
### 一、简介
```
const fs = require('fs');

const readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};
  
// Generator 函数依次读取两个文件
const gen = function* () {
  const f1 = yield readFile('/etc/fstab');
  const f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
  
// async 函数就是将 Generator 函数的星号 * 替换成 async，将 yield 替换成await
const asyncReadFile = async function () {
  const f1 = await readFile('/etc/fstab');
  const f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```
async函数对 Generator 函数的改进，体现在以下四点：   
1、Generator 函数的执行必须靠执行器，所以才有了 co 模块，而 async 函数自带执行器。也就是说，async 函数的执行，与普通函数一模一样，只要一行：`asyncReadFile()`，然后它就会自动执行输出最后结果。这完全不像 Generator 函数，需要调用 next 方法，或者用 co 模块，才能真正执行得到最后结果；    
2、async 和 await 比起星号和 yield 语义更清楚了。async 表示函数里有异步操作，await 表示紧跟在后面的表达式需要等待结果；    
3、co 模块约定，yield 命令后面只能是 Thunk 函数或 Promise 对象，而 async 函数的 await 命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。    
4、async 函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。async 函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而 await 命令就是内部 then 命令的语法糖。   
### 二、基本用法
async 函数返回一个 Promise 对象，可以使用 then 方法添加回调函数。当函数执行的时候，一旦遇到 await 就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。
```
// 函数前面的 async 关键字，表明该函数内部有异步操作。调用该函数时，会立即返回一个 Promise 对象
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}
getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
  
// 指定 50 毫秒以后，输出 hello world
function timeout(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}
async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}
asyncPrint('hello world', 50);
  
// async 函数返回的是 Promise 对象，可以作为 await 命令的参数
async function timeout(ms) {
  await new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}
async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}
asyncPrint('hello world', 50);
  
// async 函数有多种使用形式
async function foo() {} // 函数声明
const foo = async function () {}; // 函数表达式
let obj = { async foo() {} }; // 对象的方法
obj.foo().then(...)
class Storage { // Class 的方法
  constructor() {
    this.cachePromise = caches.open('avatars');
  }
  async getAvatar(name) {
    const cache = await this.cachePromise;
    return cache.match(`/avatars/${name}.jpg`);
  }
}
const storage = new Storage();
storage.getAvatar('jake').then(…);
const foo = async () => {}; // 箭头函数，注意箭头函数不能用作 Generator 函数
```
### 三、语法
async 函数的语法规则总体上比较简单，难点是错误处理机制。
```
// async 函数返回一个 Promise 对象，内部 return 语句返回的值，会成为 then 方法回调函数的参数
async function f() {
  return 'hello world';
}
f().then(v => console.log(v)) // "hello world"
  
// async 函数内部抛出错误，会导致返回的 Promise 对象变为 reject 状态，抛出的错误对象会被 catch 方法回调函数接收到
async function f() {
  throw new Error('出错了');
}
f().then(
  v => console.log(v),
  e => console.log(e) // Error: 出错了
)
  
// async 函数返回的 Promise 对象，必须等到内部所有 await 命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到 return 语句或者抛出错误
async function getTitle(url) {
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
getTitle('https://tc39.github.io/ecma262/').then(console.log) // 只有 async 函数内部的异步操作执行完，才会执行 then 方法指定的回调函数
// "ECMAScript 2017 Language Specification"
  
// await 命令后面是一个 Promise 对象，如果不是会被转成一个立即 resolve 的 Promise 对象
async function f() {
  return await 123;
}
f().then(v => console.log(v)) // 123
  
// await 命令后面的 Promise 对象如果变为 reject 状态，则 reject 的参数会被 catch 方法的回调函数接收到
async function f() {
  await Promise.reject('出错了');  // await 语句前面没有 return，但是 reject 方法的参数依然传入了 catch 方法的回调函数，如果在 await 前面加上 return 效果是一样的
}
f()
.then(v => console.log(v))
.catch(e => console.log(e)) // 出错了
  
// 只要一个 await 语句后面的 Promise 变为 reject，那么整个 async 函数都会中断执行
async function f() {
  await Promise.reject('出错了');
  await Promise.resolve('hello world'); // 不会执行
}
  
// 将 await 放在 try...catch 结构里面，这样不管这个异步操作是否成功，后面的 await 都会执行
async function f() {
  try {
    await Promise.reject('出错了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}
f()
.then(v => console.log(v))  // hello world
  
// 或者 await 后面的 Promise 对象再跟一个 catch 方法处理前面可能出现的错误
async function f() {
  await Promise.reject('出错了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}
f()
.then(v => console.log(v))
// 出错了
// hello world
  
// 如果 await 后面的异步操作出错，那么等同于 async 函数返回的 Promise 对象被 reject
async function f() {
  await new Promise(function (resolve, reject) {
    throw new Error('出错了');
  });
}
f()
.then(v => console.log(v))
.catch(e => console.log(e)) // Error：出错了
  
// 防止出错的方法是将 await 放在 try...catch 代码块之中
async function f() {
  try {
    await new Promise(function (resolve, reject) {
      throw new Error('出错了');
    });
  } catch(e) {
  }
  return await('hello world');
}
  
// 有多个 await 命令，可以统一放在 try...catch 结构中
async function main() {
  try {
    const val1 = await firstStep();
    const val2 = await secondStep(val1);
    const val3 = await thirdStep(val1, val2);
    console.log('Final: ', val3);
  }
  catch (err) {
    console.error(err);
  }
}
  
// 使用 try...catch 结构实现多次重复尝试（如果 await 操作成功，就会使用 break 语句退出循环；如果失败会被 catch 语句捕捉，然后进入下一轮循环）
const superagent = require('superagent');
const NUM_RETRIES = 3;
async function test() {
  let i;
  for (i = 0; i < NUM_RETRIES; ++i) {
    try {
      await superagent.get('http://google.com/this-throws-an-error');
      break;
    } catch(err) {}
  }
  console.log(i); // 3
}
test();
  
// 多个 await 命令后面的异步操作，如果不存在继发关系最好让它们同时触发，这样比较耗时
let foo = await getFoo();
let bar = await getBar();
  
// 可以让独立的异步操作同时触发，缩短程序的执行时间
let [foo, bar] = await Promise.all([getFoo(), getBar()]); // 写法一
let fooPromise = getFoo();  // 写法二
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
  
// await 命令只能用在 async 函数之中，如果用在普通函数会报错
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  docs.forEach(function (doc) { // 报错
    await db.post(doc);
  });
}
  
// 可能不会正常工作，原因是这时三个 db.post 操作将是并发执行，也就是同时执行，而不是继发执行
function dbFuc(db) { //这里不需要 async
  let docs = [{}, {}, {}];
  docs.forEach(async function (doc) { // 可能得到错误结果
    await db.post(doc);
  });
}
  
// 继发执行正确的写法是采用 for 循环
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  for (let doc of docs) {
    await db.post(doc);
  }
}
  
// 希望多个请求并发执行，可以使用 Promise.all 方法。当三个请求都会 resolved 时，下面两种写法效果相同
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));
  let results = await Promise.all(promises);
  console.log(results);
}
async function dbFuc(db) {  // 或者使用下面的写法
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));
  let results = [];
  for (let promise of promises) {
    results.push(await promise);
  }
  console.log(results);
}
```
### 四、async 函数的实现原理
async 函数的实现原理，就是将 Generator 函数和自动执行器，包装在一个函数里。
```
// 所有的 async 函数都可以写成下面的第二种形式，其中的 spawn 函数就是自动执行器
async function fn(args) {
  // ...
}
function fn(args) { // 等同于
  return spawn(function* () {
    // ...
  });
}
  
// spawn 函数的实现，基本就是上文 Generator 自动执行器的翻版
function spawn(genF) {
  return new Promise(function(resolve, reject) {
    const gen = genF();
    function step(nextF) {
      let next;
      try {
        next = nextF();
      } catch(e) {
        return reject(e);
      }
      if(next.done) {
        return resolve(next.value);
      }
      Promise.resolve(next.value).then(function(v) {
        step(function() { return gen.next(v); });
      }, function(e) {
        step(function() { return gen.throw(e); });
      });
    }
    step(function() { return gen.next(undefined); });
  });
}
```
### 五、与其他异步处理方法的比较
通过一个例子，来看 async 函数与 Promise、Generator 函数的比较：假定某个 DOM 元素上面，部署了一系列的动画，前一个动画结束，才能开始后一个。如果当中有一个动画出错，就不再往下执行，返回上一个成功执行的动画的返回值。
```
// Promise 的写法：比回调函数的写法大大改进，但是代码完全都是 Promise 的 API（then、catch等等），操作本身的语义反而不容易看出来
function chainAnimationsPromise(elem, animations) {
  let ret = null; // 变量 ret 用来保存上一个动画的返回值
  let p = Promise.resolve();  // 新建一个空的 Promise
  for(let anim of animations) { // 使用 then 方法，添加所有动画
    p = p.then(function(val) {
      ret = val;
      return anim(elem);
    });
  }
  return p.catch(function(e) {  // 返回一个部署了错误捕捉机制的 Promise
    /* 忽略错误，继续执行 */
  }).then(function() {
    return ret;
  });
}
  
// Generator 函数的写法：语义比 Promise 写法更清晰，问题在于必须有一个任务运行器自动执行 Generator 函数且 yield 语句后面的表达式必须返回一个 Promise
function chainAnimationsGenerator(elem, animations) {
  return spawn(function*() {
    let ret = null;
    try {
      for(let anim of animations) {
        ret = yield anim(elem);
      }
    } catch(e) {
      /* 忽略错误，继续执行 */
    }
    return ret;
  });
}
  
// async 函数的写法：实现最简洁，最符合语义，几乎没有语义不相关的代码，Generator 写法中的自动执行器改在语言层面提供，不暴露给用户，代码量最少
async function chainAnimationsAsync(elem, animations) {
  let ret = null;
  try {
    for(let anim of animations) {
      ret = await anim(elem);
    }
  } catch(e) {
    /* 忽略错误，继续执行 */
  }
  return ret;
}
```
### 六、实例：按顺序完成异步操作
实际开发中，经常遇到一组异步操作需要按照顺序完成。比如，依次远程读取一组 URL，然后按照读取的顺序输出结果。
```
// Promise 的写法：写法不太直观，可读性比较差
function logInOrder(urls) {
  const textPromises = urls.map(url => {  // 远程读取所有URL
    return fetch(url).then(response => response.text());
  });
  textPromises.reduce((chain, textPromise) => { // 依次处理每个 Promise 对象，然后使用 then 将所有 Promise 对象连起来，因此就可以依次输出结果
    return chain.then(() => textPromise)
      .then(text => console.log(text));
  }, Promise.resolve());
}
  
// async 函数实现：所有远程操作都是继发，效率很差，非常浪费时间
async function logInOrder(urls) {
  for (const url of urls) {
    const response = await fetch(url);
    console.log(await response.text());
  }
}
  
// async 函数并发实现：只有 async 函数内部是继发执行，外部不受影响。后面的 for..of 循环内部使用了 await，因此实现了按顺序输出
async function logInOrder(urls) {
  const textPromises = urls.map(async url => {  // 并发读取远程URL
    const response = await fetch(url);
    return response.text();
  });
  for (const textPromise of textPromises) { // 按次序输出
    console.log(await textPromise);
  }
}
```
### 七、异步遍历器
Iterator 接口是一种数据遍历的协议，只要调用遍历器对象的 next 方法，就会得到一个对象，表示当前遍历指针所在的那个位置的信息。next方法返回的对象的结构是 {value, done}，其中 value 表示当前的数据的值，done 是一个布尔值，表示遍历是否结束。这里隐含着一个规定，next 方法必须是同步的，只要调用就必须立刻返回值。也就是说，一旦执行 next 方法，就必须同步地得到 value 和 done 这两个属性。如果遍历指针正好指向同步操作，当然没有问题，但对于异步操作，就不太合适了。目前的解决方法是，Generator 函数里面的异步操作，返回一个 Thunk 函数或者 Promise 对象，即 value 属性是一个 Thunk 函数或者 Promise 对象，等待以后返回真正的值，而 done 属性则还是同步产生的。ES2018 引入了 "异步遍历器"（Async Iterator），为异步操作提供原生的遍历器接口，即 value 和 done 这两个属性都是异步产生。