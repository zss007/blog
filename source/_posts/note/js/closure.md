---
title: js 之闭包
categories:
- note
---
闭包，是javascript中重要的一个概念，在这里做个总结。
<!--more-->
### 一、什么是闭包
我们以一个最简单的例子来看下什么是闭包。
```
function A() {
  function B() {
    console.log("Hello Closure!");
  }

  return B;
}
var c = A();
c();//Hello Closure!
```
让我们简单分析一下它和普通函数有什么不同：  
- 1、定义了一个普通函数A；   
- 2、在A中定义了普通函数B；  
- 3、在A中返回B(确切的讲，在A中返回B的引用)；  
- 4、执行A()，把A的返回结果赋值给变量C；  
- 5、执行C()  

把这些操作总结下就是：函数A的内部函数B被函数A外的一个变量C引用，即当一个内部函数被其外部函数之外的变量引用时，就形成了一个闭包。
### 二、闭包的作用
举个例子，如果HTML中有如下面相同的四个a元素，现在要给它们指定事件处理程序，使它们在用户单击时报告自己在页面中的顺序。
```
<a href="#">点我测试</a> 
<a href="#">点我测试</a> 
<a href="#">点我测试</a> 
<a href="#">点我测试</a>
```
很多人会掉入陷阱中，写下如下代码
```
function badClosureExample() {
  var as = document.querySelectorAll('a');
  for (var i = 0; i < 4; i++) {
    as[i].onclick = function () {
      alert('单击第' + i + '个');
    }
  }
}

badClosureExample();
```
但是运行结果却是单击任何任意一个链接全都是"单击第4个"，因为在badClosureExample()函数中指定给element.onclick的事件处理程序，也就是onclick那个匿名函数是在badClosureExample()函数运行完成后（用户单击链接时）才被调用的。而调用时，需要对变量i求值，解析程序首先会在事件处理程序内部查找，但i没有定义。然后，又到badClosureExample()函数中查找，此时有定义，但i的值是4（只有i大于4才会停止执行for循环）。解决方案如下
```
function badClosureExample() {
  var as = document.querySelectorAll('a');
  for (var i = 0; i < 4; i++) {
    (function (i) {
      as[i].onclick = function () {
        alert('单击第' + i + '个');
      }
    })(i);
  }
}

badClosureExample();
```
再举个例子(返回对象)：
```
(function (document) {
  var viewport;
  var obj = {
    init: function (id) {
      viewport = document.querySelector("#" + id);
    },
    addChild: function (child) {
      viewport.appendChild(child);
    },
    removeChild: function (child) {
      viewport.removeChild(child);
    }
  };
  window.jView = obj;
})(document);
```
在这段代码中似乎看到了闭包的影子，但匿名函数中没有任何返回值，似乎不具备闭包的条件。不过把obj赋值给了window全局对象的一个变量jView，即全局变量jView引用了obj，而obj对象中的函数又引用了匿名函数中的变量viewport，因此f中的viewport不会被GC回收，会一直保存到内存中，所以这种写法满足闭包的条件。  
再举个例子(对象创建):
```
function a() {
  var n = 0;
  this.inc = function () {
    n++;
    console.log(n);
  };
}
var c = new a();
c.inc();  //1
c.inc();  //2
```
这里inc函数访问了构造函数a里面的变量n，所以形成了一个闭包。
### 三、函数作用域链
```
function fun(n,o) {
  console.log(o)
  return {
    fun:function(m){
      return fun(m,n);
    }
  };
}
var a = fun(0);  a.fun(1);  a.fun(2);  a.fun(3);//undefined,?,?,?
var b = fun(0).fun(1).fun(2).fun(3);//undefined,?,?,?
var c = fun(0).fun(1);  c.fun(2);  c.fun(3);//undefined,?,?,?
```
先举个例子，这是一道非常典型的JS闭包问题。其中嵌套了三层fun函数，搞清楚每层fun的函数是那个fun函数尤为重要。
#### 3.1、创建函数的几种方式
- 1、声明函数

```
function fn1(){}
```

- 2、创建匿名函数表达式

```
var fn1=function (){};
```

- 3、创建具名函数表达式

```
var fn1=function xxcanghai(){};
```
创建一个变量，内容为一个带有名称的函数，具名函数表达式的函数名只能在创建函数内部使用，即采用此种方法创建的函数在函数外层只能使用fn1不能使用xxcanghai的函数名。xxcanghai的命名只能在创建的函数内部使用

