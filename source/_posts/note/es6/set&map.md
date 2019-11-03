---
title: es6 之 Set & Map
categories:
- note
---
ES6 引入了新的数据结构 Set、WeakSet、Map 以及 WeakMap。
<!--more-->
### 一、Set
ES6 提供了新的数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。
```
// Set 本身是一个构造函数，用来生成 Set 数据结构
const s = new Set();
[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));
for (let i of s) {
  console.log(i); // 2 3 5 4（结果表明 Set 结构不会添加重复的值）
}
  
// Set 函数可以接受一个数组（或者具有 iterable 接口的其他数据结构）作为参数，用来初始化
const set = new Set([1, 2, 3, 4, 4]); // 例一
[...set]  // [1, 2, 3, 4]
const items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);  // 例二
items.size // 5
const set = new Set(document.querySelectorAll('div'));  // 例三，接受类似数组的对象作为参数
set.size // 56
const set = new Set();  // 类似于
document
 .querySelectorAll('div')
 .forEach(div => set.add(div));
set.size // 56
  
// 可用于去除数组的重复成员
[...new Set(array)]
  
// 向 Set 加入值的时候，不会发生类型转换，所以 5 和 "5" 是两个不同的值
let set = new Set();
let a = NaN;
let b = NaN;
set.add(a);
set.add(b);
set // Set {NaN}（Set 内部判断两个值是否不同类似于精确相等运算符 ===，主要的区别是 NaN 等于自身，而精确相等运算符认为 NaN 不等于自身）
  
// 两个对象总是不相等的
let set = new Set();
set.add({});
set.size // 1
set.add({});
set.size // 2
  
// 原型属性：constructor、size；实例操作方法：add、delete、has、clear（清除所有成员，没有返回值）
s.add(1).add(2).add(2); // 注意2被加入了两次
s.size // 2
s.has(1) // true
s.has(2) // true
s.has(3) // false
s.delete(2);
s.has(2) // false
  
// Object 结构和 Set 结构在判断是否包括一个键上面的写法不相同
const properties = {  // 对象的写法
  'width': 1,
  'height': 1
};
if (properties[someName]) {
  // do something
}
const properties = new Set(); // Set的写法
properties.add('width');
properties.add('height');
if (properties.has(someName)) {
  // do something
}
  
// Array.from 方法可以将 Set 结构转为数组
const items = new Set([1, 2, 3, 4, 5]);
const array = Array.from(items);
  
// 提供了去除数组重复成员的另一种方法
function dedupe(array) {
  return Array.from(new Set(array));
}
dedupe([1, 1, 2, 3]) // [1, 2, 3]
  
// Set 结构的遍历方法：keys、values、entries
let set = new Set(['red', 'green', 'blue']);
for (let item of set.keys()) {
  console.log(item);  // red green blue，由于 Set 结构没有键名，只有键值，所以 keys 方法和 values 方法的行为完全一致
}
for (let item of set.values()) {
  console.log(item);  // red green blue
}
for (let item of set.entries()) {
  console.log(item);  // ["red", "red"] ["green", "green"] ["blue", "blue"]
}
  
// Set 结构的遍历方法：forEach（参数与数组的 forEach 一致，依次为键值、键名、集合本身）
set = new Set([1, 4, 9]);
set.forEach((value, key) => console.log(key + ' : ' + value)) // 注意 Set 结构的键名就是键值，因此第一个参数与第二个参数的值永远都一样
// 1 : 1
// 4 : 4
// 9 : 9
  
// Set 结构的实例默认可遍历，它的默认遍历器生成函数就是它的values方法
Set.prototype[Symbol.iterator] === Set.prototype.values
  
// 可以省略 values 方法，直接用 for...of 循环遍历 Set
let set = new Set(['red', 'green', 'blue']);
for (let x of set) {
  console.log(x); // red green blue
}
  
// 扩展运算符（...）内部使用 for...of 循环，所以也可以用于 Set 结构
let set = new Set(['red', 'green', 'blue']);
let arr = [...set]; // ['red', 'green', 'blue']
  
// 扩展运算符和 Set 结构相结合，就可以去除数组的重复成员
let arr = [3, 5, 2, 2, 5, 5];
let unique = [...new Set(arr)]; // [3, 5, 2]
  
// 数组的 map 和 filter 方法也可以间接用于 Set
let set = new Set([1, 2, 3]);
set = new Set([...set].map(x => x * 2));  // 返回Set结构：{2, 4, 6}
let set = new Set([1, 2, 3, 4, 5]);
set = new Set([...set].filter(x => (x % 2) == 0));  // 返回Set结构：{2, 4}
  
// 使用 Set 可以很容易地实现并集（Union）、交集（Intersect）和差集（Difference）
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);
let union = new Set([...a, ...b]);  // Set {1, 2, 3, 4}，并集
let intersect = new Set([...a].filter(x => b.has(x)));  // set {2, 3}，交集
let difference = new Set([...a].filter(x => !b.has(x)));  // Set {1}，差集
  
// 没有直接改变原来 Set 结构的方法：利用原 Set 结构映射出一个新的结构，然后赋值给原来的 Set 结构；利用 Array.from 方法
let set = new Set([1, 2, 3]); // 方法一
set = new Set([...set].map(val => val * 2));  // set的值是2, 4, 6
let set = new Set([1, 2, 3]); // 方法二
set = new Set(Array.from(set, val => val * 2)); // set的值是2, 4, 6
```
### 二、SetWeakSet
WeakSet 结构与 Set 类似，也是不重复的值的集合。但是，它与 Set 有两个区别。首先，WeakSet 的成员只能是对象，而不能是其他类型的值。其次，WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。WeakSet 的成员是不适合引用的，因为它会随时消失，取决于垃圾回收机制有没有运行，因此 ES6 规定 WeakSet 不可遍历。
```
// WeakSet 是一个构造函数，可以使用 new 命令，创建 WeakSet 数据结构
const ws = new WeakSet();
  
// 作为构造函数，WeakSet 可以接受一个数组或类似数组的对象作为参数
const a = [[1, 2], [3, 4]];
const ws = new WeakSet(a);  // WeakSet {[1, 2], [3, 4]}（注意，是 a 数组的成员成为 WeakSet 的成员，而不是 a 数组本身）
  
// WeakSet 的成员只能是对象，而不能是其他类型的值
const ws = new WeakSet();
ws.add(1) // TypeError: Invalid value used in weak set
ws.add(Symbol())  // TypeError: invalid value used in weak set
  
// 注意，作为构造函数参数的数组，它的成员只能是对象
const b = [3, 4];
const ws = new WeakSet(b);  // Uncaught TypeError: Invalid value used in weak set(…)
  
// WeakSet 原型方法：add、delete、has
const ws = new WeakSet();
const obj = {};
const foo = {};
ws.add(window);
ws.add(obj);
ws.has(window); // true
ws.has(foo);    // false
ws.delete(window);
ws.has(window);    // false
  
// WeakSet 没有 size 属性，没有办法遍历它的成员
ws.size // undefined
ws.forEach // undefined
ws.forEach(function(item){ console.log('WeakSet has ' + item)}) // TypeError: undefined is not a function
  
// WeakSet 的一个用处，是储存 DOM 节点，而不用担心这些节点从文档移除时，会引发内存泄漏
// 另外一个作用保证 Foo 的实例方法只能在 Foo 的实例上调用，好处是删除实例的时候不用考虑foos，也不会出现内存泄漏
const foos = new WeakSet()
class Foo {
  constructor() {
    foos.add(this)
  }
  method () {
    if (!foos.has(this)) {
      throw new TypeError('Foo.prototype.method 只能在Foo的实例上调用！');
    }
  }
}
```
### 三、Map
JavaScript 的对象（Object），本质上是键值对的集合（Hash 结构），但是传统上只能用字符串当作键，这给它的使用带来了很大的限制。为了解决这个问题，ES6 提供了 Map 数据结构。它类似于对象，也是键值对的集合，但是 "键" 的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object 结构提供了 "字符串—值" 的对应，Map 结构提供了 "值—值" 的对应，是一种更完善的 Hash 结构实现。如果你需要 "键值对" 的数据结构，Map 比 Object 更合适。
```
// 原意是将一个 DOM 节点作为对象 data 的键，但是由于对象只接受字符串作为键名，所以 element 被自动转为字符串 [object HTMLDivElement]
const data = {};
const element = document.getElementById('myDiv');
data[element] = 'metadata';
data['[object HTMLDivElement]'] // "metadata"
  
// 使用 Map 结构的 set 方法，将对象 o 当作 m 的一个键，然后又使用 get 方法读取这个键，接着使用 delete 方法删除了这个键
const m = new Map();
const o = {p: 'Hello World'};
m.set(o, 'content')
m.get(o) // "content"
m.has(o) // true
m.delete(o) // true
m.has(o) // false
  
// 作为构造函数，Map 也可以接受一个数组作为参数，该数组的成员是一个个表示键值对的数组
const map = new Map([
  ['name', '张三'],
  ['title', 'Author']
]);
map.size // 2
map.has('name') // true
map.get('name') // "张三"
map.has('title') // true
map.get('title') // "Author"
  
// Map 构造函数接受数组作为参数，实际上执行的是下面的算法
const items = [
  ['name', '张三'],
  ['title', 'Author']
];
const map = new Map();
items.forEach(
  ([key, value]) => map.set(key, value)
);
  
// 任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构都可以当作 Map 构造函数的参数。这就是说，Set 和 Map 都可以用来生成新的 Map
const set = new Set([
  ['foo', 1],
  ['bar', 2]
]);
const m1 = new Map(set);
m1.get('foo') // 1
const m2 = new Map([['baz', 3]]);
const m3 = new Map(m2);
m3.get('baz') // 3
  
// 对同一个键多次赋值，后面的值将覆盖前面的值
const map = new Map();
map.set(1, 'aaa').set(1, 'bbb');
map.get(1) // "bbb"
  
// 读取一个未知的键返回 undefined
new Map().get('asfddfsasadf') // undefined
  
// 只有对同一个对象的引用，Map 结构才将其视为同一个键
const map = new Map();
map.set(['a'], 555);
map.get(['a']) // undefined，表面是针对同一个键，但实际上这是两个值，内存地址是不一样的，而 Map 的键实际上是跟内存地址绑定的
  
// 同样的值的两个实例，在 Map 结构中被视为两个键
const map = new Map();
const k1 = ['a'];
const k2 = ['a'];
map.set(k1, 111).set(k2, 222);
map.get(k1) // 111
map.get(k2) // 222
  
// 只要两个值严格相等，Map 将其视为一个键，undefined 和 null 也是两个不同的键。虽然 NaN 不严格相等于自身，但 Map 将其视为同一个键
let map = new Map();
map.set(-0, 123);
map.get(+0) // 123
map.set(true, 1);
map.set('true', 2);
map.get(true) // 1
map.set(undefined, 3);
map.set(null, 4);
map.get(undefined) // 3
map.set(NaN, 123);
map.get(NaN) // 123
  
// size 属性返回 Map 结构的成员总数
const map = new Map();
map.set('foo', true);
map.set('bar', false);
map.size // 2
  
// set 方法设置键名 key 对应的键值为 value，然后返回整个 Map 结构。如果 key 已经有值，则键值会被更新，否则就新生成该键
const m = new Map();
m.set('edition', 6)        // 键是字符串
m.set(262, 'standard')     // 键是数值
m.set(undefined, 'nah')    // 键是 undefined
  
// set 方法返回的是当前的 Map 对象，因此可以采用链式写法
let map = new Map().set(1, 'a').set(2, 'b').set(3, 'c');
  
// get 方法读取 key 对应的键值，如果找不到 key，返回 undefined
const m = new Map();
const hello = function() {console.log('hello');};
m.set(hello, 'Hello ES6!') // 键是函数
m.get(hello)  // Hello ES6!
  
// has 方法返回一个布尔值，表示某个键是否在当前 Map 对象之中
const m = new Map();
m.set('edition', 6);
m.set(262, 'standard');
m.set(undefined, 'nah');
m.has('edition')     // true
m.has('years')       // false
m.has(262)           // true
m.has(undefined)     // true
  
// delete 方法删除某个键，返回 true。如果删除失败，返回 false
const m = new Map();
m.set(undefined, 'nah');
m.has(undefined)     // true
m.delete(undefined)
m.has(undefined)       // false
  
// clear 方法清除所有成员，没有返回值
let map = new Map();
map.set('foo', true);
map.set('bar', false);
map.size // 2
map.clear()
map.size // 0
  
// Map 结构原生提供的三个遍历器生成函数：keys、values、entries，注意 Map 的遍历顺序就是插入顺序
const map = new Map([
  ['F', 'no'],
  ['T',  'yes'],
]);
for (let key of map.keys()) {
  console.log(key); // "F" "T"
}
for (let value of map.values()) {
  console.log(value); // "no" "yes"
}
for (let item of map.entries()) {
  console.log(item[0], item[1]);
}
// "F" "no"
// "T" "yes"
for (let [key, value] of map.entries()) { // 或者
  console.log(key, value);
}
// "F" "no"
// "T" "yes"
for (let [key, value] of map) { // 等同于使用 map.entries()
  console.log(key, value);
}
// "F" "no"
// "T" "yes"
  
// Map 结构的默认遍历器接口（Symbol.iterator属性），就是 entries 方法
map[Symbol.iterator] === map.entries  // true
  
// Map 结构转为数组结构，比较快速的方法是使用扩展运算符（...）
const map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);
[...map.keys()] // [1, 2, 3]
[...map.values()] // ['one', 'two', 'three']
[...map.entries()]  // [[1,'one'], [2, 'two'], [3, 'three']]
[...map]  // [[1,'one'], [2, 'two'], [3, 'three']]
  
// 结合数组的 map 方法、filter 方法，可以实现 Map 的遍历和过滤（Map 本身没有 map 和 filter 方法）
const map0 = new Map().set(1, 'a').set(2, 'b').set(3, 'c');
const map1 = new Map(
  [...map0].filter(([k, v]) => k < 3) // 产生 Map 结构 {1 => 'a', 2 => 'b'}
);
const map2 = new Map(
  [...map0].map(([k, v]) => [k * 2, '_' + v]) // 产生 Map 结构 {2 => '_a', 4 => '_b', 6 => '_c'}
);
  
// Map 还有一个 forEach 方法，与数组的 forEach 方法类似，也可以实现遍历
map.forEach(function(value, key, map) {
  console.log("Key: %s, Value: %s", key, value);
});
  
// forEach 方法还可以接受第二个参数，用来绑定 this
const reporter = {
  report: function(key, value) {
    console.log("Key: %s, Value: %s", key, value);
  }
};
map.forEach(function(value, key, map) {
  this.report(key, value);
}, reporter);
  
// Map 转为数组，使用扩展运算符（...）
const myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
[...myMap]  // [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]
  
// 将数组传入 Map 构造函数，就可以转为 Map
new Map([
  [true, 7],
  [{foo: 3}, ['abc']]
])
// Map {
//   true => 7,
//   Object {foo: 3} => ['abc']
// }
  
// 如果所有 Map 的键都是字符串，它可以无损地转为对象；如果有非字符串的键名，那么这个键名会被转成字符串，再作为对象的键名
function strMapToObj(strMap) {
  let obj = Object.create(null);
  for (let [k,v] of strMap) {
    obj[k] = v;
  }
  return obj;
}
const myMap = new Map().set('yes', true).set('no', false);
strMapToObj(myMap)  // { yes: true, no: false }
  
// 对象转为 Map
function objToStrMap(obj) {
  let strMap = new Map();
  for (let k of Object.keys(obj)) {
    strMap.set(k, obj[k]);
  }
  return strMap;
}
objToStrMap({yes: true, no: false}) // Map {"yes" => true, "no" => false}
  
// Map 的键名都是字符串，这时可以选择转为对象 JSON
function strMapToJson(strMap) {
  return JSON.stringify(strMapToObj(strMap));
}
let myMap = new Map().set('yes', true).set('no', false);
strMapToJson(myMap) // '{"yes":true,"no":false}'
  
// Map 的键名有非字符串，这时可以选择转为数组 JSON
function mapToArrayJson(map) {
  return JSON.stringify([...map]);
}
let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
mapToArrayJson(myMap) // '[[true,7],[{"foo":3},["abc"]]]'
  
// JSON 转为 Map，正常情况下，所有键名都是字符串
function jsonToStrMap(jsonStr) {
  return objToStrMap(JSON.parse(jsonStr));
}
jsonToStrMap('{"yes": true, "no": false}')  // Map {'yes' => true, 'no' => false}
  
// 特殊情况整个 JSON 就是一个数组，且每个数组成员本身，又是一个有两个成员的数组，可以一一对应地转为 Map。这往往是 Map 转为数组 JSON 的逆操作
function jsonToMap(jsonStr) {
  return new Map(JSON.parse(jsonStr));
}
jsonToMap('[[true,7],[{"foo":3},["abc"]]]') // Map {true => 7, Object {foo: 3} => ['abc']}
```
### 四、WeakMap
```
// WeakMap 可以使用 set 方法添加成员
const wm1 = new WeakMap();
const key = {foo: 1};
wm1.set(key, 2);
wm1.get(key) // 2
  
// WeakMap 也可以接受一个数组，作为构造函数的参数
const k1 = [1, 2, 3];
const k2 = [4, 5, 6];
const wm2 = new WeakMap([[k1, 'foo'], [k2, 'bar']]);
wm2.get(k2) // "bar"
  
// WeakMap 只接受对象作为键名（null 除外）
const map = new WeakMap();
map.set(1, 2) // TypeError: 1 is not an object!
map.set(Symbol(), 2)  // TypeError: Invalid value used as weak map key
map.set(null, 2)  // TypeError: Invalid value used as weak map key
  
// 有时我们想在某个对象上面存放一些数据，但是这会形成对于这个对象的引用
const e1 = document.getElementById('foo');
const e2 = document.getElementById('bar');
const arr = [
  [e1, 'foo 元素'],
  [e2, 'bar 元素'],
]; 
arr [0] = null; // 不需要 e1 和 e2 的时候，必须手动删除引用
arr [1] = null;
  
// WeakMap 的键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内
const wm = new WeakMap();
const element = document.getElementById('example');
wm.set(element, 'some information');
wm.get(element) // "some information"，当该 DOM 元素被清除，其所对应的 WeakMap 记录就会自动被移除
  
// WeakMap 弱引用的只是键名，而不是键值。键值依然是正常引用
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};
wm.set(key, obj);
obj = null;
wm.get(key) // Object {foo: 1}
  
// WeakMap 没有遍历操作，也没有 size 属性，而且无法清空，即不支持 clear 方法。因此 WeakMap 只有四个方法可用：get()、set()、has()、delete()
const wm = new WeakMap();
wm.size // undefined
wm.forEach // undefined
wm.clear // undefined
  
// WeakMap 应用的典型场合就是 DOM 节点作为键名
let myElement = document.getElementById('logo');
let myWeakmap = new WeakMap();
myWeakmap.set(myElement, {timesClicked: 0});
myElement.addEventListener('click', function() {
  let logoData = myWeakmap.get(myElement);
  logoData.timesClicked++;
}, false);
  
// WeakMap 的另一个用处是部署私有属性
const _counter = new WeakMap();
const _action = new WeakMap();
class Countdown {
  constructor(counter, action) {
    _counter.set(this, counter);
    _action.set(this, action);
  }
  dec() {
    let counter = _counter.get(this);
    if (counter < 1) return;
    counter--;
    _counter.set(this, counter);
    if (counter === 0) {
      _action.get(this)();
    }
  }
}
const c = new Countdown(2, () => console.log('DONE'));
c.dec()
c.dec() // DONE
```
WeakMap 结构与 Map 结构类似，也是用于生成键值对的集合。不过 WeakMap 只接受对象作为键名（null 除外），不接受其他类型的值作为键名，而且 WeakMap 的键名所指向的对象，不计入垃圾回收机制。