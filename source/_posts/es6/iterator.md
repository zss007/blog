---
title: es6 之 Iterator
categories:
- es6
---
JavaScript 有四种表示 "集合" 数据集合：Array、Object、Map 以及 Set，用户可以组合使用它们，定义自己的数据结构，比如数组的成员是Map，Map 的成员是对象。这样就需要一种统一的接口机制，来处理所有不同的数据结构。遍历器（Iterator）就是这样一种机制，它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署 Iterator 接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。 
<!--more-->
### 一、简介
Iterator 的作用有三个：一是为各种数据结构，提供一个统一的、简便的访问接口；二是使得数据结构的成员能够按某种次序排列；三是 ES6 创造了一种新的遍历命令 for...of 循环，Iterator 接口主要供 for...of 消费。
Iterator 的遍历过程：创建一个指针对象，指向当前数据结构的起始位置；第一次调用指针对象的 next 方法，可以将指针指向数据结构的第一个成员；第二次调用指针对象的 next 方法，指针就指向数据结构的第二个成员；不断调用指针对象的 next 方法，直到它指向数据结构的结束位置。每一次调用 next 方法，都会返回数据结构的当前成员的信息。具体来说，就是返回一个包含 value 和 done 两个属性的对象。其中，value 属性是当前成员的值，done 属性是一个布尔值，表示遍历是否结束。
```
// 调用指针对象的 next 方法，就可以遍历事先给定的数据结构
var it = makeIterator(['a', 'b']);
it.next() // { value: "a", done: false }
it.next() // { value: "b", done: false }
it.next() // { value: undefined, done: true }
function makeIterator(array) {
  var nextIndex = 0;
  return {
    next: function() {  // 对于遍历器对象来说，done: false 和 value: undefined 属性都是可以省略的
      return nextIndex < array.length ?
        {value: array[nextIndex++]} :
        {done: true};
    }
  };
}
  
// 遍历器与它所遍历的那个数据结构，实际上是分开的，可以写出没有对应数据结构的遍历器对象
var it = idMaker();
it.next().value // 0
it.next().value // 1
it.next().value // 2
function idMaker() {
  var index = 0;

  return {
    next: function() {
      return {value: index++, done: false};
    }
  };
}
```
### 二、默认 Iterator 接口
ES6 规定，默认的 Iterator 接口部署在数据结构的 Symbol.iterator 属性，或者说，一个数据结构只要具有 Symbol.iterator 属性，就可以认为是 "可遍历的"（iterable）。Symbol.iterator 属性本身是一个函数，就是当前数据结构默认的遍历器生成函数。执行这个函数，就会返回一个遍历器。至于属性名 Symbol.iterator，它是一个表达式，返回 Symbol 对象的 iterator 属性，这是一个预定义好的、类型为 Symbol 的特殊值，所以要放在方括号内。
```
// 部署 Symbol.iterator 属性
const obj = {
  [Symbol.iterator] : function () {
    return {
      next: function () {
        return {
          value: 1,
          done: true
        };
      }
    };
  }
};
  
// 存在原生具备遍历器接口的数据结构：Array、Map、Set、String、TypedArray、函数的 arguments 对象、NodeList 对象
let arr = ['a', 'b', 'c'];
let iter = arr[Symbol.iterator]();
iter.next() // { value: 'a', done: false }
iter.next() // { value: 'b', done: false }
iter.next() // { value: 'c', done: false }
iter.next() // { value: undefined, done: true }
  
// 对象需要自己在 Symbol.iterator 属性部署遍历器才会被 for...of 循环遍历（不是很必要，因为 ES6 原生提供了 Map 结构）
class RangeIterator {
  constructor(start, stop) {
    this.value = start;
    this.stop = stop;
  }
  [Symbol.iterator]() { return this; }
  next() {
    var value = this.value;
    if (value < this.stop) {
      this.value++;
      return {done: false, value: value};
    }
    return {done: true, value: undefined};
  }
}
function range(start, stop) {
  return new RangeIterator(start, stop);
}
for (var value of range(0, 3)) {
  console.log(value); // 0, 1, 2
}
  
// 遍历器实现指针结构
function Obj(value) {
  this.value = value;
  this.next = null;
}
Obj.prototype[Symbol.iterator] = function() {
  var iterator = { next: next };
  var current = this;
  function next() {
    if (current) {
      var value = current.value;
      current = current.next;
      return { done: false, value: value };
    } else {
      return { done: true };
    }
  }
  return iterator;
}
var one = new Obj(1);
var two = new Obj(2);
var three = new Obj(3);
one.next = two;
two.next = three;
for (var i of one){
  console.log(i); // 1, 2, 3
}
  
// 类似数组的对象（存在数值键名和 length 属性），直接引用数组的 Iterator 接口部署
NodeList.prototype[Symbol.iterator] = Array.prototype[Symbol.iterator];
NodeList.prototype[Symbol.iterator] = [][Symbol.iterator];  // 或者
[...document.querySelectorAll('div')] // 可以执行了，NodeList 对象是类似数组的对象，本来就具有遍历接口，遍历接口改写后，没有任何影响
  
// 另一个类似数组的对象调用数组的 Symbol.iterator 方法
let iterable = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
};
for (let item of iterable) {
  console.log(item); // 'a', 'b', 'c'
}
  
// 普通对象部署数组的 Symbol.iterator 方法，并无效果
let iterable = {
  a: 'a',
  b: 'b',
  c: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
};
for (let item of iterable) {
  console.log(item); // undefined, undefined, undefined
}
  
// 如果 Symbol.iterator 方法对应的不是遍历器生成函数（即会返回一个遍历器对象），解释引擎将会报错
var obj = {};
obj[Symbol.iterator] = () => 1;
[...obj] // TypeError: [] is not a function
  
// 有了遍历器接口，数据结构就可以用 for...of 循环遍历，也可以使用 while 循环遍历
var $iterator = ITERABLE[Symbol.iterator]();
var $result = $iterator.next();
while (!$result.done) {
  var x = $result.value;
  // ...
  $result = $iterator.next();
}
```
### 三、调用 Iterator 接口的场合
有一些场合会默认调用 Iterator 接口（即 Symbol.iterator 方法），除了 for...of 循环，还有几个别的场合。
```
// 对数组和 Set 结构进行解构赋值时
let set = new Set().add('a').add('b').add('c');
let [x,y] = set;  // x='a'; y='b'
let [first, ...rest] = set; // first='a'; rest=['b','c'];
  
// 扩展运算符（...）也会调用默认的 Iterator 接口
var str = 'hello';
[...str]  // ['h','e','l','l','o']
let arr = ['b', 'c'];
['a', ...arr, 'd']  // ['a', 'b', 'c', 'd']
let arr = [...iterable];  // 只要某个数据结构部署了 Iterator 接口，就可以对它使用扩展运算符将其转为数组
  
// yield* 后面跟的是一个可遍历的结构
let generator = function* () {
  yield 1;
  yield* [2,3,4];
  yield 5;
};
var iterator = generator();
iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }
  
// 其他场合（for...of、Array.from()、Map(), Set(), WeakMap(), WeakSet()、Promise.all()、Promise.race()）
```
### 四、字符串的 Iterator 接口
字符串是一个类似数组的对象，也原生具有 Iterator 接口。
```
// 调用 Symbol.iterator 方法返回一个遍历器对象，在这个遍历器上可以调用 next 方法，实现对于字符串的遍历
var someString = "hi";
typeof someString[Symbol.iterator]  // "function"
var iterator = someString[Symbol.iterator]();
iterator.next()  // { value: "h", done: false }
iterator.next()  // { value: "i", done: false }
iterator.next()  // { value: undefined, done: true }
  
// 可以覆盖原生的 Symbol.iterator 方法，达到修改遍历器行为的目的
var str = new String("hi");
[...str] // ["h", "i"]
str[Symbol.iterator] = function() {
  return {
    next: function() {
      if (this._first) {
        this._first = false;
        return { value: "bye", done: false };
      } else {
        return { done: true };
      }
    },
    _first: true
  };
};
[...str] // ["bye"]，字符串 str 的 Symbol.iterator 方法被修改了，所以扩展运算符（...）返回的值变成了 bye
str // "hi"，而字符串本身还是 hi
```
### 五、Iterator 接口与 Generator 函数
Symbol.iterator 方法的最简单实现，还是使用 Generator 函数。
```
// Symbol.iterator 方法几乎不用部署任何代码，只要用 yield 命令给出每一步的返回值即可
let myIterable = {
  [Symbol.iterator]: function* () {
    yield 1;
    yield 2;
    yield 3;
  }
}
[...myIterable] // [1, 2, 3]
// 或者采用下面的简洁写法
let obj = {
  * [Symbol.iterator]() {
    yield 'hello';
    yield 'world';
  }
};
for (let x of obj) {
  console.log(x);
}
// "hello"
// "world"
```
### 六、遍历器对象的 return()，throw() 
遍历器对象除了具有 next 方法，还可以具有 return 方法和 throw 方法。next 方法是必须部署的，return 方法和 throw 方法是否部署是可选的。return 方法的使用场合是 for...of 循环提前退出（通常是因为出错，或者有 break 语句或 continue 语句），就会调用 return 方法。如果一个对象在完成遍历前，需要清理或释放资源，就可以部署 return 方法。注意，return 方法必须返回一个对象，这是 Generator 规格决定的。throw 方法主要是配合 Generator 函数使用，一般的遍历器对象用不到这个方法。
```
function readLinesSync(file) {
  return {
    [Symbol.iterator]() {
      return {
        next() {
          return { done: false };
        },
        return() {
          file.close();
          return { done: true };
        }
      };
    },
  };
}
// 情况一：输出文件的第一行以后，就会执行 return 方法，关闭这个文件
for (let line of readLinesSync(fileName)) {
  console.log(line);
  break;
}
// 情况二：输出所有行以后，执行 return 方法，关闭该文件
for (let line of readLinesSync(fileName)) {
  console.log(line);
  continue;
}
// 情况三：执行 return 方法关闭文件之后再抛出错误
for (let line of readLinesSync(fileName)) {
  console.log(line);
  throw new Error();
}
```
### 七、for...of 循环
```
// 数组原生具备 iterator 接口
const arr = ['red', 'green', 'blue'];
for(let v of arr) {
  console.log(v); // red green blue
}
const obj = {};
obj[Symbol.iterator] = arr[Symbol.iterator].bind(arr);
for(let v of obj) {
  console.log(v); // red green blue
}
  
// for...of 循环可以代替数组实例的 forEach 方法
arr.forEach(function (element, index) {
  console.log(element); // red green blue
  console.log(index);   // 0 1 2
});
  
// for...in 循环只能获得对象的键名；for...of 循环允许遍历获得键值（如果要获取数组的索引，可以借助数组实例的 entries 方法和 keys 方法）
var arr = ['a', 'b', 'c', 'd'];
for (let a in arr) {
  console.log(a); // 0 1 2 3
}
for (let a of arr) {
  console.log(a); // a b c d
}
  
// for...of 循环只遍历具有数字索引的属性，跟 for...in 循环不一样
let arr = [3, 5, 7];
arr.foo = 'hello';
for (let i in arr) {
  console.log(i); // "0", "1", "2", "foo"
}
for (let i of arr) {
  console.log(i); //  "3", "5", "7"
}
  
// Set 原生具有 Iterator 接口（遍历的顺序是按照各个成员被添加进数据结构的顺序）
var engines = new Set(["Gecko", "Trident", "Webkit", "Webkit"]);
for (var e of engines) {
  console.log(e);
}
// Gecko
// Trident
// Webkit
  
// Map 原生具有 Iterator 接口（遍历的顺序是按照各个成员被添加进数据结构的顺序）
var es6 = new Map();
es6.set("edition", 6);
es6.set("committee", "TC39");
es6.set("standard", "ECMA-262");
for (var [name, value] of es6) {
  console.log(name + ": " + value);
}
// edition: 6
// committee: TC39
// standard: ECMA-262
  
// entries、keys、values 方法调用后均生成遍历器对象
let arr = ['a', 'b', 'c'];
for (let pair of arr.entries()) {
  console.log(pair);
}
// [0, 'a']
// [1, 'b']
// [2, 'c']
  
// 遍历字符串
let str = "hello";
for (let s of str) {
  console.log(s); // h e l l o
}
  
// 遍历 DOM NodeList 对象
let paras = document.querySelectorAll("p");
for (let p of paras) {
  p.classList.add("test");
}
  
// 遍历 arguments 对象
function printArgs() {
  for (let x of arguments) {
    console.log(x);
  }
}
printArgs('a', 'b');
// 'a'
// 'b'
  
// for...of 循环可正确识别 32 位 UTF-16 字符
for (let x of 'a\uD83D\uDC0A') {
  console.log(x);
}
// 'a'
// '\uD83D\uDC0A'
  
// 并不是所有类似数组的对象都具有 Iterator 接口，一个简便的解决方法，就是使用 Array.from 方法将其转为数组
let arrayLike = { length: 2, 0: 'a', 1: 'b' };
for (let x of arrayLike) {  // 报错
  console.log(x);
}
for (let x of Array.from(arrayLike)) {  // 正确
  console.log(x);
}
  
// 普通的对象，for...of 结构不能直接使用，会报错，必须部署了 Iterator 接口后才能使用，不过 for...in 循环依然可以用来遍历键名
let es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
};
for (let e in es6) {
  console.log(e);
}
// edition
// committee
// standard
for (let e of es6) {
  console.log(e);
}
// TypeError: es6[Symbol.iterator] is not a function
  
// 可以使用 Object.keys 方法将对象的键名生成一个数组，然后 for...of 遍历这个数组
for (var key of Object.keys(someObject)) {
  console.log(key + ': ' + someObject[key]);
}
  
// 或者使用 Generator 函数将对象重新包装一下
function* entries(obj) {
  for (let key of Object.keys(obj)) {
    yield [key, obj[key]];
  }
}
for (let [key, value] of entries(obj)) {
  console.log(key, '->', value);
}
// a -> 1
// b -> 2
// c -> 3
  
// 遍历方法对比：
// 写法麻烦
for (var index = 0; index < myArray.length; index++) {
  console.log(myArray[index]);
}
// 简洁，但是无法中途跳出 forEach 循环，break 命令或 return 命令都不能奏效
myArray.forEach(function (value) {
  console.log(value);
});
// 简洁，不过数组的键名是数字，还会遍历手动添加的其他键，包括原型链上的键，甚至在某种情况下会以任意顺序遍历键名（主要为遍历对象而设计）
for (var index in myArray) {
  console.log(myArray[index]);
}
// 简洁，没有 for...in 那些缺点，可以与 break、continue 和 return 配合使用，提供了遍历所有数据结构的统一操作接口
for (let value of myArray) {
  console.log(value);
}
  
// break 语句跳出 for...of 循环（如果当前项大于 1000，就会使用 break 语句跳出 for...of 循环）
for (var n of fibonacci) {
  if (n > 1000)
    break;
  console.log(n);
}
```
一个数据结构只要部署了 Symbol.iterator 属性，就被视为具有 iterator 接口，就可以用 for...of 循环遍历它的成员。也就是说，for...of 循环内部调用的是数据结构的 Symbol.iterator 方法。for...of 循环可以使用的范围包括数组、Set 和 Map 结构、某些类似数组的对象（比如 arguments 对象、DOM NodeList 对象）、Generator 对象，以及字符串。