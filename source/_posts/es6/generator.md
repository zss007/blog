---
title: es6 之 Generator
categories:
- es6
---
Generator 函数是 ES6 提供的一种异步编程解决方案，语法行为与传统函数完全不同。Generator 函数有多种理解角度，首先可以理解成 Generator 函数是一个状态机，封装了多个内部状态。执行 Generator 函数会返回一个遍历器对象，也就是说，Generator 函数除了状态机，还是一个遍历器对象生成函数。返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。它有两个特征。一是，function 关键字与函数名之间有一个星号；二是，函数体内部使用 yield 表达式，定义不同的内部状态（yield 在英语里的意思就是 "产出"）。
遍历器对象的 next 方法的运行逻辑：1、遇到 yield 表达式就暂停执行后面的操作，并将紧跟在 yield 后面的那个表达式的值，作为返回的对象的 value 属性值；2、下一次调用 next 方法时，再继续往下执行，直到遇到下一个 yield 表达式；3、如果没有再遇到新的 yield 表达式，就一直运行到函数结束，直到 return 语句为止，并将 return 语句后面的表达式的值作为返回的对象的 value 属性值；4、如果该函数没有 return 语句，则返回的对象的 value 属性值为 undefined。
<!--more-->
### 一、简介
```
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}
var hw = helloWorldGenerator(); // 调用后并不执行，返回的也不是函数运行结果，而是遍历器对象（Iterator Object）
// Generator 函数是分段执行的，yield 表达式是暂停执行的标记，而 next 方法可以恢复执行
hw.next() // { value: 'hello', done: false }
hw.next() // { value: 'world', done: false }
hw.next() // { value: 'ending', done: true }，value 属性就是在 return 后面的表达式的值，done 属性的值 true 表示遍历已经结束
hw.next() // { value: undefined, done: true }，函数已经运行完毕，以后再调用 next 方法返回的都是这个值
  
// 没有规定 function 关键字与函数名之间的星号写在哪个位置，下面的写法都能通过
function * foo(x, y) { ··· }
function *foo(x, y) { ··· }
function* foo(x, y) { ··· }
function*foo(x, y) { ··· }
  
// Generator 函数可以不用 yield 表达式，这时就变成了一个单纯的暂缓执行函数
function* f() {
  console.log('执行了！')
}
var generator = f();
setTimeout(function () {
  generator.next()  // 调用 next 方法时，函数 f 才会执行
}, 2000);
  
// yield 表达式只能用在 Generator 函数里面，用在其他地方都会报错
(function (){
  yield 1;  // SyntaxError: Unexpected number
})()
  
// forEach 方法的参数是一个普通函数，但是在里面使用了 yield 表达式，可改用 for 循环
var arr = [1, [[2, 3], 4], [5, 6]];
var flat = function* (a) {
  a.forEach(function (item) {
    if (typeof item !== 'number') {
      yield* flat(item);  // 产生句法错误
    } else {
      yield item;
    }
  });
};
for (var f of flat(arr)){
  console.log(f);
}
  
// yield 表达式如果用在另一个表达式之中，必须放在圆括号里面
function* demo() {
  console.log('Hello' + yield); // SyntaxError
  console.log('Hello' + yield 123); // SyntaxError
  console.log('Hello' + (yield)); // OK
  console.log('Hello' + (yield 123)); // OK
}
  
// yield 表达式用作函数参数或放在赋值表达式的右边，可以不加括号
function* demo() {
  foo(yield 'a', yield 'b'); // OK
  let input = yield; // OK
}
  
// 可以把 Generator 赋值给对象的 Symbol.iterator 属性，从而使得该对象具有 Iterator 接口
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};
[...myIterable] // [1, 2, 3]
  
// Generator 函数执行后，返回一个遍历器对象。该对象本身也具有 Symbol.iterator 属性，执行后返回自身
function* gen(){
  // some code
}
var g = gen();
g[Symbol.iterator]() === g  // true
```
### 二、next 方法的参数
yield 表达式本身没有返回值，或者说总是返回 undefined。next 方法可以带一个参数，该参数就会被当作上一个 yield 表达式的返回值。
```
function* f() {
  for(var i = 0; true; i++) {
    var reset = yield i;
    if(reset) { i = -1; }
  }
}
var g = f();
g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false }，当 next 方法带一个参数 true，reset 被重置，因此 i 会等于 -1，下一轮循环就会从 -1 开始递增
  
// 通过 next 方法的参数，就有办法在 Generator 函数开始运行之后，继续向函数体内部注入值，从而调整函数行为（第一次使用 next 方法时传递参数是无效的）
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}
var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}，next 方法不带参数，导致 y 的值等于 2 * undefined（即 NaN），除以 3 以后还是 NaN
a.next() // Object{value:NaN, done:true}，不带参数所以 z 等于 undefined，返回对象的 value 属性等于 5 + NaN + undefined，即 NaN
var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false }，将上一次 yield 表达式的值设为 12，因此 y 等于 24，返回 y / 3 的值 8
b.next(13) // { value:42, done:true }，将上一次 yield表 达式的值设为 13，因此 z 等于13，这时 x 等于 5，y 等于 24，所以 return 语句的值等于 42
  
// 用 wrapper 将 Generator 函数先包一层，使得第一次调用 next 方法就输入参数
function wrapper(generatorFunction) {
  return function (...args) {
    let generatorObject = generatorFunction(...args);
    generatorObject.next();
    return generatorObject;
  };
}
const wrapped = wrapper(function* () {
  console.log(`First input: ${yield}`);
  return 'DONE';
});
wrapped().next('hello!')  // First input: hello!
```
### 三、for...of 循环
for...of 循环可以自动遍历 Generator 函数时生成的 Iterator 对象，且此时不再需要调用 next 方法。
```
// 一旦 next 方法的返回对象的 done 属性为 true，for...of 循环就会中止且不包含该返回对象
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6; // return 语句返回的 6 不包括在 for...of 循环之中
}
for (let v of foo()) {
  console.log(v); // 1 2 3 4 5
}
  
// 利用 Generator 函数和 for...of 循环实现斐波那契数列（for...of 语句不需要使用 next 方法）
function* fibonacci() {
  let [prev, curr] = [0, 1];
  for (;;) {
    [prev, curr] = [curr, prev + curr];
    yield curr;
  }
}
for (let n of fibonacci()) {
  if (n > 1000) break;
  console.log(n);
}
  
// 为原生的 JavaScript 对象添加遍历接口
function* objectEntries(obj) {
  let propKeys = Reflect.ownKeys(obj);

  for (let propKey of propKeys) {
    yield [propKey, obj[propKey]];
  }
}
let jane = { first: 'Jane', last: 'Doe' };
for (let [key, value] of objectEntries(jane)) {
  console.log(`${key}: ${value}`);
}
// first: Jane
// last: Doe
  
// 加上遍历器接口的另一种写法是将 Generator 函数加到对象的 Symbol.iterator 属性上面
jane[Symbol.iterator] = objectEntries;
  
// 可以将 Generator 函数返回的 Iterator 对象作为参数传递给扩展运算符、Array.from、解构赋值、for...of 循环
function* numbers () {
  yield 1
  yield 2
  return 3
  yield 4
}
[...numbers()] // [1, 2]，扩展运算符
Array.from(numbers()) // [1, 2]，Array.from 方法
let [x, y] = numbers(); // x: 1， y: 2，解构赋值
for (let n of numbers()) { // for...of 循环
  console.log(n)  // 1 2
}
```
### 四、Generator.prototype.throw()
Generator 函数返回的遍历器对象，都有一个 throw 方法，可以在函数体外抛出错误，然后在 Generator 函数体内捕获。
```
var g = function* () {
  try {
    yield;
  } catch (e) {
    console.log('内部捕获', e);
  }
};
var i = g();
i.next();
try {
  i.throw('a'); // 建议抛出 Error 对象的实例
  i.throw('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 内部捕获 a（第一个错误被 Generator 函数体内的 catch 语句捕获）
// 外部捕获 b（第二次抛出错误，由于 Generator 函数内部的 catch 语句已经执行过了，不会再捕捉到这个错误了，所以这个错误就被抛出了 Generator 函数体，被函数体外的 catch 语句捕获）
  
// 全局的 throw 命令抛出的只能被函数体外的 catch 语句捕获（不要混淆遍历器对象的 throw 方法和全局的 throw 命令）
var g = function* () {
  while (true) {
    try {
      yield;
    } catch (e) {
      if (e != 'a') throw e;
      console.log('内部捕获', e);
    }
  }
};
var i = g();
i.next();
try {
  throw new Error('a');
  throw new Error('b');
} catch (e) {
  console.log('外部捕获', e); // 外部捕获 [Error: a]，函数体外的 catch 语句块捕获了抛出的 a 错误以后，就不会再继续 try 代码块里面剩余的语句了
}
  
// 如果 Generator 函数内部没有部署 try...catch 代码块，那么 throw 方法抛出的错误将被外部 try...catch 代码块捕获
var g = function* () {
  while (true) {
    yield;
    console.log('内部捕获', e);
  }
};
var i = g();
i.next();
try {
  i.throw('a');
  i.throw('b');
} catch (e) {
  console.log('外部捕获', e); // 外部捕获 a
}
  
// 如果 Generator 函数内部和外部都没有部署 try...catch 代码块，那么程序将报错，直接中断执行
var gen = function* gen(){
  yield console.log('hello');
  yield console.log('world');
}

var g = gen();
g.next(); // hello
g.throw();  // Uncaught undefined
  
// throw 方法被捕获以后，会附带执行下一条 yield 表达式。也就是说，会附带执行一次 next 方法
var gen = function* gen(){
  try {
    yield console.log('a');
  } catch (e) {
    // ...
  }
  yield console.log('b');
  yield console.log('c');
}
var g = gen();
g.next() // a
g.throw() // b，只要 Generator 函数内部部署了 try...catch 代码块，那么遍历器的 throw 方法抛出的错误不影响下一次遍历
g.next() // c
  
// throw 命令与 g.throw 方法是无关的，两者互不影响
// 函数体内捕获错误的机制，大大方便了对错误的处理。多个 yield 表达式，可以只用一个 try...catch 代码块来捕获错误。如果使用回调函数的写法，
// 想要捕获多个错误，就不得不为每个函数内部写一个错误处理语句，现在只在 Generator 函数内部写一次 catch 语句就可以了。
var gen = function* gen(){
  yield console.log('hello');
  yield console.log('world');
}
var g = gen();
g.next();
try {
  throw new Error();
} catch (e) {
  g.next();
}
// hello
// world
  
// Generator 函数体外抛出的错误，可以在函数体内捕获；反过来，Generator 函数体内抛出的错误，也可以被函数体外的 catch 捕获
function* foo() {
  var x = yield 3;
  var y = x.toUpperCase();
  yield y;
}
var it = foo();
it.next(); // { value:3, done:false }
try {
  it.next(42);  // 数值是没有 toUpperCase 方法的，所以会抛出一个 TypeError 错误，被函数体外的 catch 捕获
} catch (err) {
  console.log(err);
}
  
// Generator 执行过程中抛出错误，且没有被内部捕获，就不会再执行下去了
function* g() {
  yield 1;
  console.log('throwing an exception');
  throw new Error('generator broke!');
  yield 2;
  yield 3;
}
function log(generator) {
  var v;
  console.log('starting generator');
  try {
    v = generator.next();
    console.log('第一次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  try {
    v = generator.next();
    console.log('第二次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  try {
    v = generator.next();
    console.log('第三次运行next方法', v);
  } catch (err) {
    console.log('捕捉错误', v);
  }
  console.log('caller done');
}
log(g());
// starting generator
// 第一次运行 next 方法 { value: 1, done: false }
// throwing an exception
// 捕捉错误 { value: 1, done: false }，抛出错误且没有被内部捕获，停止执行，还是变量 v 值还是之前的
// 第三次运行next方法 { value: undefined, done: true }，此后还调用 next 方法，将返回一个 value 属性等于 undefined、done 属性等于 true 的对象
// caller done，如果将 throw 用 try...catch 包起来，那么 next 的 value 属性依次为 1、2、3
```
### 五、Generator.prototype.return()
Generator 函数返回的遍历器对象，还有一个 return 方法，可以返回给定的值，并且终结遍历 Generator 函数。
```
// 调用 return 方法后，返回值的 value 属性就是 return 方法的参数 foo，并且 Generator 函数的遍历就终止了，返回值的done属性为true
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}
var g = gen();
g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }，以后再调用 next 方法，done 属性总是返回 true
  
// 如果 return 方法调用时，不提供参数，则返回值的 value 属性为 undefined
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}
var g = gen();
g.next()        // { value: 1, done: false }
g.return() // { value: undefined, done: true }
  
// 如果 Generator 函数内部有 try...finally 代码块，那么 return 方法会推迟到 finally 代码块执行完再执行
function* numbers () {
  yield 1;
  try {
    yield 2;
    yield 3;
  } finally {
    yield 4;
    yield 5;
  }
  yield 6;
}
var g = numbers();
g.next() // { value: 1, done: false }
g.next() // { value: 2, done: false }
g.return(7) // { value: 4, done: false }
g.next() // { value: 5, done: false }
g.next() // { value: 7, done: true }
```
### 六、next()、throw()、return() 的共同点
next()、throw()、return() 这三个方法本质上是同一件事，可以放在一起理解。它们的作用都是让 Generator 函数恢复执行，并且使用不同的语句替换 yield 表达式。
```
// next() 是将 yield 表达式替换成一个值
const g = function* (x, y) {
  let result = yield x + y;
  return result;
};
const gen = g(1, 2);
gen.next(); // Object {value: 3, done: false}
gen.next(1); // Object {value: 1, done: true}
// 相当于将 let result = yield x + y
// 替换成 let result = 1;
  
// throw() 是将 yield 表达式替换成一个 throw 语句
gen.throw(new Error('出错了')); // Uncaught Error: 出错了
// 相当于将 let result = yield x + y
// 替换成 let result = throw(new Error('出错了'));
  
// return() 是将 yield 表达式替换成一个 return 语句
gen.return(2); // Object {value: 2, done: true}
// 相当于将 let result = yield x + y
// 替换成 let result = return 2;
```
### 七、yield* 表达式
如果在 Generator 函数内部，调用另一个 Generator 函数，默认情况下是没有效果的。这个时候就需要用到 yield* 表达式，用来在一个 Generator 函数里面执行另一个 Generator 函数。
```
function* foo() {
  yield 'a';
  yield 'b';
}
function* bar() {
  yield 'x';
  foo();  // 在 bar 里面调用 foo，是不会有效果的
  yield 'y';
}
for (let v of bar()){
  console.log(v);
}
// "x"
// "y"
  
// yield* 执行另一个 Generator 函数
function* bar() {
  yield 'x';
  yield* foo();
  yield 'y';
}
// 等同于
function* bar() {
  yield 'x';
  yield 'a';
  yield 'b';
  yield 'y';
}
// 等同于
function* bar() {
  yield 'x';
  for (let v of foo()) {
    yield v;
  }
  yield 'y';
}
for (let v of bar()){
  console.log(v);
}
// "x"
// "a"
// "b"
// "y"
  
// 如果 yield 表达式后面跟的是一个遍历器对象，需要在 yield 表达式后面加上星号，表明它返回的是一个遍历器对象，这被称为 yield* 表达式
function* inner() {
  yield 'hello!';
}
function* outer1() {
  yield 'open';
  yield inner();
  yield 'close';
}
var gen = outer1()
gen.next().value // "open"
gen.next().value // 返回一个遍历器对象
gen.next().value // "close"
function* outer2() {
  yield 'open'
  yield* inner()
  yield 'close'
}
var gen = outer2()
gen.next().value // "open"
gen.next().value // "hello!"
gen.next().value // "close"
  
// yield* 后面的 Generator 函数没有 return 语句时，等同于在 Generator 函数内部部署一个 for...of 循环
function* concat(iter1, iter2) {
  yield* iter1;
  yield* iter2;
}
// 等同于
function* concat(iter1, iter2) {
  for (var value of iter1) {
    yield value;
  }
  for (var value of iter2) {
    yield value;
  }
}
// 在有 return 语句时，则需要用 yield* iterator 的形式获取 return 语句的值
var value = yield* iterator;
  
// 如果 yield* 后面跟着一个数组，由于数组原生支持遍历器，因此就会遍历数组成员
function* gen(){
  yield* ["a", "b", "c"]; // 如果不加星号，返回的是整个数组
}
gen().next() // { value:"a", done:false }
  
// 任何数据结构只要有 Iterator 接口，就可以被 yield* 遍历
let read = (function* () {
  yield 'hello';
  yield* 'hello';
})();
read.next().value // "hello"，yield 表达式返回整个字符串
read.next().value // "h"，yield* 语句返回单个字符
  
// 如果被代理的 Generator 函数有 return 语句，那么就可以向代理它的 Generator 函数返回数据
function* foo() {
  yield 2;
  yield 3;
  return "foo";
}
function* bar() {
  yield 1;
  var v = yield* foo();
  console.log("v: " + v);
  yield 4;
}
var it = bar();
it.next() // {value: 1, done: false}
it.next() // {value: 2, done: false}
it.next() // {value: 3, done: false}
it.next() // "v: foo"，{value: 4, done: false}，屏幕上会有输出是因为函数 foo 的 return 语句，向函数 bar 提供了返回值
it.next() // {value: undefined, done: true}
  
// 存在两次遍历，第一次是扩展运算符遍历函数 logReturned 返回的遍历器对象，第二次是 yield* 语句遍历函数 genFuncWithReturn 返回的遍历器对象
function* genFuncWithReturn() {
  yield 'a';
  yield 'b';
  return 'The result';
}
function* logReturned(genObj) {
  let result = yield* genObj;
  console.log(result);
}
[...logReturned(genFuncWithReturn())]
// The result
// 值为 [ 'a', 'b' ]
  
// yield* 命令可以很方便地取出嵌套数组的所有成员
function* iterTree(tree) {
  if (Array.isArray(tree)) {
    for(let i=0; i < tree.length; i++) {
      yield* iterTree(tree[i]);
    }
  } else {
    yield tree;
  }
}
const tree = [ 'a', ['b', 'c'], ['d', 'e'] ];
for(let x of iterTree(tree)) {
  console.log(x); // a, b, c, d, e
}
  
// 使用 yield* 语句遍历完全二叉树
// 下面是二叉树的构造函数，
// 三个参数分别是左树、当前节点和右树
function Tree(left, label, right) {
  this.left = left;
  this.label = label;
  this.right = right;
}
// 下面是中序（inorder）遍历函数。
// 由于返回的是一个遍历器，所以要用generator函数。
// 函数体内采用递归算法，所以左树和右树要用yield*遍历
function* inorder(t) {
  if (t) {
    yield* inorder(t.left);
    yield t.label;
    yield* inorder(t.right);
  }
}
// 下面生成二叉树
function make(array) {
  // 判断是否为叶节点
  if (array.length == 1) return new Tree(null, array[0], null);
  return new Tree(make(array[0]), array[1], make(array[2]));
}
let tree = make([[['a'], 'b', ['c']], 'd', [['e'], 'f', ['g']]]);
// 遍历二叉树
var result = [];
for (let node of inorder(tree)) {
  result.push(node);
}
result
// ['a', 'b', 'c', 'd', 'e', 'f', 'g']
```
### 八、作为对象属性的 Generator 函数
```
// 对象的属性是 Generator 函数可以简写成下面的形式
let obj = {
  * myGeneratorMethod() {
    // ···
  }
};
  
// 完整形式如下，与上面的写法是等价的
let obj = {
  myGeneratorMethod: function* () {
    // ···
  }
};
```
### 九、Generator 函数的 this
```
// Generator 函数 g 返回的遍历器 obj，是 g 的实例，而且继承了 g.prototype
function* g() {}
g.prototype.hello = function () {
  return 'hi!';
};
let obj = g();
obj instanceof g // true
obj.hello() // 'hi!'
  
// 如果把 g 当作普通的构造函数，并不会生效，因为 g 返回的总是遍历器对象，而不是 this 对象
function* g() {
  this.a = 11;
}
let obj = g();
obj.next();
obj.a // undefined
  
// Generator 函数也不能跟 new 命令一起用，会报错
function* F() {
  yield this.x = 2;
  yield this.y = 3;
}
new F() // TypeError: F is not a constructor，因为 F 不是构造函数
  
// 使用 call 方法绑定 Generator 函数内部的 this
function* F() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}
var obj = {};
var f = F.call(obj);
f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}
obj.a // 1
obj.b // 2
obj.c // 3
  
// 执行的是遍历器对象 f 和生成的对象实例是 obj 统一
function* F() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}
var f = F.call(F.prototype);
f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}
f.a // 1
f.b // 2
f.c // 3
  
// 再将 F 改成构造函数，就可以对它执行 new 命令了
function* gen() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}
function F() {
  return gen.call(gen.prototype);
}
var f = new F();
f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}
f.a // 1
f.b // 2
f.c // 3
```
Generator 函数总是返回一个遍历器，ES6 规定这个遍历器是 Generator 函数的实例，也继承了 Generator 函数的 prototype 对象上的方法。