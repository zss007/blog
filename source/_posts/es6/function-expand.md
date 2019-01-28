---
title: es6 之函数扩展
categories:
- es6
---
ES6 扩展了函数。
<!--more-->
### 一、函数参数的默认值
ES6 之前，不能直接为函数的参数指定默认值，只能采用变通的方法。
```
function log(x, y) {
  y = y || 'World';
  console.log(x, y);
}
log('Hello') // Hello World
log('Hello', 'China') // Hello China
// 如果参数 y 赋值了，但是对应的布尔值为 false，则该赋值不起作用。就像参数 y 等于空字符，结果被改为默认值
log('Hello', '') // Hello World
    
// 为了避免这个问题，通常需要先判断一下参数 y 是否被赋值
if (typeof y === 'undefined') {
  y = 'World';
}
    
// ES6 允许为函数的参数设置默认值，即直接写在参数定义的后面
function log(x, y = 'World') {
  console.log(x, y);
}
log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello
    
// 参数变量是默认声明的，所以不能用 let 或 const 再次声明
function foo(x = 5) {
  let x = 1; // error
  const x = 2; // error
}
    
// 使用参数默认值时，函数不能有同名参数
function foo(x, x, y) {
  // 不报错
}
function foo(x, x, y = 1) {
  // 报错。SyntaxError: Duplicate parameter name not allowed in this context
}
    
// 参数默认值不是传值的，而是每次都重新计算默认值表达式的值。也就是说，参数默认值是惰性求值的
let x = 99;
function foo(p = x + 1) {
  console.log(p);
}
foo() // 100
x = 100;
foo() // 101
    
// 参数默认值可以与解构赋值的默认值，结合起来使用
function foo({x, y = 5}) {
  console.log(x, y);
}
foo({}) // undefined 5
foo({x: 1}) // 1 5
foo({x: 1, y: 2}) // 1 2
foo() // TypeError: Cannot read property 'x' of undefined
    
// 提供函数参数的默认值，避免报错
function foo({x, y = 5} = {}) {
  console.log(x, y);
}
foo() // undefined 5
    
// 练习
// 写法一
function m1({x = 0, y = 0} = {}) {
  return [x, y];
}
// 写法二
function m2({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}
// 函数没有参数的情况
m1() // [0, 0]
m2() // [0, 0]
// x 和 y 都有值的情况
m1({x: 3, y: 8}) // [3, 8]
m2({x: 3, y: 8}) // [3, 8]
// x 有值，y 无值的情况
m1({x: 3}) // [3, 0]
m2({x: 3}) // [3, undefined]
// x 和 y 都无值的情况
m1({}) // [0, 0];
m2({}) // [undefined, undefined]
m1({z: 3}) // [0, 0]
m2({z: 3}) // [undefined, undefined]
    
// 参数默认值的位置（如果非尾部的参数设置默认值，实际上这个参数是没法省略的）
// 例一
function f(x = 1, y) {
  return [x, y];
}
f() // [1, undefined]
f(2) // [2, undefined])
f(, 1) // 报错
// 例二
f(undefined, 1) // [1, 1]
function f(x, y = 5, z) {
  return [x, y, z];
}
f() // [undefined, 5, undefined]
f(1) // [1, 5, undefined]
f(1, ,2) // 报错
f(1, undefined, 2) // [1, 5, 2]
    
// 如果传入 undefined，将触发该参数等于默认值，null 则没有这个效果
function foo(x = 5, y = 6) {
  console.log(x, y);
}
foo(undefined, null)
// 5 null
    
// 函数的 length 属性（返回没有指定默认值的参数个数。指定了默认值后，length 属性将失真）
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2
// rest 参数也不会计入 length 属性
(function(...args) {}).length // 0
// 如果设置了默认值的参数不是尾参数，那么 length 属性也不再计入后面的参数了
(function (a = 0, b, c) {}).length // 0
(function (a, b = 1, c) {}).length // 1

// 作用域（一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域。等到初始化结束，这个作用域就会消失。这种语法行为，在不设置参数默认值时，是不会出现的）
var x = 1;
function f(x, y = x) {
  console.log(y);
}
// 参数 y 的默认值等于变量 x。调用函数 f 时，默认值变量 x 指向第一个参数 x，而不是全局变量 x，所以输出是 2
f(2) // 2
    
let x = 1;  // 全局变量 x 如果不存在会报错（ReferenceError: x is not defined）
function f(y = x) {
  let x = 2;
  console.log(y);
}
// 函数 f 调用时，变量 x 本身没有定义，所以指向外层的全局变量 x 。函数调用时，函数体内部的局部变量 x 影响不到默认值变量 x
f() // 1
    
var x = 1;
function foo(x = x) {
  // ...
}
// 数 x = x 形成一个单独作用域。实际执行的是 let x = x，由于暂时性死区的原因，这行代码会报错 ”x 未定义“
foo() // ReferenceError: x is not defined
    
// y 的默认值是一个匿名函数。这个匿名函数内部的变量 x，指向同一个作用域的第一个参数 x。函数 foo 内部又声明了一个内部变量 x，
// 该变量与第一个参数 x 由于不是同一个作用域，所以不是同一个变量，因此执行 y 后，内部变量 x 和外部全局变量 x 的值都没变
var x = 1;
function foo(x, y = function() { x = 2; }) {
  var x = 3;
  y();
  console.log(x);
}
foo() // 3
x // 1
    
// 如果将 var x = 3 的 var 去除，函数 foo 的内部变量 x 就指向第一个参数 x，与匿名函数内部的 x 是一致的，所以最后输出的就是 2，
// 而外层的全局变量 x 依然不受影响
var x = 1;
function foo(x, y = function() { x = 2; }) {
  x = 3;
  y();
  console.log(x);
}
foo() // 2
x // 1
```
### 二、rest 参数
ES6 引入 rest 参数（形式为...变量名），用于获取函数的多余参数，这样就不需要使用 arguments 对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。
```
// arguments 变量的写法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort();
}
    
// rest 参数的写法
const sortNumbers = (...numbers) => numbers.sort();
    
// rest 参数改写数组 push 方法
function push(array, ...items) {
  items.forEach(function(item) {
    array.push(item);
    console.log(item);
  });
}
var a = [];
push(a, 1, 2, 3)
  
// rest 参数之后不能再有其他参数（即只能是最后一个参数），否则会报错
function f(a, ...b, c) {  // 报错
  // ...
}

// 函数的 length 属性，不包括 rest 参数
(function(a) {}).length  // 1
(function(...a) {}).length  // 0
(function(a, ...b) {}).length  // 1
```
### 三、严格模式
从 ES5 开始，函数内部可以设定为严格模式。ES2016 做了一点修改，规定只要函数参数使用了默认值、解构赋值、或者扩展运算符，那么函数内部就不能显式设定为严格模式，否则会报错。    
这样规定的原因是，函数内部的严格模式，同时适用于函数体和函数参数。但是，函数执行的时候，先执行函数参数，然后再执行函数体。这样就有一个不合理的地方，只有从函数体之中，才能知道参数是否应该以严格模式执行，但是参数却应该先于函数体执行。
```
// 报错
function doSomething(a, b = a) {
  'use strict';
  // code
}
    
// 设定全局性的严格模式规避限制
'use strict';
function doSomething(a, b = a) {
  // code
}
    
// 函数包在一个无参数的立即执行函数里面规避限制
const doSomething = (function () {
  'use strict';
  return function(value = 42) {
    return value;
  };
}());
```
### 四、name 属性
函数的 name 属性，返回该函数的函数名。ES6 对这个属性的行为做出了一些修改。如果将一个匿名函数赋值给一个变量，ES5 的 name 属性，会返回空字符串，而 ES6 的 name 属性会返回实际的函数名。
```
var f = function () {};
// ES5
f.name // ""
// ES6
f.name // "f"
    
// Function 构造函数返回的函数实例，name 属性的值为 anonymous
(new Function).name // "anonymous"
    
// bind 返回的函数，name 属性值会加上 bound 前缀
function foo() {};
foo.bind({}).name // "bound foo"
(function(){}).bind({}).name // "bound "
```
### 五、函数参数的尾逗号
ES2017 允许函数的最后一个参数有尾逗号（trailing comma）。此前，函数定义和调用时，都不允许最后一个参数后面出现逗号。
```
function clownsEverywhere(
  param1,
  param2,
) { /* ... */ }

clownsEverywhere(
  'foo',
  'bar',
);
```
### 六、箭头函数
ES6 允许使用“箭头”（=>）定义函数。
```
var f = v => v;
    
// 如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分
var f = () => 5;
var sum = (num1, num2) => num1 + num2;
    
// 如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用 return 语句返回
var sum = (num1, num2) => { return num1 + num2; }
    
// 如果箭头函数直接返回一个对象，必须在对象外面加上括号，否则会报错
let getTempItem = id => { id: id, name: "Temp" };  // 报错
let getTempItem = id => ({ id: id, name: "Temp" });  // // 不报错
    
// 箭头函数可以与变量解构结合使用
const full = ({ first, last }) => first + ' ' + last;
    
// rest 参数与箭头函数结合
const headAndTail = (head, ...tail) => [head, tail];
headAndTail(1, 2, 3, 4, 5)
// [1,[2,3,4,5]]
    
function Timer() {
  this.s1 = 0;
  this.s2 = 0;
  // 箭头函数，this 总是指向函数定义生效时所在的对象
  setInterval(() => this.s1++, 1000);
  // 普通函数，执行时 this 应该指向全局对象 window
  setInterval(function () {
    this.s2++;
  }, 1000);
}
var timer = new Timer();
setTimeout(() => console.log('s1: ', timer.s1), 3100);
setTimeout(() => console.log('s2: ', timer.s2), 3100);
// s1: 3
// s2: 0
    
// 所有的内层函数都是箭头函数，都没有自己的 this，它们的 this 其实都是最外层 foo 函数的 this
function foo() {
  return () => {
    return () => {
      return () => {
        console.log('id:', this.id);
      };
    };
  };
}
var f = foo.call({id: 1});
var t1 = f.call({id: 2})()(); // id: 1
var t2 = f().call({id: 3})(); // id: 1
var t3 = f()().call({id: 4}); // id: 1
```
箭头函数有几个使用注意点：   
（1）函数体内的 this 对象，就是定义时所在的对象，而不是使用时所在的对象。    
（2）不可以当作构造函数，也就是说，不可以使用 new 命令，否则会抛出一个错误。   
（3）不可以使用 arguments 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。   
（4）不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。    