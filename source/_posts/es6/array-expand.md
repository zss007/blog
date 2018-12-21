---
title: es6 之数组扩展
categories:
- es6
---
ES6 扩展了数组。
<!--more-->
### 一、扩展运算符
扩展运算符（spread）是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。
```
console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5
    
[...document.querySelectorAll('div')]
// [<div>, <div>, <div>]
    
// 主要用于函数调用
function push(array, ...items) {
  array.push(...items);
}
    
// 扩展运算符与正常的函数参数可以结合使用，非常灵活
function f(v, w, x, y, z) { }
const args = [0, 1];
f(-1, ...args, 2, ...[3]);
    
// 扩展运算符后面还可以放置表达式
const arr = [
  ...(x > 0 ? ['a'] : []),
  'b',
];
    
// 扩展运算符后面是一个空数组，则不产生任何效果
[...[], 1]
// [1]
    
// 替代函数的 apply 方法
Math.max.apply(null, [14, 3, 77])  // ES5 的写法
Math.max(...[14, 3, 77]) // ES6 的写法
Math.max(14, 3, 77);  // 等同于
    
// bind 用于绑定参数，apply 使 bind 内部的 this 绑定 Date
new (Date.bind.apply(Date, [null, 2015, 1, 1])) // ES5
new Date(...[2015, 1, 1]); // ES6
    
// 扩展运算符复制数组
const a1 = [1, 2];
const a2 = a1;
a2[0] = 2;
// 数组是复合的数据类型，直接复制的话，只是复制了指向底层数据结构的指针，而不是克隆一个全新的数组
a1 // [2, 2]
// ES5 只能用变通方法来复制数组
const a2 = a1.concat();
// 扩展运算符提供了复制数组的简便写法
const a2 = [...a1];
const [...a2] = a1;
    
// 扩展运算符提供了数组合并的新写法
var arr1 = ['a', 'b'];
var arr2 = ['c'];
var arr3 = ['d', 'e'];
arr1.concat(arr2, arr3);  // ES5 的合并数组，[ 'a', 'b', 'c', 'd', 'e' ]
[...arr1, ...arr2, ...arr3]   // ES6 的合并数组，[ 'a', 'b', 'c', 'd', 'e' ]
    
// 扩展运算符与解构赋值结合
a = list[0], rest = list.slice(1)  // ES5
[a, ...rest] = list   // ES6
const [first, ...rest] = [];
first // undefined
rest  // []
const [first, ...rest] = ["foo"];
first  // "foo"
rest   // []
    
// 如果将扩展运算符用于数组赋值，只能放在参数的最后一位，否则会报错
const [...butLast, last] = [1, 2, 3, 4, 5];   // 报错
const [first, ...middle, last] = [1, 2, 3, 4, 5];   // 报错
    
// 扩展运算符将字符串转为真正的数组
[...'hello']
// [ "h", "e", "l", "l", "o" ]
    
// 扩展运算符能够正确识别四个字节的 Unicode 字符
'x\uD83D\uDE80y'.length // 4
[...'x\uD83D\uDE80y'].length // 3
    
// 用扩展运算符将实现了 Iterator 接口的对象转为真正的数组
let nodeList = document.querySelectorAll('div');
let array = [...nodeList];
    
// 没有部署 Iterator 接口的类似数组的对象，扩展运算符就无法将其转为真正的数组
let arrayLike = {
  '0': 'a',
  '1': 'b',
  '2': 'c',
  length: 3
};
// TypeError: Cannot spread non-iterable object.
let arr = [...arrayLike];
    
// 用扩展运算符遍历 Map、Set 结构
let map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);
let arr = [...map.keys()]; // [1, 2, 3]
    
// 用扩展运算符遍历 Generator 函数
const go = function*(){
  yield 1;
  yield 2;
  yield 3;
};
[...go()] // [1, 2, 3]
    
// 没有 Iterator 接口的对象，使用扩展运算符，将会报错
const obj = {a: 1, b: 2};
let arr = [...obj]; // TypeError: Cannot spread non-iterable object
```
### 二、Array.from()
Array.from 方法用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）
```
let arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};
var arr1 = [].slice.call(arrayLike); // // ES5 的写法，['a', 'b', 'c']
let arr2 = Array.from(arrayLike); // ES6 的写法，['a', 'b', 'c']
    
// 只要是部署了 Iterator 接口的数据结构，Array.from 都能将其转为数组
Array.from('hello')   // ['h', 'e', 'l', 'l', 'o']
let namesSet = new Set(['a', 'b'])
Array.from(namesSet)  // ['a', 'b']
    
// 参数是一个真正的数组，Array.from 会返回一个一模一样的新数组
Array.from([1, 2, 3])
// [1, 2, 3]
    
// Array.from 方法还支持转换类似数组的对象，即必须有 length 属性
Array.from({ length: 3 });
// [ undefined, undefined, undefined ]
    
// Array.from 还可以接受第二个参数，作用类似于数组的 map 方法，用来对每个元素进行处理，将处理后的值放入返回的数组
Array.from(arrayLike, x => x * x);
Array.from(arrayLike).map(x => x * x);  // 等同于
Array.from([1, 2, 3], (x) => x * x)
// [1, 4, 9]
    
// Array.from 可以将字符串转为数组，然后返回字符串的长度，可以避免 JavaScript 将大于 \uFFFF 的 Unicode 字符
function countSymbols(string) {
  return Array.from(string).length;
}
```
### 三、Array.of()
Array.of 方法用于将一组值，转换为数组，用于弥补数组构造函数 Array() 的不足。因为参数个数的不同，会导致 Array() 的行为有差异。
```
Array() // []
// 参数个数只有一个时，实际上是指定数组的长度
Array(3) // [, , ,]
// 只有当参数个数不少于 2 个时，Array()才会返回由参数组成的新数组
Array(3, 11, 8) // [3, 11, 8]
    
// Array.of 基本上可以用来替代 Array() 或 new Array()，并且不存在由于参数不同而导致的重载。它的行为非常统一
Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]
```
### 四、copyWithin()
数组实例的 copyWithin 方法，在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。也就是说，使用这个方法，会修改当前数组。
```
Array.prototype.copyWithin(target, start = 0, end = this.length)
// target（必需）：从该位置开始替换数据。如果为负值，表示倒数。
// start（可选）：从该位置开始读取数据，默认为 0。如果为负值，表示倒数。
// end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示倒数。
    
// 从 3 号位直到数组结束的成员（4 和 5），复制到从 0 号位开始的位置，结果覆盖了原来的 1 和 2
[1, 2, 3, 4, 5].copyWithin(0, 3)
// [4, 5, 3, 4, 5]
    
// 将 3 号位复制到 0 号位
[1, 2, 3, 4, 5].copyWithin(0, 3, 4)
// [4, 2, 3, 4, 5]
    
// -2 相当于 3 号位，-1 相当于 4 号位
[1, 2, 3, 4, 5].copyWithin(0, -2, -1)
// [4, 2, 3, 4, 5]
    
// 将 3 号位复制到 0 号位
[].copyWithin.call({length: 5, 3: 1}, 0, 3)
// {0: 1, 3: 1, length: 5}
    
// 将 2 号位到数组结束，复制到 0 号位
let i32a = new Int32Array([1, 2, 3, 4, 5]);
i32a.copyWithin(0, 2);
// Int32Array [3, 4, 5, 4, 5]
```
### 五、find() & findIndex()
数组实例的 find 方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为 true 的成员，然后返回该成员。如果没有符合条件的成员，则返回 undefined。    
数组实例的 findIndex 方法的用法与 find 方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回 -1。
```
// 接受三个参数，依次为当前的值、当前的位置和原数组
[1, 5, 10, 15].find(function(value, index, arr) {
  return value > 9;
}) // 10
    
// 接受三个参数，依次为当前的值、当前的位置和原数组
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
    
// 这两个方法都可以接受第二个参数，用来绑定回调函数的 this 对象
function f(v){
  return v > this.age;
}
let person = {name: 'John', age: 20};
[10, 12, 26, 15].find(f, person);    // 26
    
// 这两个方法都可以发现 NaN，弥补了数组的 indexOf 方法的不足（findIndex 方法可以借助 Object.is 方法做到）
[NaN].indexOf(NaN)  // -1
[NaN].findIndex(y => Object.is(NaN, y)) // 0
```
### 六、fill()
fill方法使用给定值，填充一个数组。
```
// 数组中已有的元素，会被全部抹去
['a', 'b', 'c'].fill(7)
// [7, 7, 7]
    
// fill 方法用于空数组的初始化非常方便
new Array(3).fill(7)
// [7, 7, 7]
    
// fill 方法还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置
['a', 'b', 'c'].fill(7, 1, 2)
// ['a', 7, 'c']
    
// 如果填充的类型为对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象
let arr = new Array(3).fill({name: "Mike"});
arr[0].name = "Ben";
arr
// [{name: "Ben"}, {name: "Ben"}, {name: "Ben"}]
let arr = new Array(3).fill([]);
arr[0].push(5);
arr
// [[5], [5], [5]]
```
### 七、entries()、keys() & values()
ES6 提供三个新的方法——entries()，keys() 和 values()——用于遍历数组，它们都返回一个遍历器对象可以用 for...of 循环进行遍历。
```
// keys() 是对键名的遍历
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1
    
// values() 是对键值的遍历
for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'
    
// entries() 是对键值对的遍历
for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```
### 八、includes()
Array.prototype.includes 方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的 includes 方法类似。ES2016 引入了该方法。
```
[1, 2, 3].includes(2)     // true
[1, 2, 3].includes(4)     // false
[1, 2, NaN].includes(NaN) // true
    
// 第二个参数表示搜索的起始位置，默认为 0
[1, 2, 3].includes(3, 3);  // false
// 如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为 -4，但数组长度为 3），则会重置为从 0 开始
[1, 2, 3].includes(3, -1); // true

// indexOf 不够语义化，需要去比较是否不等于 -1 而且内部使用严格相等运算符（===）进行判断，这会导致对 NaN 的误判
[NaN].indexOf(NaN)  // -1
// includes 使用的是不一样的判断算法，就没有这个问题
[NaN].includes(NaN)
```
另外，Map 和 Set 数据结构有一个 has 方法，需要注意与 includes 区分。    
Map 结构的 has 方法，是用来查找键名的，比如 Map.prototype.has(key)、WeakMap.prototype.has(key)、Reflect.has(target, propertyKey)。    
Set 结构的 has 方法，是用来查找值的，比如 Set.prototype.has(value)、WeakSet.prototype.has(value)。
### 九、数组的空位
数组的空位指，数组的某一个位置没有任何值。比如，Array 构造函数返回的数组都是空位。
```
Array(3) // [, , ,]
    
// 空位不是 undefined，一个位置的值等于 undefined，依然是有值的。空位是没有任何值，in 运算符可以说明这一点
0 in [undefined, undefined, undefined] // true
0 in [, , ,] // false
    
// forEach(), filter(), reduce(), every() 和 some() 都会跳过空位
[,'a'].forEach((x,i) => console.log(i)); // 1
['a',,'b'].filter(x => true) // ['a','b']
[,'a'].every(x => x==='a') // true
[1,,2].reduce((x,y) => return x+y) // 3
[,'a'].some(x => x !== 'a') // false
    
// map 会跳过空位，但会保留这个值
[,'a'].map(x => 1) // [,1]
    
// join() 和 toString() 会将空位视为 undefined，而 undefined 和 null 会被处理成空字符串
[,'a',undefined,null].join('#') // "#a##"
[,'a',undefined,null].toString() // ",a,,"
    
// Array.from 会将空位转为 undefined
Array.from(['a',,'b'])  // [ "a", undefined, "b" ]
    
// 扩展运算符会将空位转为 undefined
[...['a',,'b']] // [ "a", undefined, "b" ]
    
// copyWithin 会连空位一起拷贝
[,'a','b',,].copyWithin(2,0) // [,"a",,"a"]
    
// fill 会将空位视为正常的数组位置
new Array(3).fill('a') // ["a","a","a"]
    
// for...of 循环会遍历空位
let arr = [, ,];
for (let i of arr) {
  console.log(1);
}
// 1
// 1
    
// entries()、keys()、values()、find() 和 findIndex() 会将空位处理成 undefined
[...[,'a'].entries()] // [[0,undefined], [1,"a"]]
[...[,'a'].keys()] // [0,1]
[...[,'a'].values()] // [undefined,"a"]
[,'a'].find(x => true) // undefined
[,'a'].findIndex(x => true) // 0
```
由于空位的处理规则非常不统一，所以建议避免出现空位。