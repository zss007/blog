---
title: es6 之 module 加载
categories:
- es6
---
介绍浏览器和 Node 之中加载 ES6 模块，以及实际开发中经常遇到的一些问题（比如循环加载）。
<!--more-->
### 一、浏览器加载
HTML 网页中，浏览器通过 `<script>` 标签加载 JavaScript 脚本。默认情况下，浏览器是同步加载 JavaScript 脚本，即渲染引擎遇到 `<script>` 标签就会停下来，等到执行完脚本，再继续向下渲染。如果是外部脚本，还必须加入脚本下载的时间。如果脚本体积很大，下载和执行的时间就会很长，因此造成浏览器堵塞，用户会感觉到浏览器“卡死”了，没有任何响应。这显然是很不好的体验，所以浏览器允许脚本异步加载。
```
// defer 要等到整个页面在内存中正常渲染结束（DOM 结构完全生成，以及其他脚本执行完成），才会执行；多个 defer 脚本，会按照它们在页面出现的顺序加载
<script src="path/to/myModule.js" defer></script>
  
// async 一旦下载完，渲染引擎就会中断渲染，执行这个脚本以后，再继续渲染；多个 async 脚本是不能保证加载顺序
<script src="path/to/myModule.js" async></script>
  
// 浏览器加载 ES6 模块（加入 type="module" 属性，效果等同于 defer 属性）
<script type="module" src="./foo.js"></script>
  
// 浏览器加载 ES6 模块，同时打开 async 属性（只要加载完成，渲染引擎就会中断渲染立即执行。执行完成后，再恢复渲染）
<script type="module" src="./foo.js" async></script>
  
// ES6 模块也允许内嵌在网页中，语法行为与加载外部脚本完全一致
<script type="module">
  import utils from "./utils.js";
  // other code
</script>
  
// ES6 模块之中，顶层的 this 返回undefined，而不是指向 window 。也就是说，在模块顶层使用 this 关键字，是无意义的
import utils from 'https://example.com/js/utils.js';
const x = 1;
console.log(x === window.x); //false
console.log(this === undefined); // true
  
// 利用顶层的 this 等于 undefined 这个语法点，可以侦测当前代码是否在 ES6 模块之中
const isNotModuleScript = this !== undefined;
```
对于外部的模块脚本，有几点需要注意：  
1.代码是在模块作用域之中运行，而不是在全局作用域运行。模块内部的顶层变量，外部不可见。  
2.模块脚本自动采用严格模式，不管有没有声明use strict。   
3.模块之中，可以使用import命令加载其他模块（.js后缀不可省略，需要提供绝对 URL 或相对 URL），也可以使用export命令输出对外接口。  
4.模块之中，顶层的this关键字返回undefined，而不是指向window。也就是说，在模块顶层使用this关键字，是无意义的。 
5.同一个模块如果加载多次，将只执行一次。 
### 二、ES6 模块与 CommonJS 模块的差异
他们有两个重大差异：CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用；CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。第二个差异是因为 CommonJS 加载的是一个对象（即module.exports属性），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。
```
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};
  
// main.js（mod.counter 是一个原始类型的值，会被缓存）
var mod = require('./lib');
console.log(mod.counter);  // 3
mod.incCounter();
console.log(mod.counter); // 3
  
// lib.js（除非写成一个函数，才能得到内部变动后的值）
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  get counter() { // 输出的 counter 属性实际上是一个取值器函数
    return counter
  },
  incCounter: incCounter,
};
  
// lib.js（ES6 模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块）
export let counter = 3;
export function incCounter() {
  counter++;
}
  
// main.js（原始值变了，import 加载的值也会跟着变）
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
  
// lib.js
export let obj = {};
  
// main.js
import { obj } from './lib';
obj.prop = 123; // OK
obj = {}; // TypeError（变量 obj 指向的地址是只读的，不能重新赋值，这就好比 main.js 创造了一个名为 obj 的 const 变量）
  
// mod.js（export 通过接口，输出的是同一个值。不同的脚本加载这个接口，得到的都是同样的实例）
function C() {
  this.sum = 0;
  this.add = function () {
    this.sum += 1;
  };
  this.show = function () {
    console.log(this.sum);
  };
}
export let c = new C();
  
// x.js
import {c} from './mod';
c.add();
  
// y.js
import {c} from './mod';
c.show();
  
// main.js（输出 1，证明了 x.js 和 y.js 加载的都是 C 的同一个实例）
import './x';
import './y';
```
### 三、Node 加载
Node 对 ES6 模块的处理比较麻烦，因为它有自己的 CommonJS 模块格式，与 ES6 模块格式是不兼容的。目前的解决方案是，将两者分开，ES6 模块和 CommonJS 采用各自的加载方案。Node 要求 ES6 模块采用 .mjs 后缀文件名。也就是说，只要脚本文件里面使用 import 或者 export 命令，那么就必须采用 .mjs 后缀名。require 命令不能加载 .mjs 文件，会报错，只有 import 命令才可以加载 .mjs 文件。反过来，.mjs 文件里面也不能使用 require 命令，必须使用 import。目前，这项功能还在试验阶段。安装 Node v8.5.0 或以上版本，要用 --experimental-modules 参数才能打开该功能。
```
// 支持 URL 路径，Node 会按 URL 规则解读；同一个脚本只要参数不同，就会被加载多次，并且保存成不同的缓存。由于这个原因，只要文件名中含有:、%、#、? 等特殊字符，最好对这些字符进行转义
import './foo?query=1'; // 加载 ./foo 传入参数 ?query=1
  
// 如果模块名不含路径，那么 import 命令会去 node_modules 目录寻找这个模块
import 'baz';
import 'abc/123';
  
// 如果模块名包含路径，那么 import 命令会按照路径去寻找这个名字的脚本文件
import 'file:///etc/config/app.json';  // 只支持加载本地模块（file:协议），不支持加载远程模块
import './foo';
import './foo?search';
import '../bar';
import '/baz';
```
如果脚本文件省略了后缀名，比如 import './foo'，Node 会依次尝试四个后缀名：./foo.mjs、./foo.js、./foo.json、./foo.node。如果这些脚本文件都不存在，Node 就会去加载 ./foo/package.json 的 main 字段指定的脚本。如果 ./foo/package.json 不存在或者没有 main 字段，那么就会依次加载 ./foo/index.mjs、./foo/index.js、./foo/index.json、./foo/index.node。如果以上四个文件还是都不存在，就会抛出错误。Node 的 import 命令是异步加载，这一点与浏览器的处理方法相同。  
ES6 模块应该是通用的，同一个模块不用修改，就可以用在浏览器环境和服务器环境。为了达到这个目标，Node 规定 ES6 模块之中不能使用 CommonJS 模块的特有的一些内部变量。首先，就是this关键字。ES6 模块之中，顶层的 this 指向 undefined；CommonJS 模块的顶层 this 指向当前模块。其次，以下这些顶层变量在 ES6 模块之中都是不存在的：arguments、require、module、exports、`__filename`、__dirname。
### 四、ES6 模块加载 CommonJS 模块
CommonJS 模块的输出都定义在 module.exports 这个属性上面。Node 的 import 命令加载 CommonJS 模块，Node 会自动将 module.exports 属性，当作模块的默认输出，即等同于export default xxx。
```
// a.js
module.exports = {
  foo: 'hello',
  bar: 'world'
};
// 等同于（即 import 命令实际上输入的是这样一个对象 { default: module.exports }）
export default {
  foo: 'hello',
  bar: 'world'
};
  
// 写法一（一共有三种写法，可以拿到 CommonJS 模块的 module.exports）
import baz from './a';
// baz = {foo: 'hello', bar: 'world'};
// 写法二
import {default as baz} from './a';
// baz = {foo: 'hello', bar: 'world'};
// 写法三（通过 baz.default 拿到 module.exports）
import * as baz from './a';
// baz = {
//   get default() {return module.exports;},
//   get foo() {return this.default.foo}.bind(baz),
//   get bar() {return this.default.bar}.bind(baz)
// }
  
// b.js
module.exports = null;
// es.js
import foo from './b';
// foo = null;
import * as bar from './b';
// bar = { default:null }; // 通过 bar.default 这样的写法，才能拿到 module.exports
  
// c.js
module.exports = function two() {
  return 2;
};
// es.js
import foo from './c';
foo(); // 2
import * as bar from './c';
bar.default(); // 2
bar(); // throws, bar is not a function（bar 本身是一个对象，不能当作函数调用，只能通过 bar.default 调用）
  
// foo.js（CommonJS 模块的输出缓存机制，在 ES6 加载方式下依然有效）
module.exports = 123;
setTimeout(_ => module.exports = null); // 对于加载 foo.js 的脚本，module.exports 将一直是 123，而不会变成 null
  
// ES6 模块是编译时确定输出接口，CommonJS 模块是运行时确定输出接口
import { readfile } from 'fs';  // 不正确，因为 fs 是 CommonJS 格式，只有在运行时才能确定 readfile 接口，而 import 命令要求编译时就确定这个接口
  
// 解决方法就是改为整体输入
import * as express from 'express'; // 写法一
const app = express.default();
import express from 'express';  // 写法二
const app = express();
```
### 五、CommonJS 模块加载 ES6 模块
CommonJS 模块加载 ES6 模块，不能使用 require 命令，而要使用 import() 函数。ES6 模块的所有输出接口，会成为输入对象的属性。
```
// es.mjs
let foo = { bar: 'my-default' };
export default foo;
foo = null;
  
// cjs.js
const es_namespace = await import('./es');
// es_namespace = {
//   get default() {
//     ...
//   }
// }
console.log(es_namespace.default); // 由于存在缓存机制，es.mjs 对 foo 的重新赋值没有在模块外部反映出来
// { bar:'my-default' }
  
// es.js
export let foo = { bar:'my-default' };
export { foo as bar };
export function f() {};
export class c {};

// cjs.js
const es_namespace = await import('./es');
// es_namespace = {
//   get foo() {return foo;}
//   get bar() {return foo;}
//   get f() {return f;}
//   get c() {return c;}
// }
```
### 六、循环加载
"循环加载" 指的是，a 脚本的执行依赖 b 脚本，而 b 脚本的执行又依赖 a 脚本。通常，“循环加载”表示存在强耦合，如果处理不好，还可能导致递归加载，使得程序无法执行，因此应该避免出现。但是实际上，这是很难避免的，尤其是依赖关系复杂的大项目，很容易出现 a 依赖 b，b 依赖 c，c 又依赖 a 这样的情况。这意味着，模块加载机制必须考虑 "循环加载" 的情况。目前最常见的两种模块格式 CommonJS 和 ES6，处理“循环加载”的方法是不一样的，返回的结果也不一样。
```
// require 命令生成的对象
{
  id: '...',  // id 属性是模块名
  exports: { ... }, // 模块输出的各个接口
  loaded: true, // 布尔值，表示该模块的脚本是否执行完毕
  ... // 其他属性
}
  
// a.js
exports.done = false;
var b = require('./b.js');  // 1、加载另一个脚本文件 b.js，此时 a.js 代码就停在这里，等待 b.js 执行完毕，再往下执行
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 执行完毕'); // 5、a.js 接着往下执行，直到执行完毕
  
// b.js
exports.done = false;
var a = require('./a.js');  // 2、发生了 "循环加载"，系统会去 a.js 模块对应对象的 exports 属性取值，可是因为 a.js 还没有执行完，从 exports 属性只能取回已经执行的部分，而不是最后的值
console.log('在 b.js 之中，a.done = %j', a.done); // 3、已经执行的部分，只有一行，从 a.js 只输入一个变量 done，值为 false
exports.done = true;
console.log('b.js 执行完毕'); // 4、全部执行完毕，再把执行权交还给 a.js
  
// main.js
var a = require('./a.js');  // 0、开始加载 a.js
var b = require('./b.js');  // 6、不会再次执行 b.js，而是输出缓存的 b.js 的执行结果
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);
// 在 b.js 之中，a.done = false
// b.js 执行完毕
// 在 a.js 之中，b.done = true
// a.js 执行完毕
// 在 main.js 之中, a.done=true, b.done=true
  
// 由于 CommonJS 模块遇到循环加载时，返回的是当前已经执行的部分的值，而不是代码全部执行后的值，两者可能会有差异。所以，输入变量的时候，必须非常小心。
var a = require('a'); // 安全的写法
var foo = require('a').foo; // 危险的写法

exports.good = function (arg) {
  return a.foo('good', arg); // 使用的是 a.foo 的最新值
};
exports.bad = function (arg) {
  return foo('bad', arg); // 使用的是一个部分加载时的值
};
```
1、先介绍目前最流行的 CommonJS 模块格式的加载原理：CommonJS 的一个模块，就是一个脚本文件。require 命令第一次加载该脚本，就会执行整个脚本，然后在内存生成一个对象。以后需要用到这个模块的时候，就会到 exports 属性上面取值。即使再次执行 require 命令，也不会再次执行该模块，而是到缓存之中取值。也就是说，CommonJS 模块无论加载多少次，都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果，除非手动清除系统缓存。CommonJS 模块的重要特性是加载时执行，即脚本代码在 require 的时候，就会全部执行。一旦出现某个模块被 "循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。如上。
```
// a.mjs
import {bar} from './b'; // 1、执行 b.mjs，然后再执行 a.js
console.log('a.mjs');
console.log(bar);
export let foo = 'foo';
  
// b.mjs
import {foo} from './a'; // 2、不会去执行 a.mjs，而是认为这个接口已经存在了，继续往下执行
console.log('b.mjs');
console.log(foo);  // 3、发现这个接口根本没定义，因此报错
export let bar = 'bar';
// node --experimental-modules a.mjs
// b.mjs
// ReferenceError: foo is not defined
  
// 解决方法：因为函数具有提升作用，在执行 import {bar} from './b' 时，函数 foo 就已经有定义了，所以 b.mjs 加载的时候不会报错
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar());
function foo() { return 'foo' }
export {foo};
  
// b.mjs
import {foo} from './a';
console.log('b.mjs');
console.log(foo());
function bar() { return 'bar' }
export {bar};
  
// a.mjs（代码第四行改成了函数表达式，就不具有提升作用，执行就会报错）
import {bar} from './b';
console.log('a.mjs');
console.log(bar());
const foo = () => 'foo';
export {foo};
  
// even.js
import { odd } from './odd'
export var counter = 0;
export function even(n) {
  counter++;
  return n === 0 || odd(n - 1);
}
// odd.js
import { even } from './even';
export function odd(n) {
  return n !== 0 && even(n - 1);
}
$ babel-node
> import * as m from './even.js';
> m.even(10);
true
> m.counter
6
> m.even(20)
true
> m.counter
17
// 要是改写成 CommonJS，就根本无法执行，会报错：
// even.js 加载 odd.js，而 odd.js 又去加载 even.js，形成 "循环加载"。这时，执行引擎就会输出 even.js 已经执行的部分（不存在任何结果），
// 所以在 odd.js 之中，变量 even 等于 null，等到后面调用 even(n-1) 就会报错。
```
2、ES6 模块是动态引用，如果使用 import 从一个模块加载变量（即import foo from 'foo'），那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。