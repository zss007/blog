---
title: es6 之 module
categories:
- es6
---
历史上，JavaScript 一直没有模块（module）体系，无法将一个大程序拆分成互相依赖的小文件，再用简单的方法拼装起来。在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器。ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。
<!--more-->
### 一、简介
```
// CommonJS 模块
let { stat, exists, readFile } = require('fs');
// 等同于
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;
  
// ES6 模块
import { stat, exists, readFile } from 'fs';
```
上面 CommonJS 模块代码的实质是整体加载 fs 模块（即加载 fs 的所有方法），生成一个对象，然后再从这个对象上面读取 3 个方法。这种加载称为“运行时加载”，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。    
ES6 模块不是对象，而是通过 export 命令显式指定输出的代码，再通过 import 命令输入。上面 ES6 模块代码的实质是从 fs 模块加载 3 个方法，其他方法不加载。这种加载称为 “编译时加载” 或者静态加载，即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高。当然，这也导致了没法引用 ES6 模块本身，因为它不是对象。
### 二、export 命令
export 命令用于规定模块的对外接口，一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用 export 关键字输出该变量。
```
// profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;
  
// 等同于（后面方式更为推荐）
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;
export {firstName, lastName, year};
  
// export 命令除了输出变量，还可以输出函数或类
export function multiply(x, y) {
  return x * y;
};
  
// export 输出的变量就是本来的名字，但是可以使用 as 关键字重命名
function v1() { ... }
function v2() { ... }
export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
  
// export 命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系
export 1; // 报错
var m = 1;
export m; // 报错
export var m = 1; // 正确写法一
export {m}; // 正确写法二
var n = 1;
export {n as m};  // 正确写法三
  
// function 和 class 的输出，也必须遵守这样的写法
function f() {}
export f;  // 报错
export function f() {}; // 正确
export {f}; // 正确
  
// export 语句输出的接口，与其对应的值是动态绑定关系（输出变量 foo，值为 bar，500 毫秒之后变成 baz，与 CommonJS 规范完全不同）
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
  
// export 命令需要处于模块顶层。如果处于块级作用域内，就会报错，因为这样就没法做静态优化了
function foo() {
  export default 'bar' // SyntaxError
}
foo()
```
### 三、import 命令
使用 export 命令定义了模块的对外接口以后，其他 JS 文件就可以通过 import 命令加载这个模块
```
// main.js（大括号里面的变量名，必须与被导入模块 profile.js 对外接口的名称相同）
import {firstName, lastName, year} from './profile.js';
function setName(element) {
  element.textContent = firstName + ' ' + lastName;
}
  
// import 命令要使用 as 关键字，将输入的变量重命名
import { lastName as surname } from './profile.js';
  
// import 命令输入的变量都是只读的，不允许在加载模块的脚本里面，改写接口
import {a} from './xxx.js'
a = {}; // Syntax Error : 'a' is read-only;
// 但是，如果 a 是一个对象，改写 a 的属性是允许的，其他模块也可以读到改写后的值，不过这种写法很难查错，不建议
a.foo = 'hello'; // 合法操作
  
// import 后面的 from 指定模块文件的位置，可以是相对路径，也可以是绝对路径，.js后缀可以省略。模块名不带有路径，必须有配置文件告诉 JavaScript 引擎该模块的位置
import {myMethod} from 'util';
  
// import 命令具有提升效果，会提升到整个模块的头部，首先执行
foo(); // 不会报错，因为 import 的执行早于 foo 的调用。这种行为的本质是，import 命令是编译阶段执行的，在代码运行之前
import { foo } from 'my_module';
  
// import 是静态执行，所以不能使用表达式和变量
import { 'f' + 'oo' } from 'my_module'; // 报错
let module = 'my_module';
import { foo } from module; // 报错
if (x === 1) {  // 报错
  import { foo } from 'module1';
} else {
  import { foo } from 'module2';
}
  
// import 语句会执行所加载的模块
import 'lodash';  // 代码仅仅执行 lodash 模块，但是不输入任何值
  
// 多次重复执行同一句 import 语句，那么只会执行一次，而不会执行多次
import 'lodash';
import 'lodash'; // 加载了两次 lodash，但是只会执行一次
  
// import 语句是 Singleton 模式
import { foo } from 'my_module';
import { bar } from 'my_module';
// 等同于
import { foo, bar } from 'my_module';
  
// 通过 Babel 转码，CommonJS 模块的 require 命令和 ES6 模块的 import 命令，可以写在同一个模块里面，但是最好不要这样做，可能不会得到预期结果。因为 import 在静态解析阶段执行，所以它是一个模块之中最早执行的
require('core-js/modules/es6.symbol');
require('core-js/modules/es6.promise');
import React from 'React';
```
### 四、模块的整体加载
除了指定加载某个输出值，还可以使用整体加载，即用星号（*）指定一个对象，所有输出值都加载在这个对象上面。
```
// circle.js
export function area(radius) {
  return Math.PI * radius * radius;
}
export function circumference(radius) {
  return 2 * Math.PI * radius;
}
  
// main.js（逐一指定要加载的方法）
import { area, circumference } from './circle';
console.log('圆面积：' + area(4));
console.log('圆周长：' + circumference(14));
  
// main.js（整体加载的写法）
import * as circle from './circle';
console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(14));
  
// 模块整体加载所在的对象是可以静态分析的，所以不允许运行时改变。下面的写法都是不允许的
import * as circle from './circle';
circle.foo = 'hello';
circle.area = function () {};
```
### 五、export default 命令
使用 import 命令的时候，用户需要知道所要加载的变量名或函数名，否则无法加载。但是，用户肯定希望快速上手，未必愿意阅读文档，去了解模块有哪些属性和方法。这个时候就要用到 export default 命令，为模块指定默认输出。
```
// export-default.js
export default function () {
  console.log('foo');
}
  
// import-default.js（import 命令可以为该默认函数指定任意名字，后面不使用大括号。）
import customName from './export-default';
customName(); // 'foo'
  
// export-default.js（用在非匿名函数前，也是可以的；函数名 foo，在模块外部是无效的，加载的时候，视同匿名函数加载）
export default function foo() {
  console.log('foo');
}
// 或者写成
function foo() {
  console.log('foo');
}
export default foo;
  
// modules.js（export default 命令用于指定模块的默认输出，只能使用一次）
function add(x, y) {
  return x * y;
}
export {add as default};
// 等同于
// export default add;
  
// app.js（export default 就是输出一个叫做 default 的变量或方法，然后系统允许你为它取任意名字）
import { default as foo } from 'modules';
// 等同于
// import foo from 'modules';
  
// 因为 export default 命令其实只是输出一个叫做 default 的变量，所以它后面不能跟变量声明语句
export var a = 1; // 正确
var a = 1;
export default a; // 正确
export default var a = 1; // 错误
  
// 也可以直接将一个值写在 export default 之后
export default 42;  // 正确
export 42;  // 报错
  
// 有了 export default 命令，输入模块时就非常直观了
import _ from 'lodash';
  
// 在一条 import 语句中，同时输入默认方法和其他接口
import _, { each, each as forEach } from 'lodash';
  
// 对应上面代码的 export 语句如下
export default function (obj) {
  // ···
}
export function each(obj, iterator, context) {
  // ···
}
export { each as forEach }; // 暴露出 forEach 接口，默认指向 each 接口
  
// export default 也可以用来输出类
// MyClass.js
export default class { ... }
// main.js
import MyClass from 'MyClass';
let o = new MyClass();
```
### 六、export 与 import 的复合写法
如果在一个模块之中，先输入后输出同一个模块，import 语句可以与 export 语句写在一起。
```
// 写成一行以后，foo 和 bar 实际上并没有被导入当前模块，只是相当于对外转发了这两个接口，导致当前模块不能直接使用 foo 和 bar
export { foo, bar } from 'my_module';
// 可以简单理解为
import { foo, bar } from 'my_module';
export { foo, bar };
  
// 模块的接口改名和整体输出，也可以采用这种写法
export { foo as myFoo } from 'my_module'; // 接口改名
export * from 'my_module';  // 整体输出
  
// 默认接口的写法
export { default } from 'foo';
  
// 具名接口改为默认接口的写法
export { es6 as default } from './someModule';
// 等同于
import { es6 } from './someModule';
export default es6;
  
// 默认接口也可以改名为具名接口
export { default as es6 } from './someModule';
  
// 下面三种 import 语句，没有对应的复合写法
import * as someIdentifier from "someModule";
import someIdentifier from "someModule";
import someIdentifier, { namedIdentifier } from "someModule";
```
### 七、模块的继承
模块之间也可以继承。假设有一个 circleplus 模块，继承了 circle 模块。
```
// circleplus.js
export * from 'circle'; // export * 命令会忽略 circle 模块的 default 方法
export var e = 2.71828182846;
export default function(x) {
  return Math.exp(x);
}
  
// main.js
import * as math from 'circleplus';
import exp from 'circleplus'; // 将 circleplus 模块的默认方法加载为 exp 方法
console.log(exp(math.e));
```
### 八、跨模块常量
const 声明的常量只在当前代码块有效，如果想设置跨模块的常量或者一个值要被多个模块共享，可以采用下面的写法。
```
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;
  
// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3
  
// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
  
// 如果要使用的常量非常多，可以建一个专门的 constants 目录
export const db = { // constants/db.js
  url: 'http://my.couchdbserver.local:5984',
  admin_username: 'admin',
  admin_password: 'admin password'
};
export const users = ['root', 'admin', 'staff', 'ceo', 'chief', 'moderator']; // constants/user.js
  
// constants/index.js（将这些文件输出的常量，合并在 index.js 里面）
export {db} from './db';
export {users} from './users';
  
// script.js（使用的时候，直接加载 index.js 就可以了）
import {db, users} from './index';
```
### 九、import()
前面介绍过，import 命令会被 JavaScript 引擎静态分析，先于模块内的其他模块执行。这样的设计，固然有利于编译器提高效率，但也导致无法在运行时加载模块。在语法上，条件加载就不可能实现。如果 import 命令要取代 Node 的 require 方法，这就形成了一个障碍。因为 require 是运行时加载模块，import 命令无法取代 require 的动态加载功能。因此有一个提案，建议引入 import() 函数完成动态加载。