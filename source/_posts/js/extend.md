---
title: js 之面对对象
categories:
- js
---
本节主要介绍类、实例和继承，并整理总结 JS 继承的几种方法，并分析各方法的优缺点。
<!--more-->
### 一、类的声明和实例化
```
// 类的声明
function Animal() {
    this.name = 'name'
}

// ES6 中的 class 声明
class Animal2 {
    constructor() {
        this.name = 'name'
    }
}

// 类的实例化
console.log(new Animal(), new Animal2())
```
### 二、构造函数继承
```
  function Parent1() {
    this.name = 'parent1';
  }

  Parent1.prototype.say = function () {
  };
  
  function Child1() {
    Parent1.call(this);
    this.type = 'child1';
  }
  
  console.log(new Child1());  // Child1 {name: "parent1", type: "child1"}
  console.log(new Child1().say());  // Uncaught TypeError: xxx say is not a function
```
优点：继承了 Parent1 的 name 属性；
缺点：没有继承 Parent1 原型链上的方法。
### 二、原型链方法继承
```
  function Parent2() {
    this.name = 'parent2';
    this.play = [1, 2, 3];
  }

  Parent2.prototype.say = function () {
    console.log('Parent2 say');
  };

  function Child2() {
    this.type = 'child2';
  }

  Child2.prototype = new Parent2();

  var s1 = new Child2();
  s1.say();  //P arent2 say
  console.log(s1.name, s1.type); // parent2 child2
  var s2 = new Child2();
  s1.play.push(4);
  console.log(s1.play, s2.play);  // [1, 2, 3, 4]、[1, 2, 3, 4]
```
优点：不仅继承了 Parent2 的 name、play 属性而且继承了其原型链上的方法；
缺点：由于将 Parent2 的实例作为 Child2 的原型，导致所有 Child2 实例共享 Parent2 的属性方法，其中一个 Child2 实例改变了原型链上 Parent2 的实例属性，其他实例会受到影响跟着改变。
### 三、组合方法继承
```
  function Parent3 () {
    this.name = 'parent3';
    this.play = [1, 2, 3];
  }

  function Child3 () {
    Parent3.call(this);
    this.type = 'child3';
  }

  Child3.prototype = new Parent3();
  
  var s3 = new Child3();
  var s4 = new Child3();
  s3.play.push(4);
  console.log(s3.play, s4.play); // [1, 2, 3, 4]、[1, 2, 3]
```
优点：避免了原型链方法中出现的实例间相互影响(调用 Parent3.call(this) 使得 Child3 实例上有 play 属性，不必找到原型链上)；
缺点：每次得到一个 Child3 实例，都会调用两次 Parent3 函数。
### 四、组合方法(优化一)
```
  function Parent4 () {
    this.name = 'parent4';
    this.play = [1, 2, 3];
  }

  function Child4 () {
    Parent4.call(this);
    this.type = 'child4';
  }

  Child4.prototype = Parent4.prototype;

  var s5 = new Child4();
  var s6 = new Child4();
  console.log(s5); // Child4 {name: "parent4", play: [1, 2, 3] type: "child4"}
  console.log(s6); // Child4 {name: "parent4", play: [1, 2, 3] type: "child4"}
  console.log(s5 instanceof Child4, s5 instanceof Parent4); // true true
  console.log(s5.constructor); // Parent4() {this.name = 'parent4';this.play = [1, 2, 3];}
```
优点：避免了 Parent4 重复调用问题；
缺点：Child4 实例的构造函数不是 Child4，而是 Parent4(其实这不是优化带来的问题，优化前同样存在这个问题,赋值后 child 上的 constructor 只能在原型链上查找)
### 五、组合方法(优化二)
```
  function Parent5 () {
    this.name = 'parent5';
    this.play = [1, 2, 3];
  }

  function Child5 () {
    Parent5.call(this);
    this.type = 'child5';
  }

  Child5.prototype = Object.create(Parent5.prototype);
  Child5.prototype.constructor = Child5;
  
  var s7 = new Child5();
  console.log(s7 instanceof Child5, s7 instanceof Parent5); //true true
  console.log(s7.constructor); //Child5() {Parent5.call(this);this.type = 'child5';}
```
优点：Child5 实例的构造函数是 Child5(这里采用 Object.create 方法而不是直接 Parent4.prototype 赋值，直接赋值会影响到 Parent5 实例的构造函数)。