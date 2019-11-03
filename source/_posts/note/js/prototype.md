---
title: js 之原型链
categories:
- note
---
JavaScript 的原型有好好的研究过，现在重新捡起来整理下。
<!--more-->
### 一、创建对象
##### 1.1、对面字向量 | new Object
```
var o1 = {name:'o1'}
var o11 = new Object({name:'o11'})
```
##### 1.2、new 创建
```
var M = function(){this.name='o2'}
var o2 = new M()
```
##### 1.3、Object.create()
```
var P = {name:'o3'}
var o3 = Object.create(P)
```
### 二、__proto__
JS 的原始数据类型有六种：undefined、null、boolean、string、number、Symbol。引用类型通常叫做类，常见有：array、object 等。引用类型 object 的每个实例称之为对象，每个对象都拥有一个原型对象，而指向该原型对象的内部指针则`__proto__`，通过它可以从中继承原型对象的属性，原型是 JavaScript 中的基因链接，有了这个，才能知道这个对象的祖祖辈辈。从对象中的`__proto__`可以访问到他所继承的原型对象。
```
var a = new Array();
console.log('prototype', a.__proto__ === Array.prototype); // true
```
Array.prototype 本身也是一个对象，也有继承的原型：
```
a.__proto__.__proto__ === Object.prototype  // true
// 等同于 
Array.prototype.__proto__ === Object.prototype  // true
```
这就说明，array 本身也是继承自 object，而 object 的原型则是指向原始类型 null。
```
a.__proto__.__proto__.__proto__ === null  // true
// 等同于 
Object.prototype.__proto__ === null  // true
```
<img src="/assets/js/proto1.png">
除了使用`__proto__`方法访问对象的原型，还可以通过 Object.getPrototypeOf 方法来获取对象的原型，以及通过
Object.setPrototypeOf 方法来重写对象的原型。值得注意的是，按照语言的标准，`__proto__`属性只有浏览器才能部署，其他环境可以没有这个属性，而且前后的两根下划线表示它本地是一个内部属性，不应该对使用者暴露。因此，应该尽量少用这个属性，而是用 Object.getPrototypeOf 和 Object.setPrototypeOf，进行原型对象的读写操作。这里用`__proto__`属性来描述对象中的原型，是因为这样来的更加形象，而且容易理解。
### 三、prototype
函数作为 JavaScript 中的一等公民，它既是函数又是对象，函数的原型指向 Function.prototype。
```
var Foo = function() {}
Foo.__proto__ === Function.prototype // true
```
函数实例除了拥有`__proto__`属性之外，还拥有 prototype 属性。通过该函数构造的实例对象，其原型指针`__proto__`
会指向该函数的 prototype 属性。
```
var a = new Foo();
a.__proto__ === Foo.prototype; // true
```
而函数的 prototype 属性，本身是一个由 object 构造的实例对象。
```
Foo.prototype.__proto__ === Object.prototype; // true
// prototype 属性很特殊，它还有一个隐式的 constructor，指向了构造函数本身。
Foo.prototype.constructor === Foo; // true
a.constructor === Foo; // true
a.constructor === Foo.prototype.constructor; // true
```
<img src="/assets/js/proto2.png">
### 四、原型链
原型链作为实现继承的主要方法，其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。假如我们让原型对象等于另一个类型的实例，此时的原型对象将包含一个指向另一个原型的指针，如此就构造了原型链的基本概念。
“原型链”的作用在于，当读取对象的某个属性时，JavaScript 引擎先寻找对象本身的属性，如果找不到，就到它的原型去找，如果还是找不到，就到原型的原型去找。以此类推，如果直到最顶层的 Object.prototype 还是找不到，则返回 undefined。
### 五、判断方法
原始数据类型一般使用 typeof 来判断（两种情况下回返回 undefined：1、变量没有声明；2、变量为 undefined）。
- typeof null 返回 object，其他引用类型均返回 object。
- instanceof 判断是否由某个构造函数创建，返回 boolean 类型值（只要处在原型链上就返回 true，可使用 `__proto__.constructor`准确返回构造函数）。
- Object.isPrototypeOf() 只要某个对象处在原型链上，isProtypeOf 都返回 true

```
var Bar = function() {}
var b = new Bar();
b instanceof Bar // true
b instanceof Object // true
b.__proto__.constructor // ƒ Bar() {}
Bar.prototype.isPrototypeOf(b) // true
Object.prototype.isPrototypeOf(Bar) // true
```
要注意，实例 b 的原型是 Bar.prototype 而不是 Bar。
<img src="/assets/js/proto3.png">
从上图中，能看到一个有趣的地方。`Function.prototype.__proto__`指向了 Object.prototype，这说明Function.prototype 是一个 Object 实例，那么应当是先有的 Object 再有 Function。但是`Object.prototype.constructor.__proto__`又指向了 Function.prototype。这样看来，没有 Function，Object 也不能创建实例。 这就产生了一种类「先有鸡还是先有蛋」的经典问题，到底是先有的 Object 还是先有的 Function 呢？ 
### 六、Object.create
```
function Point(){};
var Circle = Object.create(Point);
console.log(Circle.__proto__ === Point); // true
console.log(Circle.__proto__ === Point.prototype); // false
```
使用指定的原型对象和其属性创建了一个新的对象，在例子中实例 Circle 的原型指向 Point。
### 七、new 运算符
使用 new 运算符创建对象过程如下：
- 7.1、一个新对象被创建，他继承自 foo.prototype
- 7.2、构造函数 foo 被执行，执行的时候相应的传参会被传入，同时上下文（this）会被指定为这个新实例，new foo 等同于 new foo()，只能用在不传递任何参数的情况
- 7.3、如果构造函数返回了一个“对象”，那么这个对象会取代整个 new 出来的结果，如果构造函数没有返回对象，那么 new 出来的结果是步骤 7.1 创建的对象

```
var new2 = function (func) {
  var o = Object.create(func.prototype)
  var k = func.call(o)
  if (typeof k === 'object') {
    return k
  } else {
    return o
  }
}
```