---
title: es6 之 let & const
categories:
- es6
---
ES6 添加 let 和 const 两种声明变量的命令，现在简单介绍下。
<!--more-->
### 一、let
用法类似于 var，但是所声明的变量，只在 let 命令所在的代码块内有效：
```
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```
再比如循环中的使用：    
使用 ES5 的 var 变量声明的话，在例子中会导致函数内部的 i 都指向同一个 i（全局的），最后都输出 10；  
如果使用 let，声明的变量仅在块级作用域内有效，最后输出的是 6。  
变量 i 是 let 声明的，当前的 i 只在本轮循环有效，所以每一次循环的 i 其实都是一个新的变量，所以最后输出的是 6，
JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量 i 时，就在上一轮循环的基础上进行计算。   
另外，for 循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。
```
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
    
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
    
// 正常运行，函数内部的变量 i 与循环变量 i 不在同一个作用域，有各自单独的作用域。如果在同一作用域会报错，见下文。
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
```
let 命令改变了 ES6 的 var 语法行为，它所声明的变量一定要在声明后使用，在声明它之前，变量bar是不存在的，这时如果用到它，就会抛出一个错误。
```
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;
    
// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```
let 不允许在相同作用域内，重复声明同一个变量
```
// 报错
function func() {
  let a = 10;
  var a = 1;
}
    
// 报错
function func() {
  let a = 10;
  let a = 1;
}
    
function func(arg) {
  let arg; // 报错
}
    
function func(arg) {
  {
    let arg; // 不报错
  }
}
```
只要块级作用域内存在 let 命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响（称为暂时性死区）。本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。
```
// 存在全局变量 tmp，但是块级作用域内 let 又声明了一个局部变量 tmp，导致后者绑定这个块级作用域，所以在 let 声明变量前，对 tmp 赋值会报错。
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
    
// 参数 x 默认值等于另一个参数 y ，而此时 y 还没有声明，属于”死区“（某些实现可能不报错）
function bar(x = y, y = 2) {
  return [x, y];
}

bar(); // 报错
    
// 不报错
var x = x;

// 报错；在变量 x 的声明语句还没有执行完成前，就去取 x 的值，导致报错”x 未定义“。
let x = x;
// ReferenceError: x is not defined
```
### 二、const
const 声明一个只读的常量。一旦声明，常量的值就不能改变。一旦声明变量，就必须立即初始化，不能留到以后赋值，否则会报错。
```
const PI = 3.1415;
PI // 3.1415

PI = 3;
// TypeError: Assignment to constant variable.
    
const foo;
// SyntaxError: Missing initializer in const declaration
```
作用域与 let 命令相同：只在声明所在的块级作用域内有效。
```
if (true) {
  const MAX = 5;
}
MAX // Uncaught ReferenceError: MAX is not defined
    
// 声明的常量同样不提升，同样存在暂时性死区，只能在声明的位置后面使用。
if (true) {
  console.log(MAX); // ReferenceError
  const MAX = 5;
}
    
// 声明的常量，也与let一样不可重复声明。
var message = "Hello!";
let age = 25;
// 以下两行都会报错
const message = "Goodbye!";
const age = 30;
```
const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指针，const只能保证这个指针是固定的，至于它指向的数据结构是不是可变的，就完全不能控制了。
```
// 常量foo储存的是一个地址，这个地址指向一个对象。不可变的只是这个地址，即不能把foo指向另一个地址，但对象本身是可变的，所以依然可以为其添加新属性。
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
      
// 常量 a 是一个数组，这个数组本身是可写的，但是如果将另一个数组赋值给 a ，就会报错。
const a = [];
a.push('Hello'); // 可执行
a.length = 0;    // 可执行
a = ['Dave'];    // 报错
      
// 真的想将对象冻结，应该使用Object.freeze方法。
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```
### 三、顶层对象的属性
顶层对象，在浏览器环境指的是window对象，在 Node 指的是global对象。ES5 之中，顶层对象的属性与全局变量是等价的。
```
window.a = 1;
a // 1
    
a = 2;
window.a // 2
```
ES6 规定，为了保持兼容性，var 命令和 function 命令声明的全局变量，依旧是顶层对象的属性，而let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性。也就是说，从 ES6 开始，全局变量将逐步与顶层对象的属性脱钩。
```
var a = 1;
// 如果在 Node 的 REPL 环境，可以写成 global.a
// 或者采用通用方法，写成 this.a
window.a // 1
    
let b = 1;
window.b // undefined
```
### 四、块级作用域
ES5 只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。
```
// 内层变量可能会覆盖外层变量
var tmp = new Date();
function f() {
  console.log(tmp);
  if (false) {
    var tmp = 'hello world';
  }
}
f(); // undefined
    
// 用来计数的循环变量泄露为全局变量。
var s = 'hello';
for (var i = 0; i < s.length; i++) {
  console.log(s[i]);
}
console.log(i); // 5
```
let、const 实际上为 JavaScript 新增了块级作用域
```
// 运行后输出 5，表示外层代码块不受内层代码块的影响。如果两次都使用 var 定义变量 n，最后输出的值才是 10。
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
    
// 允许块级作用域的任意嵌套，外层作用域无法读取内层作用域的变量。
{{{{
  {let insane = 'Hello World'}
  console.log(insane); // 报错
}}}};
    
// 内层作用域可以定义外层作用域的同名变量。
{{{{
  let insane = 'Hello World';
  {let insane = 'Hello World'}
}}}};
    
// 块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。
// IIFE 写法
(function () {
  var tmp = ...;
  ...
}());
// 块级作用域写法
{
  let tmp = ...;
  ...
}
```
ES5 规定，函数只能在顶层作用域和函数作用域之中声明，不能在块级作用域声明，但是为了兼容以前的旧代码，浏览器没有遵守这个规定。    
ES6 引入了块级作用域，明确允许在块级作用域之中声明函数。ES6 规定，块级作用域之中，函数声明语句的行为类似于 let，在块级作用域之外不可引用。同时ES6 在附录里面规定，浏览器兼容为了老代码，实现上可以不遵守上面的规定，有自己的行为方式，函数声明类似于var。考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式，而不是函数声明语句。