---
title: js 之this
categories:
- note
---
之前较为系统的学习了JavaScript的this用法，但是没多久又忘记了。从网上看到些不错的经验，所以借鉴了下，感觉挺不错的。
<!--more-->
### 一、简介
说到this，我们会想到function，各种面试之类的考察function中的this，平时也经常用得上。这时我们要明确两件事：1、function是一个对象；2、function是要在特定上下文执行的。
### 二、call方法
call可以用来改变函数的this对象指向，第一个参数用来指定this，后面的参数用来传参。在这里，我们就用call帮助我们记住function的this到底指向什么，即在函数调用的时候转化为call方法调用。
```
//例一，介绍下call的用法
function demoOne(param) {
  console.log("Good " + param + ", " + this);
}
demoOne.call("Bob", "morning");//Good morning, Bob

//例二，可以使用call方法代替function直接调用
function demoTow(param) {
  console.log(param);
}
demoTow("Hello world");
demoTow.call(window, "Hello world");

//例三，匿名函数同样适用
(function (name) {
})("aa");
//转化为
(function (name) {
}).call(window, "aa");

//例四，函数作为对象方法被调用
var name = 'Make';
var demoFour = {
  name: 'Ben',
  showMessage: function (param) {
    console.log(param + ", " + this.name);
  }
};
demoFour.showMessage('Hello'); //Hello, Ben
demoFour.showMessage.call(demoFour, 'Hello');  //Hello, Ben
demoFour.showMessage.call(window, 'Hello');  //Hello, Make

//例五，通过call方法修改this指定值使获取需求值
var num = 10;
var obj = {
  num: 20,
  f: function () {
    console.log(this.num);
  }
};
var foo = function () {
  console.log(this.num);
};
obj.f();  //20
foo();    //10
foo.call(obj);  //20

//例六，通过call方法修改this指定值，就算赋值转换，套用我们的方法还是可以的
var numTwo = 10;
var objTwo = {
  numTwo: 20,
  f: function () {
    console.log(this.numTwo);
  }
};
var fOut = objTwo.f;
var objThree = {
  numTwo: 30,
  f: objTwo.f
};
fOut();     // fOut.call(window)//==> 10
objTwo.f();  // objTwo.f.call(objTwo)// ==> 20
objThree.f();// objThree.f.call(objThree)//==> 30

//例七，构造函数
//其实new constructor()是一种创建对象的语法糖
function Person(name) {
  this.name = name;
}
//通过new创建了一个对象
var studentOne = new Person("deen");
console.log(studentOne.name);
//等价于
var studentTwo = {
  constructor: Person,
  __proto__: Person.prototype
};
studentTwo.constructor.call(studentTwo, 'deen');
console.log(studentTwo.name);
```
只要我们把两个等价变形理解了，this就不是问题了：fun()--->fun.call(window),obj.fun()--> obj.fun.call(obj)
### 三、apply方法
apply方法相对于call而言，作用完全一样，只是接受参数的方式不太一样。
```
func.call(this, arg1, arg2);
func.apply(this, [arg1, arg2])
```
call需要把参数按顺序传递进去，而apply则是把参数放在数组里。

### 四、bind方法
bind()方法会创建一个新函数，称为绑定函数，当调用这个绑定函数时，绑定函数会以创建它时传入bind()方法的第一个参数作为this，传入bind()方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。
```
function Add(num1, num2) {
    console.log(this.value + num1 + num2);
}

//使用call方法实现该效果
//Add.bindX = function (obj, num) {
//  return function (num2) {
//    Add.call(obj, num, num2);
//  }
//};

//使用bind方法实现该效果
Add.bindX = function (obj, num) {
  return this.bind(obj, num);
};

var data = {value: 1};
var bind = Add.bindX(data, 2);
bind(3); // 6
```
### 五、总结一下
1、apply、call、bind三者都可以用来改变函数的this对象的指向的；
2、apply、call、bind三者第一个参数都是this要指向的对象，也就是想指定的上下文；
3、apply、call、bind三者都可以利用后续参数传参，call和bind是按顺序传参，apply是用数组形式传参；
4、bind是返回对应函数，便于稍后调用；apply、call则是立即调用。