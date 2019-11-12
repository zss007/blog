---
title: js 之作用域
categories:
- note
---
最近在学习es6，看到es6的作用域、新增的let和const声明命令和es5有了很大的不同，所以想着总结下es5的作用域和变量提升。
<!--more-->
### 一、作用域
es5是没有块级作用域的，当你在函数外声明一个变量，那么你在代码任何地方都能访问到，这个变量也被称为全局变量，拥有全局作用域；当你在一个函数内部声明变量的话，那么就只能在函数内部访问，这个变量被称为局部变量。
```
//es5没有块级作用域，所以声明赋值的num是全局的
if (true) {
  var num = 5;
}
console.log(num); // 5

//只有在函数内部声明变量才能称为局部变量，不能被全局访问到
function test() {
  var num2 = 5;
}
console.log(num2); //Uncaught ReferenceError: num2 is not defined
```
### 二、var变量提升
var声明的变量提升，赋值不提升。也就是说，用var命令声明赋值的语句，声明会被提升到当前作用域顶部，赋值却不提升。
```
//var num被提升到顶部，赋值不提升，所以就算是不执行，也不会报错，只是打印undefined
if (false) {
  var num = 5;
}
console.log(num); // undefined

//var num2被提升到顶部，num2=3没有提升，所以num2===undefined
console.log(num2 === undefined); //true
var num2 = 3;

//声明var num3提升到函数顶部，全部变量num3被覆盖，而函数内部赋值不提升，所以打印undefined
var num3 = "my value";
(function() {
  console.log(num3); // undefined
  var num3 = "local value";
})();

//不要被迷惑了，想着这里也是undefined，其实函数内部只有赋值语句，而赋值不提升，所以打印的是全局的变量
var num4 ='变量值';
(function(){
  console.log(num4);// 变量值
  num4 ='内部变量值';
})();
```
### 三、function函数提升
这里如果函数是声明式方法创建，则整体提升；如果是表达式方法创建则跟var提升规则相同。
```
//如果函数声明方法创建函数，则函数整体提升到顶部
var temp = fun;
console.log(temp);  //function fun() {console.log("dd");}
function fun() {
  console.log("dd");
}

//如果函数表达式创建函数，则跟var变量提升相同的规则
var tempTwo = funTwo;
console.log(tempTwo); //undefined
var funTwo = function () {
  console.log("dd");
};

//这个举个不太合适例子(同名)为了说明function和var变量谁在更顶部，事实证明function提升的位置在var之前，就算两者交换位置也是一样
console.log(test);  //function test() {console.log('dd');}
function test() {
  console.log('dd');
}
var test = 'dd';
```