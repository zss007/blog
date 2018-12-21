---
title: es6 之对象扩展
categories:
- es6
---
ES6 扩展了对象。
<!--more-->
### 一、属性的简洁表示法
ES6 允许直接写入变量和函数，作为对象的属性和方法。这样的书写更加简洁。
```
// 属性简写
const foo = 'bar';
const baz = {foo};
baz // {foo: "bar"}
const baz = {foo: foo}; // 等同于
    
// 方法简写
const o = {
  method() {
    return "Hello!";
  }
};
const o = {   // 等同于
  method: function() {
    return "Hello!";
  }
};
    
// CommonJS 模块输出一组变量非常合适使用简洁写法
let ms = {};
function getItem (key) {
  return key in ms ? ms[key] : null;
}
function setItem (key, value) {
  ms[key] = value;
}
function clear () {
  ms = {};
}
module.exports = { getItem, setItem, clear };
module.exports = { // 等同于
  getItem: getItem,
  setItem: setItem,
  clear: clear
};
    
// 简洁写法的属性名总是字符串，所以不会因为 class 属于关键字，而导致语法解析报错
const obj = {
  class () {}
};
var obj = {  // 等同于
  'class': function() {}
};
    
// 如果某个方法的值是一个 Generator 函数，前面需要加上星号
const obj = {
  * m() {
    yield 'hello world';
  }
};
```
### 二、属性名表达式
JavaScript 定义对象的属性，有两种方法。方法一是直接用标识符作为属性名，方法二是用表达式作为属性名，这时要将表达式放在方括号之内。
```
obj.foo = true;   // 方法一
obj['a' + 'bc'] = 123;    // 方法二
    
// 如果使用字面量方式定义对象（使用大括号），在 ES5 中只能使用方法一（标识符）定义属性
var obj = {
  foo: true,
  abc: 123
};
    
// ES6 允许字面量定义对象时，用方法二（表达式）作为对象的属性名
let lastWord = 'last word';
const a = {
  'first word': 'hello',
  [lastWord]: 'world'
};
a['first word'] // "hello"
a[lastWord] // "world"
a['last word'] // "world"
    
// 表达式还可以用于定义方法名
let obj = {
  ['h' + 'ello']() {
    return 'hi';
  }
};
obj.hello() // hi
    
// 属性名表达式与简洁表示法，不能同时使用，会报错
const foo = 'bar';
const bar = 'abc';
const baz = { [foo] };  // 报错
const baz = { [foo]: 'abc'};  // 正确
    
// 属性名表达式如果是一个对象，默认情况下会自动将对象转为字符串 [object Object]
const keyA = {a: 1};
const keyB = {b: 2};
const myObject = {
  [keyA]: 'valueA',
  [keyB]: 'valueB'
};
myObject // Object {[object Object]: "valueB"}，[keyA] 和 [keyB] 得到的都是 [object Object]，[keyB] 会把 [keyA] 覆盖掉
```
### 三、方法的 name 属性
函数的 name 属性，返回函数名。对象方法也是函数，因此也有 name 属性。
```
// 方法的 name 属性返回函数名（即方法名）
const person = {
  sayName() {
    console.log('hello!');
  },
};
person.sayName.name   // "sayName"
    
// 对象的方法使用了取值函数（getter）和存值函数（setter），则 name 属性不是在该方法上面，而是该方法的属性的描述对象的 get 和 set 属性上面，返回值是方法名前加上 get 和 set
const obj = {
  get foo() {},
  set foo(x) {}
};
obj.foo.name  // TypeError: Cannot read property 'name' of undefined
const descriptor = Object.getOwnPropertyDescriptor(obj, 'foo');
descriptor.get.name // "get foo"
descriptor.set.name // "set foo"
    
// Function 构造函数创造的函数，name 属性返回 anonymous
(new Function()).name // "anonymous"
    
// bind 方法创造的函数，name 属性返回 bound 加上原函数的名字
var doSomething = function() {
  // ...
};
doSomething.bind().name // "bound doSomething"
    
// 对象的方法是一个 Symbol 值，那么 name 属性返回的是 Symbol 值的描述
const key1 = Symbol('description');
const key2 = Symbol();
let obj = {
  [key1]() {},
  [key2]() {},
};
obj[key1].name // "[description]"
obj[key2].name // ""
```
### 四、Object.is()
ES5 比较两个值是否相等，只有两个运算符：相等运算符（==）和严格相等运算符（===）。它们都有缺点，前者会自动转换数据类型，后者的 NaN 不等于自身，以及 +0 等于 -0。JavaScript 缺乏一种运算，在所有环境中，只要两个值是一样的，它们就应该相等。   
ES6 提出 "Same-value equality"（同值相等）算法，用来解决这个问题。Object.is 就是部署这个算法的新方法。它用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致。不同之处只有两个：一是 +0 不等于 -0，二是 NaN 等于自身。
```
+0 === -0 //true
NaN === NaN // false
    
Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```
### 五、Object.assign()
Object.assign 方法用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target）。
```
// Object.assign 方法的第一个参数是目标对象，后面的参数都是源对象
const target = { a: 1 };
const source1 = { b: 2 };
const source2 = { c: 3 };
Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
    
// 如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。
const target = { a: 1, b: 1 };
const source1 = { b: 2, c: 2 };
const source2 = { c: 3 };
Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
    
// 如果只有一个参数，Object.assign 会直接返回该参数
const obj = {a: 1};
Object.assign(obj) === obj // true
    
// 如果该参数不是对象，则会先转成对象，然后返回
typeof Object.assign(2) // "object"
    
// 由于 undefined 和 null 无法转成对象，所以如果它们作为参数，就会报错
Object.assign(undefined) // 报错
Object.assign(null) // 报错
    
// 如果非对象参数出现在源对象的位置（即非首参数），这些参数都会转成对象，如果无法转成对象，就会跳过。这意味着，如果 undefined 和 null 不在首参数，就不会报错
let obj = {a: 1};
Object.assign(obj, undefined) === obj // true
Object.assign(obj, null) === obj // true
    
// 其他类型的值（即数值、字符串和布尔值）不在首参数，也不会报错。但是，除了字符串会以数组形式，拷贝入目标对象，其他值都不会产生效果
const v1 = 'abc';
const v2 = true;
const v3 = 10;
const obj = Object.assign({}, v1, v2, v3);
console.log(obj); // { "0": "a", "1": "b", "2": "c" }
    
// 上述结果是因为数值和布尔值都会被忽略，只有字符串的包装对象，会产生可枚举属性
Object(true) // {[[PrimitiveValue]]: true}
Object(10)  //  {[[PrimitiveValue]]: 10}
Object('abc') // {0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"}
    
// Object.assign只拷贝源对象的自身属性（不拷贝继承属性），也不拷贝不可枚举的属性（enumerable: false）
Object.assign({b: 'c'},
  Object.defineProperty({}, 'invisible', {
    enumerable: false,
    value: 'hello'
  })
)
// { b: 'c' }
    
// 属性名为 Symbol 值的属性，也会被 Object.assign 拷贝
Object.assign({ a: 'b' }, { [Symbol('c')]: 'd' })
// { a: 'b', Symbol(c): 'd' }
    
// Object.assign 方法实行的是浅拷贝，而不是深拷贝
const obj1 = {a: {b: 1}};
const obj2 = Object.assign({}, obj1);
obj1.a.b = 2;
obj2.a.b // 2
    
// 对于嵌套的对象，一旦遇到同名属性，Object.assign 的处理方法是替换
const target = { a: { b: 'c', d: 'e' } }
const source = { a: { b: 'hello' } }
Object.assign(target, source)
// { a: { b: 'hello' } }
    
// Object.assign 可以用来处理数组，但是会把数组视为属性名为 0、1、2... 的对象
Object.assign([1, 2, 3], [4, 5])
// [4, 5, 3]
    
// Object.assign 只能进行值的复制，如果要复制的值是一个取值函数，那么将求值后再复制
const source = {
  get foo() { return 1 }
};
const target = {};
Object.assign(target, source)
// { foo: 1 }

// 应用：为属性指定默认值（代码原意是将 url.port 改成 8000，url.host 不变，实际结果却是 options.url 覆盖掉 DEFAULTS.url）
const DEFAULTS = {
  url: {
    host: 'example.com',
    port: 7070
  },
};
function processContent(options) {
  options = Object.assign({}, DEFAULTS, options);
  console.log(options);
}
processContent({ url: {port: 8000} })
// {
//   url: {port: 8000}
// }
```
### 六、属性的可枚举性和遍历
对象的每个属性都有一个描述对象（Descriptor），用来控制该属性的行为。Object.getOwnPropertyDescriptor 方法可以获取该属性的描述对象。描述对象的 enumerable 属性，称为”可枚举性“，如果该属性为 false，就表示某些操作会忽略当前属性。目前，有四个操作会忽略 enumerable 为 false 的属性。    
1、for...in 循环：只遍历对象自身的和继承的可枚举的属性。    
2、Object.keys()：返回对象自身的所有可枚举的属性的键名。   
3、JSON.stringify()：只串行化对象自身的可枚举的属性。   
4、Object.assign()： 忽略 enumerable 为 false 的属性，只拷贝对象自身的可枚举的属性。    
实际上，引入"可枚举"（enumerable）这个概念的最初目的，就是让某些属性可以规避掉 for...in 操作，不然所有内部属性和方法都会被遍历到。比如，对象原型的 toString 方法，以及数组的 length 属性，就通过"可枚举性"，从而避免被 for...in 遍历到。   
ES6 一共有 5 种方法可以遍历对象的属性：   
1、for...in 循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）。    
2、Object.keys 返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名。   
3、Object.getOwnPropertyNames 返回一个数组，包含对象自身的所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名。   
4、Object.getOwnPropertySymbols 返回一个数组，包含对象自身的所有 Symbol 属性的键名。    
5、Reflect.ownKeys 返回一个数组，包含对象自身的所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。   
以上的 5 种方法遍历对象的键名，都遵守同样的属性遍历的次序规则：   
1、首先遍历所有数值键，按照数值升序排列。   
2、其次遍历所有字符串键，按照加入时间升序排列。    
3、最后遍历所有 Symbol 键，按照加入时间升序排列。   
```
// Object.getOwnPropertyDescriptor 方法获取该属性的描述对象
let obj = { foo: 123 };
Object.getOwnPropertyDescriptor(obj, 'foo')
//  {
//    value: 123,
//    writable: true,
//    enumerable: true,
//    configurable: true
//  }
    
// 规避 for...in 操作
Object.getOwnPropertyDescriptor(Object.prototype, 'toString').enumerable  // false
Object.getOwnPropertyDescriptor([], 'length').enumerable  // false
    
// ES6 规定，所有 Class 的原型的方法都是不可枚举的
Object.getOwnPropertyDescriptor(class {foo() {}}.prototype, 'foo').enumerable   // false
```
### 七、Object.getOwnPropertyDescriptors()
前面说过，Object.getOwnPropertyDescriptor 方法会返回某个对象属性的描述对象（descriptor）。ES2017 引入了 Object.getOwnPropertyDescriptors 方法，返回指定对象所有自身属性（非继承属性）的描述对象。
```
const obj = {
  foo: 123,
  get bar() { return 'abc' }
};
Object.getOwnPropertyDescriptors(obj)
// { foo:
//    { value: 123,
//      writable: true,
//      enumerable: true,
//      configurable: true },
//   bar:
//    { get: [Function: get bar],
//      set: undefined,
//      enumerable: true,
//      configurable: true } }
    
// Object.assign 方法总是拷贝一个属性的值，而不会拷贝它背后的赋值方法或取值方法
const source = {
  set foo(value) {
    console.log(value);
  }
};
const target1 = {};
Object.assign(target1, source);
Object.getOwnPropertyDescriptor(target1, 'foo')
// { value: undefined,
//   writable: true,
//   enumerable: true,
//   configurable: true }
    
// 解决 Object.assign() 无法正确拷贝 get 属性和 set 属性的问题
const source = {
  set foo(value) {
    console.log(value);
  }
};
const target2 = {};
Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
Object.getOwnPropertyDescriptor(target2, 'foo')
// { get: undefined,
//   set: [Function: set foo],
//   enumerable: true,
//   configurable: true }
    
// 两个对象合并的逻辑可以写成一个函数
const shallowMerge = (target, source) => Object.defineProperties(
  target,
  Object.getOwnPropertyDescriptors(source)
);
    
// 配合 Object.create 方法，将对象属性克隆到一个新对象（浅拷贝）
const clone = Object.create(Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj));
// 或者
const shallowClone = (obj) => Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);
    
// 继承另一个对象
const obj = {
  __proto__: prot,
  foo: 123,
};
    
// ES6 规定 __proto__ 只有浏览器要部署，其他环境需改写
const obj = Object.create(prot);
obj.foo = 123;
// 或者
const obj = Object.assign(
  Object.create(prot),
  {
    foo: 123,
  }
);
```
### 八、`__proto__` & Object.setPrototypeOf() & Object.getPrototypeOf()
JavaScript 语言的对象继承是通过原型链实现的。ES6 提供了更多原型对象的操作方法。
```
// __proto__ 由于浏览器广泛支持被加入了 ES6。只有浏览器必须部署这个属性，其他运行环境不一定需要部署，最好认为这个属性是不存在的
const obj = {
  method: function() { ... }
};
obj.__proto__ = someOtherObj;   // es6 的写法
var obj = Object.create(someOtherObj);    // es5 的写法
obj.method = function() { ... };
    
// 如果一个对象本身部署了 __proto__ 属性，该属性的值就是对象的原型
Object.getPrototypeOf({ __proto__: null })  // null
    
// Object.setPrototypeOf 方法的作用与 __proto__ 相同，用来设置一个对象的 prototype 对象
Object.setPrototypeOf(object, prototype)  // 格式
let proto = {};   // 案例
let obj = { x: 10 };
Object.setPrototypeOf(obj, proto);
proto.y = 20;
proto.z = 40;
obj.x // 10
obj.y // 20
obj.z // 40
    
// 如果第一个参数不是对象，会自动转为对象。但是由于返回的还是第一个参数，所以这个操作不会产生任何效果
Object.setPrototypeOf(1, {}) === 1 // true
Object.setPrototypeOf('foo', {}) === 'foo' // true
Object.setPrototypeOf(true, {}) === true // true
    
// 由于 undefined 和 null 无法转为对象，所以如果第一个参数是 undefined 或 null，就会报错
Object.setPrototypeOf(undefined, {}) // TypeError: Object.setPrototypeOf called on null or undefined
Object.setPrototypeOf(null, {}) // // TypeError: Object.setPrototypeOf called on null or undefined
    
// Object.getPrototypeOf 方法与 Object.setPrototypeOf 方法配套，用于读取一个对象的原型对象
function Rectangle() {
  // ...
}
const rec = new Rectangle();
Object.getPrototypeOf(rec) === Rectangle.prototype  // true
Object.setPrototypeOf(rec, Object.prototype);
Object.getPrototypeOf(rec) === Rectangle.prototype  // false
    
// 如果参数不是对象，会被自动转为对象
// 等同于 Object.getPrototypeOf(Number(1))
Object.getPrototypeOf(1)  // Number {[[PrimitiveValue]]: 0}
// 等同于 Object.getPrototypeOf(String('foo'))
Object.getPrototypeOf('foo')  // String {length: 0, [[PrimitiveValue]]: ""}
// 等同于 Object.getPrototypeOf(Boolean(true))
Object.getPrototypeOf(true) // Boolean {[[PrimitiveValue]]: false}
Object.getPrototypeOf(1) === Number.prototype // true
Object.getPrototypeOf('foo') === String.prototype // true
Object.getPrototypeOf(true) === Boolean.prototype // true
    
// 如果参数是 undefined 或 null，它们无法转为对象，所以会报错
Object.getPrototypeOf(null) // TypeError: Cannot convert undefined or null to object
Object.getPrototypeOf(undefined)  // TypeError: Cannot convert undefined or null to object
```
### 九、super 关键字
this 关键字总是指向函数所在的当前对象，ES6 又新增了另一个类似的关键字super，指向当前对象的原型对象。
```
const proto = {
  foo: 'hello'
};
const obj = {
  find() {
    return super.foo;
  }
};
Object.setPrototypeOf(obj, proto);
obj.find() // "hello"
    
// super 关键字表示原型对象时，只能用在对象的方法（简写法）之中，用在其他地方都会报错
const obj = {
  foo: super.foo  // 报错，用在属性里面
}
const obj = {
  foo: () => super.foo  // 报错，没有用简写法
}
const obj = {
  foo: function () {
    return super.foo  // 报错，没有用简写法
  }
}
    
// super.foo 等同于 Object.getPrototypeOf(this).foo（属性）或 Object.getPrototypeOf(this).foo.call(this)（方法）
const proto = {
  x: 'hello',
  foo() {
    console.log(this.x);
  },
};
const obj = {
  x: 'world',
  foo() {
    super.foo();
  }
}
Object.setPrototypeOf(obj, proto);    // 绑定的this 还是当前对象 obj
obj.foo() // "world"
```
### 十、Object.keys() & Object.values() & Object.entries()
ES5 引入了 Object.keys 方法，返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键名。    
ES2017 引入了跟 Object.keys 配套的 Object.values 和 Object.entries，作为遍历一个对象的补充手段，供 for...of 循环使用。
```
let {keys, values, entries} = Object;
let obj = { a: 1, b: 2, c: 3 };

for (let key of keys(obj)) {
  console.log(key); // 'a', 'b', 'c'
}

for (let value of values(obj)) {
  console.log(value); // 1, 2, 3
}

for (let [key, value] of entries(obj)) {
  console.log([key, value]); // ['a', 1], ['b', 2], ['c', 3]
}

// Object.values 会过滤属性名为 Symbol 值的属性
Object.values({ [Symbol()]: 123, foo: 'abc' }); // ['abc']
    
// 如果 Object.values 方法的参数是一个字符串，会返回各个字符组成的一个数组
Object.values('foo')  // ['f', 'o', 'o']
    
// 如果参数不是对象，Object.values 会先将其转为对象。由于数值和布尔值的包装对象，都不会为实例添加非继承的属性。所以，Object.values 会返回空数组
Object.values(42) // []
Object.values(true) // []
    
// Object.entries 方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值对数组
const obj = { foo: 'bar', baz: 42 };
Object.entries(obj) // [ ["foo", "bar"], ["baz", 42] ]
    
// 如果原对象的属性名是一个 Symbol 值，该属性会被忽略
Object.entries({ [Symbol()]: 123, foo: 'abc' });  // [ [ 'foo', 'abc' ] ]
    
// 应用：将对象转为真正的 Map 结构
const obj = { foo: 'bar', baz: 42 };
const map = new Map(Object.entries(obj));
map // Map { foo: "bar", baz: 42 }
```
### 十一、对象的扩展运算符
```
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
x // 1
y // 2
z // { a: 3, b: 4 }
    
// 如果等号右边是 undefined 或 null，就会报错，因为它们无法转为对象
let { x, y, ...z } = null; // 运行时错误
let { x, y, ...z } = undefined; // 运行时错误
    
// 解构赋值必须是最后一个参数，否则会报错
let { ...x, y, z } = obj; // 句法错误
let { x, ...y, ...z } = obj; // 句法错误
    
// 解构赋值的拷贝是浅拷贝
let obj = { a: { b: 1 } };
let { ...x } = obj;
obj.a.b = 2;
x.a.b // 2
    
// 扩展运算符的解构赋值，不能复制继承来自原型对象的属性
let o1 = { a: 1 };
let o2 = { b: 2 };
o2.__proto__ = o1;
let { ...o3 } = o2;
o3 // { b: 2 }
o3.a // undefined
    
// 另一个例子
const o = Object.create({ x: 1, y: 2 });
o.z = 3;
let { x, ...{ y, z } } = o;
x // 1
y // undefined
z // 3
    
// 对象的扩展运算符（...）用于取出参数对象的所有可遍历属性，拷贝到当前对象之中，等同于使用 Object.assign 方法
let z = { a: 3, b: 4 };
let n = { ...z };
n // { a: 3, b: 4 }
    
// 完整克隆一个对象，同时拷贝对象原型的属性
const clone1 = {  // 写法一
  __proto__: Object.getPrototypeOf(obj),
  ...obj
};
const clone2 = Object.assign(   // 写法二
  Object.create(Object.getPrototypeOf(obj)),
  obj
);
const clone3 = Object.create(   // 写法三
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
)
    
// 扩展运算符可以用于合并两个对象
let ab = { ...a, ...b };
let ab = Object.assign({}, a, b); // 等同于
    
// 用户自定义的属性，放在扩展运算符后面，则扩展运算符内部的同名属性会被覆盖掉（注意这不是解构赋值，否则必须是最后一个参数）
let aWithOverrides = { ...a, x: 1, y: 2 };
let aWithOverrides = { ...a, ...{ x: 1, y: 2 } }; // 等同于
let x = 1, y = 2, aWithOverrides = { ...a, x, y };  // 等同于
let aWithOverrides = Object.assign({}, a, { x: 1, y: 2 });  // 等同于
    
// 修改现有对象部分的属性就很方便
let newVersion = {
  ...previousVersion,
  name: 'New Name' // Override the name property
};
    
// 如果把自定义属性放在扩展运算符前面，就变成了设置新对象的默认属性值
let aWithDefaults = { x: 1, y: 2, ...a };
let aWithDefaults = Object.assign({}, { x: 1, y: 2 }, a); // 等同于
let aWithDefaults = Object.assign({ x: 1, y: 2 }, a); // 等同于
    
// 与数组的扩展运算符一样，对象的扩展运算符后面可以跟表达式
const obj = {
  ...(x > 1 ? {a: 1} : {}),
  b: 2,
};
    
// 如果扩展运算符后面是一个空对象，则没有任何效果
{...{}, a: 1}
// { a: 1 }
    
// 如果扩展运算符的参数是 null 或 undefined，这两个值会被忽略，不会报错
let emptyObject = { ...null, ...undefined }; // 不报错

// 并不会抛出错误，因为 x 属性只是被定义，但没执行
let aWithXGetter = {
  ...a,
  get x() {
    throw new Error('not throw yet');
  }
};

// 会抛出错误，扩展运算符的参数对象之中，如果有取值函数get，这个函数是会执行的，所以 x 属性被执行了
let runtimeError = {
  ...a,
  ...{
    get x() {
      throw new Error('throw now');
    }
  }
};
```
对象的解构赋值用于从一个对象取值，相当于将目标对象自身的所有可遍历的（enumerable）、但尚未被读取的属性，分配到指定的对象上面。所有的键和它们的值，都会拷贝到新对象上面。