- 4、Function构造函数，此种方法创建的是匿名函数。

- 5、自执行函数

```
(function(){alert(1);})();
(function fn1(){alert(1);})();
```

- 6、其他创建函数的方法

比如采用eval，setTimeout，setInterval等非常用方法，属于非标准方法，这里不做过多展开。
自执行函数属于上述的“函数表达式”，规则相同。
#### 3.2、函数作用域链的问题
在说第三个fun函数之前需要先说下，在函数表达式内部能不能访问存放当前函数的变量。
```
var o = {
  fn: function () {
    console.log(fn);
  }
};
o.fn();//fn is not defined

var fn = function () {
  console.log(fn);
};
fn();//[Function: fn]
```
从上面两个例子可以看出非对象内部的函数表达式内，可以访问到存放当前函数的变量；在对象内部的不能访问到。原因也非常简单，因为函数作用域链的问题，采用非对象内部的函数表达式是在外部创建了一个fn变量，函数内部当然可以在内部寻找不到fn后向上沿作用域查找fn，而在创建对象内部时，因为没有在函数作用域内创建fn，所以无法访问。  
所以综上所述，可以得知，最内层的return出去的fun函数不是第二层fun函数，是最外层的fun函数。所以，三个fun函数的关系也理清楚了，第一个等于第三个，他们都不等于第二个。
#### 3.3、第一行
第一个fun(0)是在调用第一层fun函数。  
第二个fun(1)是在调用前一个fun的返回值的fun函数，  
所以后面几个fun(1),fun(2),fun(3),函数都是在调用第二层fun函数。所以结果是undefined、0、0、0
#### 3.4、第二行
第一次调用第一层fun(0)时，o为undefined；  
第二次调用fun(1)时m为1，此时fun闭包了外层函数的n，也就是第一次调用的n=0，即m=1，n=0，并在内部调用第一层fun函数fun(1,0)，所以o为0；
第三次调用fun(2)时m为2，此时当前的fun函数不是第一次执行的返回对象，而是第二次执行的返回对象。而在第二次执行第一层fun(1,0)，所以n=1,o=0,返回时闭包了第二次的n，遂在第三次调用第三层fun函数时m=2,n=1，即调用第一层fun函数fun(2,1)，所以o为1；  
第四次调用fun(3)时m为3，闭包了第三次调用的n，同理，最终调用第一层fun函数为fun(3,2)；所以o为2，即最终答案：undefined,0,1,2
#### 3.5、第三行
第一次调用第一层fun(0)时，o为undefined；  
第二次调用fun(1)时m为1，此时fun闭包了外层函数的n，也就是第一次调用的n=0，即m=1，n=0，并在内部调用第一层fun函数fun(1,0);所以o为0；   
第三次调用fun(2)时m为2，此时fun闭包的是第二次调用的n=1，即m=2，n=1，并在内部调用第一层fun函数fun(2,1);所以o为1;       
第四次调用fun(3)时同理，但依然是调用的第二次的返回值，遂最终调用第一层fun函数fun(3,1)，所以o还为1；  
即最终答案：undefined,0,1,1
### 四、垃圾回收原理及内存泄漏
#### 4.1、垃圾回收
- 1、在javascript中，如果一个对象不再被引用，那么这个对象就会被GC回收； 
- 2、如果两个对象互相引用，而不再被第3者所引用，那么这两个互相引用的对象也会被回收。在js中使用闭包，往往会给javascript的垃圾回收器制造难题。尤其是遇到对象间复杂的循环引用时，垃圾回收的判断逻辑非常复杂，搞不好就有内存泄漏的危险。 

#### 4.2、内存泄漏
- 内存泄漏是内存占用很大吗？不是，即使1Byte的内存也叫内存泄漏  
- 程序中提示"内存不足"不是内存泄漏，一般是某些原因导致"内存溢出"  
- 内存泄漏是堆区，栈区不会内存泄漏  
- 跳转页面后，内存泄漏仍然存在，直到关闭浏览器  
- window对象是DOM对象吗？否，window对象参与的循环引用不会导致内存泄漏  
- 内存泄漏后果不是很严重，但过多DOM操作会导致网页执行缓慢  