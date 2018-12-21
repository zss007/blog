---
title: es6 之 class
categories:
- es6
---
JavaScript 语言中，生成实例对象的传统方法是通过构造函数。这种写法跟传统的面向对象语言（比如 C++ 和 Java）差异很大，很容易让新学习这门语言的程序员感到困惑。ES6 提供了更接近传统语言的写法，引入了 Class（类）这个概念，作为对象的模板，通过 class 关键字可以定义类。基本上，ES6 的 class 可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的 class 写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。
<!--more-->
### 一、简介
```
// 传统方法
function Point(x, y) {
  this.x = x;
  this.y = y;
}
Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
};
var p = new Point(1, 2);
  
// ES6 类
class Point {
  constructor(x, y) { // ES5 的构造函数 Point，对应 ES6 的 Point 类的构造方法
    this.x = x; // this 关键字则代表实例对象
    this.y = y;
  }
  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
  
// ES6 的类完全可以看作构造函数的另一种写法
typeof Point // "function"，类的数据类型就是函数
Point === Point.prototype.constructor // true，类本身就指向构造函数
  
// 使用的时候，也是直接对类使用 new 命令，跟构造函数的用法完全一致
class Bar {
  doStuff() {
    console.log('stuff');
  }
}
var b = new Bar();
b.doStuff() // "stuff"
  
// 类的所有方法都定义在类的 prototype 属性上面
class Point {
  constructor() { // 构造函数的 prototype 属性，在 ES6 的 "类" 上面继续存在
    // ...
  }
  toString() {
    // ...
  }
  toValue() {
    // ...
  }
}
Point.prototype = { // 等同于
  constructor() {},
  toString() {},
  toValue() {},
};
  
// 在类的实例上面调用方法，其实就是调用原型上的方法
class B {}
let b = new B();
b.constructor === B.prototype.constructor // true
  
// 类的方法都定义在 prototype 对象上面，所以类的新方法可以添加在 prototype 对象上面
class Point {
  constructor(){
    // ...
  }
}
Object.assign(Point.prototype, {  // Object.assign 方法可以很方便地一次向类添加多个方法
  toString(){},
  toValue(){}
});
  
// prototype 对象的 constructor 属性，直接指向 "类" 的本身，这与 ES5 的行为是一致的
Point.prototype.constructor === Point // true
  
// 类的内部所有定义的方法都是不可枚举的
class Point {
  constructor(x, y) {
    // ...
  }
  toString() {
    // ...
  }
}
Object.keys(Point.prototype)  // []
Object.getOwnPropertyNames(Point.prototype) // ["constructor","toString"]
  
// 采用 ES5 的写法，toString 方法就是可枚举的
var Point = function (x, y) {
  // ...
};
Point.prototype.toString = function() {
  // ...
};
Object.keys(Point.prototype)  // ["toString"]
Object.getOwnPropertyNames(Point.prototype) // ["constructor","toString"]
  
// 类的属性名，可以采用表达式
let methodName = 'getArea';
class Square {
  constructor(length) {
    // ...
  }
  [methodName]() {
    // ...
  }
}
  
// 类和模块的内部，默认就是严格模式，所以不需要使用 use strict 指定运行模式。只要你的代码写在类或模块之中，就只有严格模式可用。
// 考虑到未来所有的代码，其实都是运行在模块之中，所以 ES6 实际上把整个语言升级到了严格模式。
```
### 二、constructor 方法
constructor 方法是类的默认方法，通过 new 命令生成对象实例时，自动调用该方法。一个类必须有 constructor 方法，如果没有显式定义，一个空的 constructor 方法会被默认添加。
```
// 默认添加空的 constructor 方法
class Point {
}
class Point { // 等同于
  constructor() {}
}
  
// constructor 方法默认返回实例对象（即 this），完全可以指定返回另外一个对象
class Foo {
  constructor() {
    return Object.create(null);
  }
}
new Foo() instanceof Foo  // false，返回一个全新的对象，结果导致实例对象不是 Foo 类的实例
  
// 类必须使用 new 调用，否则会报错。这是它跟普通构造函数的一个主要区别，后者不用 new 也可以执行
class Foo {
  constructor() {
    return Object.create(null);
  }
}

Foo() // TypeError: Class constructor Foo cannot be invoked without 'new'
```
### 三、类的实例对象
生成类的实例对象的写法，与 ES5 完全一样，也是使用 new 命令。前面说过，如果忘记加上 new，像函数那样调用 Class，将会报错。
```
// 类必须使用 new 调用
class Point {
  // ...
}
var point = Point(2, 3);  // 报错
var point = new Point(2, 3);  // 正确
  
// 与 ES5 一样，实例的属性除非显式定义在其本身（即定义在 this 对象上），否则都是定义在原型上（即定义在 class 上）
class Point { //定义类
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
var point = new Point(2, 3);
point.toString() // (2, 3)
point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
  
// 与 ES5 一样，类的所有实例共享一个原型对象
var p1 = new Point(2,3);
var p2 = new Point(3,2);
p1.__proto__ === p2.__proto__; //true
  
// 可以通过实例的 __proto__ 属性为 "类"添加方法（__proto__ 各大厂商具体实现时添加的私有属性，并不是语言本身的特性，推荐使用 Object.getPrototypeOf）
var p1 = new Point(2,3);
var p2 = new Point(3,2);
p1.__proto__.printName = function () { return 'Oops' };
p1.printName() // "Oops"
p2.printName() // "Oops"
var p3 = new Point(4,2);
p3.printName() // "Oops"
```
### 四、Class 表达式 & 不存在变量提升
```
// 与函数一样，类也可以使用表达式的形式定义
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};
  
// 这个类的名字是 MyClass 而不是 Me，Me 只在 Class 的内部代码可用，指代当前类
let inst = new MyClass();
inst.getClassName() // Me
Me.name // ReferenceError: Me is not defined
  
// 如果类的内部没用到的话，可以省略 Me，也就是可以写成下面的形式
const MyClass = class { /* ... */ };
  
// 采用 Class 表达式，可以写出立即执行的 Class
let person = new class {  // person 是一个立即执行的类的实例
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
}('张三');
person.sayName(); // "张三"
  
// 类不存在变量提升（hoist），这一点与 ES5 完全不同
new Foo(); // ReferenceError
class Foo {}
  
// 这种规定的原因与继承有关，必须保证子类在父类之后定义
{
  let Foo = class {};
  class Bar extends Foo { // 如果存在 class 的提升，上面代码就会报错，因为 class 会被提升到代码头部，而 let 命令是不提升的，所以导致 Bar 继承 Foo 的时候，Foo 还没有定义
  }
}
```
### 五、私有方法和私有属性
私有方法是常见需求，但 ES6 不提供，只能通过变通方法模拟实现。与私有方法一样，ES6 不支持私有属性。目前，有一个提案，为 class 加了私有属性。方法是在属性名之前，使用 # 表示。
```
// 在命名上加以区别
class Widget {
  foo (baz) { // 公有方法
    this._bar(baz);
  }
  _bar(baz) { // 私有方法，前面的下划线表示这是一个只限于内部使用的私有方法，但是不保险，外部还是可以调用到这个方法
    return this.snaf = baz;
  }
  // ...
}
  
// 将私有方法移出模块
class Widget {
  foo (baz) { // foo 是公有方法，内部调用了 bar.call(this, baz)。这使得 bar 实际上成为了当前模块的私有方法
    bar.call(this, baz);
  }
  // ...
}
function bar(baz) {
  return this.snaf = baz;
}
  
// 利用 Symbol 值的唯一性，将私有方法的名字命名为一个 Symbol 值
const bar = Symbol('bar');
const snaf = Symbol('snaf');
export default class myClass{
  foo(baz) {  // 公有方法
    this[bar](baz);
  }
  [bar](baz) {  // 私有方法，bar 和 snaf 都是 Symbol 值，导致第三方无法获取到它们，因此达到了私有方法和私有属性的效果
    return this[snaf] = baz;
  }
  // ...
};
  
// #x 就表示私有属性 x，私有属性与实例的属性是可以同名的
class Point {
  #x;
  constructor(x = 0) {
    #x = +x; // 写成 this.#x 亦可
  }
  get x() { return #x }
  set x(value) { #x = +value }
}
  
// 私有属性可以指定初始值，在构造函数执行时进行初始化
class Point {
  #x = 0; // 因为 JavaScript 是一门动态语言，所以没有采用private关键字；@ 已经被留给了 Decorator；所以采用 #
  constructor() {
    #x; // 0
  }
}
  
// 该提案只规定了私有属性的写法。但是，很自然地，它也可以用来写私有方法
class Foo {
  #a;
  #b;
  #sum() { return #a + #b; } // 私有方法
  printSum() { console.log(#sum()); }
  constructor(a, b) { #a = a; #b = b; }
}
  
// 私有属性也可以设置 getter 和 setter 方法
class Counter {
  #xValue = 0;
  get #x() { return #xValue; }  // #x 是一个私有属性，它的读写都通过 get #x() 和 set #x() 来完成
  set #x(value) {
    this.#xValue = value;
  }
  constructor() {
    super();
    // ...
  }
}
```
### 六、this 的指向 & name 属性
类的方法内部如果含有 this，它默认指向类的实例。但是，必须非常小心，一旦单独使用该方法，很可能报错。    
由于本质上，ES6 的类只是 ES5 的构造函数的一层包装，所以函数的许多特性都被 Class 继承，包括 name 属性。
```
// 单独提取 printName 出来单独使用，this 会指向该方法运行时所在的环境，因为找不到 print 方法而导致报错
class Logger {
  printName(name = 'there') {
    this.print(`Hello ${name}`);  // printName 方法中的 this，默认指向 Logger 类的实例
  }
  print(text) {
    console.log(text);
  }
}
const logger = new Logger();
const { printName } = logger;
printName(); // TypeError: Cannot read property 'print' of undefined
  
// 解决方法一：在构造方法中绑定 this
class Logger {
  constructor() {
    this.printName = this.printName.bind(this);
  }
  // ...
}
  
// 解决方法二：使用箭头函数（函数体内的 this 对象就是定义时所在的对象，而不是使用时所在的对象）
class Logger {
  constructor() {
    this.printName = (name = 'there') => {
      this.print(`Hello ${name}`);
    };
  }
  // ...
}
  
// 解决方法三：使用 Proxy，获取方法的时候自动绑定 this
function selfish (target) {
  const cache = new WeakMap();
  const handler = {
    get (target, key) {
      const value = Reflect.get(target, key);
      if (typeof value !== 'function') {
        return value;
      }
      if (!cache.has(value)) {
        cache.set(value, value.bind(target));
      }
      return cache.get(value);
    }
  };
  const proxy = new Proxy(target, handler);
  return proxy;
}
const logger = selfish(new Logger());
  
// name 属性总是返回紧跟在 class 关键字后面的类名
class Point {}
Point.name // "Point"
```
### 七、Class 的取值函数（getter）和存值函数（setter）
与 ES5 一样，在“类”的内部可以使用 get 和 set 关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。
```
// prop 属性有对应的存值函数和取值函数，因此赋值和读取行为都被自定义了
class MyClass {
  constructor() {
    // ...
  }
  get prop() {
    return 'getter';
  }
  set prop(value) {
    console.log('setter: '+value);
  }
}
let inst = new MyClass();
inst.prop = 123;  // setter: 123
inst.prop // 'getter'
  
// 存值函数和取值函数是设置在属性的 Descriptor 对象上的
class CustomHTMLElement {
  constructor(element) {
    this.element = element;
  }
  get html() {
    return this.element.innerHTML;
  }
  set html(value) {
    this.element.innerHTML = value;
  }
}
var descriptor = Object.getOwnPropertyDescriptor( // 存值函数和取值函数是定义在 html 属性的描述对象上面，这与 ES5 完全一致
  CustomHTMLElement.prototype, "html"
);
"get" in descriptor  // true
"set" in descriptor  // true
```
### 八、Class 的 Generator 方法 & 静态方法
如果某个方法之前加上星号（*），就表示该方法是一个 Generator 函数。   
类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上 static 关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为 "静态方法"。
```
// Symbol.iterator 方法前有一个星号，表示该方法是一个 Generator 函数
class Foo {
  constructor(...args) {
    this.args = args;
  }
  * [Symbol.iterator]() {
    for (let arg of this.args) {
      yield arg;
    }
  }
}
for (let x of new Foo('hello', 'world')) {  // Symbol.iterator 方法返回一个 Foo 类的默认遍历器，for...of 循环会自动调用这个遍历器
  console.log(x); // hello，world
}
  
// 如果在实例上调用静态方法，会抛出一个错误，表示不存在该方法
class Foo {
  static classMethod() {  // Foo 类的 classMethod 方法前有 static 关键字，表明该方法是一个静态方法，可以直接在 Foo类上调用
    return 'hello';
  }
}
Foo.classMethod() // 'hello'
var foo = new Foo();
foo.classMethod();  // TypeError: foo.classMethod is not a function
  
// 如果静态方法包含 this 关键字，这个 this 指的是类，而不是实例
class Foo {
  static bar () {
    this.baz();
  }
  static baz () {
    console.log('hello'); // this 指的是 Foo 类，而不是 Foo 的实例，等同于调用 Foo.baz
  }
  baz () {
    console.log('world');
  }
}
Foo.bar() // hello
  
// 父类的静态方法可以被子类继承
class Foo {
  static classMethod() {
    return 'hello';
  }
}
class Bar extends Foo {
}
Bar.classMethod() // 'hello'
  
// 静态方法也是可以从 super 对象上调用的
class Foo {
  static classMethod() {
    return 'hello';
  }
}
class Bar extends Foo {
  static classMethod() {
    return super.classMethod() + ', too';
  }
}
Bar.classMethod() // "hello, too"
```
### 九、Class 的静态属性和实例属性
静态属性指的是 Class 本身的属性，即 Class.propName，而不是定义在实例对象（this）上的属性。
```
// 为 Foo 类定义了一个静态属性 prop（目前只有这种写法可行）
class Foo {
}
Foo.prop = 1;
Foo.prop // 1
  
// 因为 ES6 明确规定 Class 内部只有静态方法，没有静态属性（以下两种写法都无效）
class Foo {
  prop: 2 // 写法一
  static prop: 2  // 写法二
}
Foo.prop // undefined
  
// 提案：类的实例属性
class MyClass {
  myProp = 42;  // 类的实例属性可以用等式，写入类的定义之中
  constructor() {
    console.log(this.myProp); // 42
  }
}
  
// 以前我们定义实例属性，只能写在类的 constructor 方法里面
class ReactCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
}
  
// 提案：有了新的写法以后，可以不在 constructor 方法里面定义，写法比以前更清晰
class ReactCounter extends React.Component {
  state = {
    count: 0
  };
}
  
// 提案：为了可读性的目的，对于那些在 constructor 里面已经定义的实例属性，新写法允许直接列出
class ReactCounter extends React.Component {
  state;
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
}
  
// 提案：类的静态属性只要在上面的实例属性写法前面，加上 static 关键字就可以了
class MyClass {
  static myStaticProp = 42;
  constructor() {
    console.log(MyClass.myStaticProp); // 42
  }
}
  
// 提案：新写法大大方便了静态属性的表达
class Foo { // 老写法，静态属性定义在类的外部，这样让人很容易忽略这个静态属性，也不符合相关代码应该放在一起的代码组织原则
  // ...
}
Foo.prop = 1;
class Foo { // 新写法，显式声明，而不是赋值处理，语义更好
  static prop = 1;
}
```
### 十、new.target 属性
```
// 确保构造函数只能通过 new 命令调用
function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 命令生成实例');
  }
}
function Person(name) { // 另一种写法
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 命令生成实例');
  }
}
var person = new Person('张三'); // 正确
var notAPerson = Person.call(person, '张三');  // 报错
  
// Class 内部调用 new.target，返回当前 Class
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    this.length = length;
    this.width = width;
  }
}
var obj = new Rectangle(3, 4); // 输出 true
  
// 子类继承父类时，new.target 会返回子类
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    // ...
  }
}
class Square extends Rectangle {
  constructor(length) {
    super(length, length);
  }
}
var obj = new Square(3); // 输出 false，new.target 会返回子类
  
// 利用这个特点，可以写出不能独立使用、必须继承后才能使用的类（注意在函数外部，使用 new.target 会报错）
class Shape {
  constructor() {
    if (new.target === Shape) { // Shape 类不能被实例化，只能用于继承
      throw new Error('本类不能实例化');
    }
  }
}
class Rectangle extends Shape {
  constructor(length, width) {
    super();
    // ...
  }
}
var x = new Shape();  // 报错
var y = new Rectangle(3, 4);  // 正确
```
new 是从构造函数生成实例对象的命令。ES6 为 new 命令引入了一个 new.target 属性，该属性一般用在构造函数之中，返回 new 命令作用于的那个构造函数。如果构造函数不是通过 new 命令调用的，new.target 会返回 undefined，因此这个属性可以用来确定构造函数是怎么调用的。