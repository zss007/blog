---
title: es6 之 Decorator
categories:
- es6
---
许多面向对象的语言都有修饰器（Decorator）函数，用来修改类的行为。目前，有一个提案将这项功能，引入了 ECMAScript。
<!--more-->
### 一、简介
```
// @testable 就是一个修饰器，修改了 MyTestableClass 这个类的行为，为它加上了静态属性 isTestable
@testable
class MyTestableClass {
  // ...
}
function testable(target) { // testable 函数的参数 target 是 MyTestableClass 类本身
  target.isTestable = true;
}
MyTestableClass.isTestable // true
  
// 修饰器的行为: 修饰器是一个对类进行处理的函数，修饰器函数的第一个参数，就是所要修饰的目标类
@decorator
class A {}
class A {}  // 等同于
A = decorator(A) || A;
  
// 如果觉得一个参数不够用，可以在修饰器外面再封装一层函数（修饰器对类的行为的改变，是代码编译时发生的，而不是在运行时）
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;
  }
}
@testable(true)
class MyTestableClass {}
MyTestableClass.isTestable // true
@testable(false)
class MyClass {}
MyClass.isTestable // false
  
// 添加实例属性可以通过目标类的 prototype 对象操作
function testable(target) {
  target.prototype.isTestable = true;
}
@testable
class MyTestableClass {}
let obj = new MyTestableClass();
obj.isTestable // true
  
// 通过修饰器 mixins，把 Foo 对象的方法添加到了 MyClass 的实例上面
export function mixins(...list) { // mixins.js
  return function (target) {
    Object.assign(target.prototype, ...list)
  }
}
import { mixins } from './mixins' // main.js
const Foo = {
  foo() { console.log('foo') }
};
@mixins(Foo)
class MyClass {}
let obj = new MyClass();
obj.foo() // 'foo'
  
// 用 Object.assign() 模拟上面的功能
const Foo = {
  foo() { console.log('foo') }
};
class MyClass {}
Object.assign(MyClass.prototype, Foo);
let obj = new MyClass();
obj.foo() // 'foo'
  
// React 与 Redux 库结合使用时，常常需要写成下面这样
class MyReactComponent extends React.Component {}
export default connect(mapStateToProps, mapDispatchToProps)(MyReactComponent);
  
// 使用装饰器改写，看上去更容易理解
@connect(mapStateToProps, mapDispatchToProps)
export default class MyReactComponent extends React.Component {}
```
### 二、方法的修饰
修饰器不仅可以修饰类，还可以修饰类的属性。
```
// 修饰器 readonly 用来修饰 "类" 的 name 方法
class Person {
  @readonly
  name() { return `${this.first} ${this.last}` }
}
  
// 修饰器函数 readonly 可接受三个参数，分别是: 类的原型对象、所要修饰的属性名、该属性的描述对象
function readonly(target, name, descriptor){
  // descriptor对象原来的值如下
  // {
  //   value: specifiedFunction,
  //   enumerable: false,
  //   configurable: true,
  //   writable: true
  // };
  descriptor.writable = false;  // 修饰器会修改属性的描述对象（descriptor），然后被修改的描述对象再用来定义属性
  return descriptor;
}
readonly(Person.prototype, 'name', descriptor);
Object.defineProperty(Person.prototype, 'name', descriptor);  // 类似于
  
// 修改属性描述对象的 enumerable 属性，使得该属性不可遍历
class Person {
  @nonenumerable
  get kidCount() { return this.children.length; }
}
function nonenumerable(target, name, descriptor) {
  descriptor.enumerable = false;
  return descriptor;
}
  
// @log修饰器，起到输出日志的作用: 在执行原始的操作之前执行一次 console.log，从而达到输出日志的目的
class Math {
  @log
  add(a, b) {
    return a + b;
  }
}
function log(target, name, descriptor) {
  var oldValue = descriptor.value;
  descriptor.value = function() {
    console.log(`Calling ${name} with`, arguments);
    return oldValue.apply(null, arguments);
  };
  return descriptor;
}
const math = new Math();
math.add(2, 4); // passed parameters should get logged now
  
// 修饰器有注释的作用: 一眼就能看出 Person 类是可测试的，而 name 方法是只读和不可枚举的
@testable
class Person {
  @readonly
  @nonenumerable
  name() { return `${this.first} ${this.last}` }
}
  
// 使用 Decorator 写法的组件，看上去一目了然
@Component({
  tag: 'my-component',
  styleUrl: 'my-component.scss'
})
export class MyComponent {
  @Prop() first: string;
  @Prop() last: string;
  @State() isVisible: boolean = true;
  render() {
    return (
      <p>Hello, my name is {this.first} {this.last}</p>
    );
  }
}
  
// 同一个方法有多个修饰器，会像剥洋葱一样，先从外到内进入，然后由内向外执行
function dec(id){
  console.log('evaluated', id);
  return (target, property, descriptor) => console.log('executed', id);
}
class Example { // 外层修饰器 @dec(1) 先进入，但是内层修饰器 @dec(2) 先执行
    @dec(1)
    @dec(2)
    method(){}
}
// evaluated 1
// evaluated 2
// executed 2
// executed 1
```
### 三、修饰器不能用于函数
修饰器只能用于类和类的方法，不能用于函数，因为存在函数提升。而类是不会提升的，所以就没有这方面的问题。
```
// 意图是执行后 counter 等于 1，但是实际上结果是 counter 等于 0
var counter = 0;
var add = function () {
  counter++;
};
@add
function foo() {
}
  
// 函数提升，使得实际执行顺序如下
@add
function foo() {
}
var counter;
var add;
counter = 0;
add = function () {
  counter++;
};
  
// 另一个例子
var readOnly = require("some-decorator");
@readOnly
function foo() {
}
  
// 实际执行顺序
var readOnly;
@readOnly
function foo() {
}
readOnly = require("some-decorator");
```
### 四、使用修饰器实现自动发布事件
我们可以使用修饰器，使得对象的方法被调用时，自动发出一个事件。
```
// 定义了一个名为 publish 的修饰器，它通过改写descriptor.value，使得原方法被调用时，会自动发出一个事件
const postal = require("postal/lib/postal.lodash"); // 使用的事件“发布/订阅”库是 Postal.js
export default function publish(topic, channel) {
  const channelName = channel || '/';
  const msgChannel = postal.channel(channelName);
  msgChannel.subscribe(topic, v => {
    console.log('频道: ', channelName);
    console.log('事件: ', topic);
    console.log('数据: ', v);
  });
  return function(target, name, descriptor) {
    const fn = descriptor.value;
    descriptor.value = function() {
      let value = fn.apply(this, arguments);
      msgChannel.publish(topic, value);
    };
  };
}
  
// 用法如下
import publish from './publish';  // index.js
class FooComponent {
  @publish('foo.some.message', 'component')
  someMethod() {
    return { my: 'data' };
  }
  @publish('foo.some.other')
  anotherMethod() {
    // ...
  }
}
let foo = new FooComponent();
foo.someMethod(); // 只要调用 someMethod 或者 anotherMethod，就会自动发出一个事件
foo.anotherMethod();
  
$ bash-node index.js
频道:  component
事件:  foo.some.message
数据:  { my: 'data' }
频道:  /
事件:  foo.some.other
数据:  undefined
```
### 五、Mixin
在修饰器的基础上，可以实现 Mixin 模式。所谓 Mixin 模式，就是对象继承的一种替代方案，中文译为 "混入"（mix in），意为在一个对象之中混入另外一个对象的方法。
```
// 简单实现
const Foo = {
  foo() { console.log('foo') }
};
class MyClass {}
Object.assign(MyClass.prototype, Foo);  // 将 foo 方法 "混入" MyClass 类，导致 MyClass 的实例 obj 对象都具有 foo 方法
let obj = new MyClass();
obj.foo() // 'foo'
  
// 部署一个通用脚本 mixins.js，将 Mixin 写成一个修饰器
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list);
  };
}
  
// 使用上面这个修饰器，为类 "混入" 各种方法，会改写 MyClass 类的 prototype 对象
import { mixins } from './mixins';
const Foo = {
  foo() { console.log('foo') }
};
@mixins(Foo)
class MyClass {}
let obj = new MyClass();
obj.foo() // "foo"
  
//  MyMixin 是一个混入类生成器，接受 superclass 作为参数，然后返回一个继承 superclass 的子类，该子类包含一个 foo 方法
let MyMixin = (superclass) => class extends superclass {
  foo() {
    console.log('foo from MyMixin');
  }
};
  
// 目标类再去继承这个混入类，就达到了 "混入" foo 方法的目的
class MyClass extends MyMixin(MyBaseClass) {
  /* ... */
}
let c = new MyClass();
c.foo(); // "foo from MyMixin"
  
// 如果需要 "混入" 多个方法，就生成多个混入类
class MyClass extends Mixin1(Mixin2(MyBaseClass)) {
  /* ... */
}
  
// 这种写法的一个好处，是可以调用 super，因此可以避免在 "混入" 过程中覆盖父类的同名方法
let Mixin1 = (superclass) => class extends superclass {
  foo() {
    console.log('foo from Mixin1');
    if (super.foo) super.foo();
  }
};
let Mixin2 = (superclass) => class extends superclass {
  foo() {
    console.log('foo from Mixin2');
    if (super.foo) super.foo();
  }
};
class S {
  foo() {
    console.log('foo from S');
  }
}
class C extends Mixin1(Mixin2(S)) {
  foo() {
    console.log('foo from C');
    super.foo();
  }
}
  
// 每一次混入发生时，都调用了父类的 super.foo 方法，导致父类的同名方法没有被覆盖，行为被保留了下来
new C().foo()
// foo from C
// foo from Mixin1
// foo from Mixin2
// foo from S
```
### 六、Trait
Trait 也是一种修饰器，效果与 Mixin 类似，但是提供更多功能，比如防止同名方法的冲突、排除混入某些方法、为混入的方法起别名等等。下面采用 traits-decorator 这个第三方模块作为例子，这个模块提供的 traits 修饰器，不仅可以接受对象，还可以接受 ES6 类作为参数。
```
// 通过 traits 修饰器，在 MyClass 类上面 "混入" 了 TFoo 类的 foo 方法和 TBar 对象的 bar 方法
import { traits } from 'traits-decorator';
class TFoo {
  foo() { console.log('foo') }
}
const TBar = {
  bar() { console.log('bar') }
};
@traits(TFoo, TBar)
class MyClass { }
let obj = new MyClass();
obj.foo() // foo
obj.bar() // bar
  
// Trait 不允许 "混入" 同名方法
import { traits } from 'traits-decorator';
class TFoo {
  foo() { console.log('foo') }
}
const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
};
@traits(TFoo, TBar)
class MyClass { }
// 报错
// throw new Error('Method named: ' + methodName + ' is defined twice.');
//        ^
// Error: Method named: foo is defined twice.
  
// 一种解决方法是排除 TBar 的 foo 方法
import { traits, excludes } from 'traits-decorator';
class TFoo {
  foo() { console.log('foo') }
}
const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
};
@traits(TFoo, TBar::excludes('foo'))  // 使用绑定运算符（::）在 TBar 上排除 foo 方法
class MyClass { }
let obj = new MyClass();
obj.foo() // foo
obj.bar() // bar
  
// 另一种方法是为 TBar 的 foo 方法起一个别名
import { traits, alias } from 'traits-decorator';
class TFoo {
  foo() { console.log('foo') }
}
const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
};
@traits(TFoo, TBar::alias({foo: 'aliasFoo'}))
class MyClass { }
let obj = new MyClass();
obj.foo() // foo
obj.aliasFoo() // foo
obj.bar() // bar
  
// alias 和 excludes 方法，可以结合起来使用（排除了 TExample 的 foo 方法和 bar 方法，为 baz 方法起了别名 exampleBaz）
@traits(TExample::excludes('foo','bar')::alias({baz:'exampleBaz'}))
class MyClass {}
  
// as 方法则为上面的代码提供了另一种写法
@traits(TExample::as({excludes:['foo', 'bar'], alias: {baz: 'exampleBaz'}}))
class MyClass {}
```
### 七、Babel 转码器的支持
```
$ npm install babel-core babel-plugin-transform-decorators
  
// 设置配置文件 .babelrc
{
  "plugins": ["transform-decorators"]
}
  
// 脚本中打开的命令如下
babel.transform("code", {plugins: ["transform-decorators"]})
```
目前，Babel 转码器已经支持 Decorator。首先安装 babel-core 和 babel-plugin-transform-decorators。由于后者包括在
babel-preset-stage-0 之中，所以改为安装 babel-preset-stage-0 亦可。