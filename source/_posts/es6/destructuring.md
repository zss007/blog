---
title: es6 之解构赋值
categories:
- es6
---
ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。
<!--more-->
### 一、数组的解构赋值
可以从数组中提取值，按照对应位置，对变量赋值。本质上等号两边的模式相同，左边的变量就会被赋予对应的值。
```
let [a, b, c] = [1, 2, 3];
a // 1
b // 2
c // 3
    
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3
    
let [ , , third] = ["foo", "bar", "baz"];
third // "baz"
    
let [x, , y] = [1, 2, 3];
x // 1
y // 3
    
let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]
    
let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
    
// 如果解构不成功，变量的值就等于 undefined
let [foo] = [];
let [bar, foo] = [1];
    
// 不完全解构，即等号左边的模式，只匹配一部分的等号右边的数组
let [x, y] = [1, 2, 3];
x // 1
y // 2
let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4
    
// 等号的右边不是可遍历的结构，那么将会报错（要么转为对象以后不具备 Iterator 接口，要么本身就不具备 Iterator 接口）
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;
let [foo] = {};
    
// 对于 Set 结构，也可以使用数组的解构赋值
let [x, y, z] = new Set(['a', 'b', 'c']);
x // "a"
    
// 只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值（fibs是一个 Generator 函数，原生具有 Iterator 接口，解构赋值会依次从这个接口获取值）
function* fibs() {
  let a = 0;
  let b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}
// 0 1 1 2 3 5
let [first, second, third, fourth, fifth, sixth] = fibs();
sixth // 5
    
// 解构赋值允许指定默认值
let [foo = true] = [];
foo // true
let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
    
// ES6 内部使用严格相等运算符（===），判断一个位置是否有值
let [x = 1] = [undefined];
x // 1
// null 不严格等于 undefined
let [x = 1] = [null];
x // null
    
// 默认值可以引用解构赋值的其他变量，但该变量必须已经声明
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
// x 用 y 做默认值时，y 还没有声明
let [x = y, y = 1] = [];     // ReferenceError: y is not defined
    
// 如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值
function f() {
  console.log('aaa');
}
let [x = f()] = [1];
// 上面代码中，因为 x 能取到值，所以函数 f 根本不会执行，上面的代码其实等价于下面的代码
let x;
if ([1][0] === undefined) {
  x = f();
} else {
  x = [1][0];
}
```
### 二、对象的解构赋值
解构不仅可以用于数组，还可以用于对象。对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。
```
let { bar, foo } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"
    
// 变量没有对应的同名属性，导致取不到值，最后等于undefined
let { baz } = { foo: "aaa", bar: "bbb" };
baz // undefined
    
// 变量名与属性名不一致时写法
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"
    
// 对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者
let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
    
// foo 是匹配的模式，baz 才是变量。真正被赋值的是变量 baz，而不是模式 foo
let { foo: baz } = { foo: "aaa", bar: "bbb" };
baz // "aaa"
foo // error: foo is not defined
     
// 解构也可以用于嵌套结构的对象
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};
let { loc, loc: { start }, loc: { start: { line }} } = node;
line // 1
loc  // Object {start: Object}
start // Object {line: 1, column: 5}
    
// 对象的解构也可以指定默认值
var {x = 3} = {};
x // 3
var {x, y = 5} = {x: 1};
x // 1
y // 5
var {x: y = 3} = {};
y // 3
var {x: y = 3} = {x: 5};
y // 5
var { message: msg = 'Something went wrong' } = {};
msg // "Something went wrong"
    
// 默认值生效的条件是，对象的属性值严格等于 undefined
var {x = 3} = {x: undefined};
x // 3
    
// null 与 undefined 不严格相等，所以是个有效的赋值，导致默认值 3 不会生效
var {x = 3} = {x: null};
x // null
    
// 解构失败，变量的值等于 undefined，如果解构模式是嵌套的对象，而且子对象所在的父属性不存在，那么将会报错
let {foo} = {bar: 'baz'};
foo // undefined
    
// 报错，相当于 let _tmp = {baz: 'baz'}; _tmp.foo.bar;
let {foo: {bar}} = {baz: 'baz'};
    
// 对象的解构赋值，可以很方便地将现有对象的方法，赋值到某个变量
let { log, sin, cos } = Math;
    
// 由于数组本质是特殊的对象，因此可以对数组进行对象属性的解构（方括号这种写法，属于“属性名表达式”）
let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;
first // 1
last // 3
    
// 如果要将一个已经声明的变量用于解构赋值，必须非常小心。下例 JavaScript 引擎会将 {x} 理解成一个代码块，从而发生语法错误
// 错误的写法 SyntaxError: syntax error
let x;
{x} = {x: 1};
// 正确的写法
let x;
({x} = {x: 1});
```
### 三、字符串的解构赋值
字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。
```
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
    
// 类似数组的对象都有一个 length 属性，因此还可以对这个属性解构赋值
let {length : len} = 'hello';
len // 5
```
### 四、数值和布尔值的解构赋值
解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。
```
let {toString: s} = 123;
s === Number.prototype.toString // true
    
let {toString: s} = true;
s === Boolean.prototype.toString // true
    
// 只要等号右边的值不是对象或数组，就先将其转为对象。由于 undefined 和 null 无法转为对象，所以对它们进行解构赋值，都会报错
let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```
### 五、函数参数的解构赋值
函数的参数也可以使用解构赋值。
```
// 传入参数的那一刻，数组参数就被解构成变量 x 和 y。对于函数内部的代码来说，它们能感受到的参数就是 x 和 y
function add([x, y]){
  return x + y;
}
add([1, 2]); // 3
    
// 另一个例子
[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [ 3, 7 ]
    
// 函数参数的解构也可以使用默认值
function move({x = 0, y = 0} = {}) {
  return [x, y];
}
move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
    
// 下面的写法会得到不一样的结果
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}
move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
    
// undefined 就会触发函数参数的默认值
[1, undefined, 3].map((x = 'yes') => x);
// [ 1, 'yes', 3 ]
```
### 六、圆括号问题
解构赋值虽然很方便，但是解析起来并不容易。对于编译器来说，一个式子到底是模式，还是表达式，没有办法从一开始就知道，必须解析到（或解析不到）等号才能知道。由此带来的问题是，如果模式中出现圆括号怎么处理。ES6 的规则是，只要有可能导致解构的歧义，就不得使用圆括号。但是，这条规则实际上不那么容易辨别，处理起来相当麻烦。因此，建议只要有可能，就不要在模式中放置圆括号。
```
// 变量声明语句，模式不能使用圆括号，报错
let [(a)] = [1];
let {x: (c)} = {};
let ({x: c}) = {};
let {(x: c)} = {};
let {(x): c} = {};
let { o: ({ p: p }) } = { o: { p: 2 } };
    
// 函数参数也属于变量声明，因此不能带有圆括号，报错
function f([(z)]) { return z; }
function f([z,(x)]) { return x; }
    
// 赋值语句的模式，报错
({ p: a }) = { p: 42 };
([a]) = [5];
[({ p: a }), { x: c }] = [{}, {}];
    
// 模式是取数组的第一个成员，跟圆括号无关
[(b)] = [3]; // 正确
    
// 模式是p，而不是d
({ p: (d) } = {}); // 正确
    
// 模式是取数组的第一个成员，跟圆括号无关
[(parseInt.prop)] = [3]; // 正确
```
### 七、用途
```
// 交换变量的值
let x = 1;
let y = 2;
[x, y] = [y, x];
    
// 从函数返回多个值（返回一个数组 | 返回一个对象）
function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();
function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
    
// 函数参数的定义（参数是一组有次序的值 | 参数是一组无次序的值）
function f([x, y, z]) { ... }
f([1, 2, 3]);
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
  
// 提取 JSON 数据
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};
let { id, status, data: number } = jsonData;
console.log(id, status, number);
// 42, "OK", [867, 5309]
    
// 函数参数的默认值（避免了在函数体内部再写var foo = config.foo || 'default foo';这样的语句）
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
}) {
  // ... do stuff
};
    
// 遍历 Map 结构
const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');
for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world
    
// 只想获取键名，或者只想获取键值
for (let [key] of map) {
  // ...
}
for (let [,value] of map) {
  // ...
}
    
// 输入模块的指定方法
const { SourceMapConsumer, SourceNode } = require("source-map");
```
变量的解构赋值用途很多，如上。