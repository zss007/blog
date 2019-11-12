---
title: js 之创建对象
categories:
- note
---
整理总结下JS创建对象的几种方法。
<!--more-->
### 一、工厂模式
这种模式抽象了创建具体对象的过程。考虑到ECMAScript中无法创建类，开发人员就发明了一种函数，用函数来封装创建对象的细节。
```
function createPerson(name, age, job){
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function(){
    alert(this.name);
  };
  return o;
}
var person1 = createPerson("Nicholas", 29, "Software Engineer");
var person2 = createPerson("Greg", 27, "Doctor");
```
函数createPerson()能够根据接受的参数来创建一个包含所有必要信息的Person对象。可以无数次地调用这个函数，而每次它都会返回一个包含三个属性一个方法的对象。工厂函数虽然解决了创建多个相似对象的问题，但却没有解决对象识别的问题(即怎样知道一个对象的类型)。随着JavaScript的发展，又一个新模式出现了。
### 二、构造函数
ECMAScript中构造函数可以用来创建特定类型的对象，例如Object和Array这样的原生构造函数。
```
function Person(name, age, job){
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = function(){
    alert(this.name);
  };
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```
在这个例子中，Person()函数取代了CreatePerson函数。我们注意到，Person()中的代码除了与createPerson()中相同的部分外，还存在以下不同之处：
- 1、没有显式地创建对象；
- 2、直接将属性和方法赋给了this对象；
- 3、没有return语句。

要创建Person的新实例，必须使用new操作符。以这种方式调用构造函数实际上会经历以下4个步骤：
- 1、创建一个新对象；
- 2、将构造函数的作用域赋给新对象(因此this就指向了这个新对象)；
- 3、执行构造函数中的代码（为这个新对象添加属性和方法）；
- 4、返回新对象。

在前面的例子最后，person1和person2分别保存着Person的一个不同的实例。这两个对象都有一个constructor(构造函数)属性指向Person，如下所示。
```
alert(person1.constructor == Person); //true
alert(person2.constructor == Person); //true
```
但是，提到检测对象类型，还是instanceof操作符要更可靠一些。我们在这例子中创建的所有对象既是Object的实例，同时也是Person的实例，这一点通过instanceof操作符可以得到验证。
```
alert(person1 instanceof Object); //true
alert(person1 instanceof Person); //true
alert(person2 instanceof Object); //true
alert(person2 instanceof Person); //true
```
构造函数的问题：使用构造函数的主要问题，就是每个方法都要在每个实例上重新创建一遍person1和person2都有一个sayName()的方法，但那两个方法不是同一个Function的实例。
```
function Person(name, age, job){
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = new Function("alert(this.name)"); //与声明函数在逻辑上是等价的
}
```
以下代码可以证明这一点。
```
alert(person1.sayName == person2.sayName); //false
```
因此，大可像下面这样，通过把函数定义转移到构造函数外部来解决这个问题。
```
function Person(name, age, job){
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = sayName;
}
function sayName(){
  alert(this.name);
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```
### 三、原型模式
我们创建每个函数都有一个prototype(原型属性)，这个属性是一个指针，指向一个对象，而这个对象的用途是包含由特定类型的所有实例共享的属性和方法。如果按照字面意思来理解，那么prototype就是通过调用构造函数而创建的这个对象实例的原型对象。
使用原型对象的好处是可以让所有对象实例共享它所含的属性和方法。换句话说，不必在构造中定义对象实例的信息，而是可以将这些信息直接添加到原型对象中。
```
function Person() {
}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function () {
  alert(this.name);
};
var person1 = new Person();
person1.sayName(); //"Nicholas"
var person2 = new Person();
person2.sayName(); //"Nicholas"
alert(person1.sayName == person2.sayName); //true
```
但与构造函数模式不同的是，新对象的这些属性和方法是由所有实例共享的。换句话说，person1和person2访问的都是同一组属性和同一个sayName函数。
#### 3.1、理解原型对象
只要创建了一个新对象，就会根据一组特定规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个constructor(构造函数)属性，这个属性包含一个指向prototype属性所在函数的指针。就拿前面的例子来说，Person.prototype.constructor指向Person。
而通过这个构造函数，我们还可继续为原型对象添加其他属性和方法。当调用构造函数创建一个新实例后，该实例的内部将包含一个指针（内部属性），指向构造函数的原型对象。Firefox、Safari和Chrome 在每个对象上都支持一个属性`__proto__`；而在其他实现中，这个属性对脚本则是完全不可见的。不过，要明确的真正重要的一点就是，这个连接存在于实例与构造函数的原型对象之间，而不是存在于实例与构造函数之间。

<img src="/assets/note/js/object1.png">

此外，要格外注意的是，虽然这两个实例都不包含属性和方法，但我们却可以调用 person1.sayName()。这是通过查找对象属性的过程来实现的(属性->原型->原型...)。虽然在所有实现中都无法访问到[[Prototype]]，但可以通过isPrototypeOf()方法来确定对象之间是否存在这种关系。从本质上讲，如果[[Prototype]]指向调用isPrototypeOf()方法的对象（Person.prototype），那么这个方法就返回true，如下所示：
```
alert(Person.prototype.isPrototypeOf(person1)); //true
alert(Person.prototype.isPrototypeOf(person2)); //true
```
ECMAScript5增加了一个新方法，叫Object.getPrototypeOf()，在所有支持的实现中，这个方法返回[[Prototype]]的值。例如：
```
alert(Object.getPrototypeOf(person1) == Person.prototype); //true
alert(Object.getPrototypeOf(person1).name); //"Nicholas"
```
当为对象实例添加一个属性时，这个属性就会屏蔽原型对象中保存的同名属性；换句话说，添加这个属性只会阻止我们访问原型中的那个属性，但不会修改那个属性。即使将这个属性设置为null，也只会在实例中设置这个属性，而不会恢复其指向原型的连接。不过，使用delete操作符则可以完全删除实例属性，从而让我们能够重新访问原型中的属性使用hasOwnProperty()方法可以检测一个属性是存在于实例中，还是存在于原型中。这个方法（不要忘了它是从Object继承来的）只在给定属性存在于对象实例中时，才会返回true。
```
function Person() {
}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function () {
  alert(this.name);
};

var person1 = new Person();
var person2 = new Person();
alert(person1.hasOwnProperty("name")); //属性中没有,返回false
person1.name = "Greg";
alert(person1.name); //"Greg",来自实例
alert(person1.hasOwnProperty("name")); //true
alert(person2.name); // "Nicholas",来自原型
alert(person2.hasOwnProperty("name")); //false
delete person1.name;
alert(person1.name); //"Nicholas",来自原型
alert(person1.hasOwnProperty("name")); //false
```
#### 3.2、原型与in操作符
有两种方式使用in操作符：单独使用和在for-in循环中使用。
在单独使用时，in操作符会在通过对象能够访问给定属性时返回true，无论该属性存在于实例中还是原型中。
```
function Person() {
}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function () {
  alert(this.name);
};

var person1 = new Person();
var person2 = new Person();
alert(person1.hasOwnProperty("name")); //false
alert("name" in person1); //true
person1.name = "Greg";
alert(person1.name); //"Greg",来自实例
alert(person1.hasOwnProperty("name")); //true
alert("name" in person1); //true
alert(person2.name); //"Nicholas",来自原型
alert(person2.hasOwnProperty("name")); //false
alert("name" in person2); //true
delete person1.name;
alert(person1.name); //"Nicholas",来自原型
alert(person1.hasOwnProperty("name")); //false
alert("name" in person1); //true
```
所以结合hasOwnProperty可以判断一个属性是否存在于原型中。
```
function hasPrototypeProperty(object, name){
  return !object.hasOwnProperty(name) && (name in object);
}
```
for-in循环中使用也能够访问对象原型上的属性和方法。
#### 3.3、更简单的原型语法
```
function Person() {
}
Person.prototype = {
  name: "Nicholas",
  age: 29,
  job: "Software Engineer",
  sayName: function () {
    alert(this.name);
  }
};
```
在上面的代码中，我们将Person.prototype设置为等于一个以对象字面量形式创建的新对象。最终结果相同，但有一个例外：constructor属性不再指向Person了。前面曾经介绍过，每创建一个函数，就会同时创建它的prototype对象，这个对象也会自动获得constructor属性。而我们在这里使用的语法，本质上完全重写了默认的prototype对象，因此constructor属性也就变成了新对象的constructor属性(指向Object构造函数)，不再指向Person函数。
此时，尽管instanceof操作符还能返回正确的结果，但通过constructor已经无法确定对象的类型了，如下所示。
```
var friend = new Person();
alert(friend instanceof Object); //true
alert(friend instanceof Person); //true
alert(friend.constructor == Person); //false
alert(friend.constructor == Object); //true
```
解决办法很简单，手动指定。
```
function Person() {
}
Person.prototype = {
  constructor: Person,
  name: "Nicholas",
  age: 29,
  job: "Software Engineer",
  sayName: function () {
    alert(this.name);
  }
};
```
注意，以这种方式重设constructor属性会导致它的[[Enumerable]]特性被设置为true。默认情况下，原生的constructor属性是不可枚举的，因此如果你使用兼容ECMAScript5的JavaScript引擎，可以试一试Object.defineProperty()。
```
//重设构造函数，只适用于ECMAScript5兼容的浏览器
Object.defineProperty(Person.prototype, "constructor", {
  enumerable: false,
  value: Person
});
```
#### 3.4、原型的动态性
由于在原型中查找值的过程是一次搜索，因此我们对原型对象所做的任何修改都能够立即从实例上反映出来——即使是先创建了实例后修改原型也照样如此
```
var friend = new Person();
Person.prototype.sayHi = function () {
  alert("hi");
};
friend.sayHi(); //"hi"（没有问题!）
```
但是，如果我们使用对象字面量的方式修改Person的原型，此时的Person.protptype指向的是一个新对象。我们知道，调用构造函数时会为实例添加一个指向最初原型的[[Prototype]]指针，而把原型修改为另外一个对象就等于切断了构造函数与最初原型之间的联系。
```
function Person() {
}
var friend = new Person();
Person.prototype = {
  constructor: Person,
  name: "Nicholas",
  age: 29,
  job: "Software Engineer",
  sayName: function () {
    alert(this.name);
  }
};
friend.sayName(); //error
```
过程如下图所示

<img src="/assets/note/js/object2.png">

#### 3.5、原生对象的原型
原型模式的重要性不仅体现在创建自定义类型方面，就连所有原生的引用类型，都是采用这种模式创建的。所有原生引用类型（Object、Array、String 等等）都在其构造函数的原型上定义了方法。例如，在Array.prototype中可以找到sort()方法，而在String.prototype中可以找到substring()方法
```
alert(typeof Array.prototype.sort); //"function"
alert(typeof String.prototype.substring); //"function"
```
通过原生对象的原型，不仅可以取得所有默认方法的引用，而且也可以定义新方法。可以像修改自定义对象的原型一样修改原生对象的原型，因此可以随时添加方法。下面的代码就给基本包装类型String添加了一个名为startsWith()的方法。
```
String.prototype.startsWith = function (text) {
  return this.indexOf(text) == 0;
};
var msg = "Hello world!";
alert(msg.startsWith("Hello")); //true
```
#### 3.6、原型对象的问题
原型模式也不是没有缺点。首先，它省略了为构造函数传递初始化参数这一环节，结果所有实例在默认情况下都将取得相同的属性值。 虽然这会在某种程度上带来一些不方便，但还不是原型的最大问题。原型模式的最大问题是由其共享的本性所导致的。尤其对于包含引用类型值的属性来说（基本值还可以隐藏）
```
function Person(){
}
Person.prototype = {
  constructor: Person,
  name : "Nicholas",
  age : 29,
  job : "Software Engineer",
  friends : ["Shelby", "Court"],
  sayName : function () {
    alert(this.name);
  }
};
var person1 = new Person();
var person2 = new Person();
person1.friends.push("Van");
alert(person1.friends); //"Shelby,Court,Van"
alert(person2.friends); //"Shelby,Court,Van"
alert(person1.friends === person2.friends); //true
```
### 四、组合使用构造函数和原型模式
创建自定义类型的最常见方式，就是组合使用构造函数模式与原型模式。构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。结果，每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度地节省了内存。另外，这种混成模式还支持向构造函数传递参数；可谓是集两种模式之长。
```
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.friends = ["Shelby", "Court"];
}
Person.prototype = {
  constructor: Person,
  sayName: function () {
    alert(this.name);
  }
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
person1.friends.push("Van");
alert(person1.friends); //"Shelby,Count,Van"
alert(person2.friends); //"Shelby,Count"
alert(person1.friends === person2.friends); //false
alert(person1.sayName === person2.sayName); //true
```
这种模式是目前在ECMAScript中使用最广泛、认同度最高的一种创建自定义类型的方法。
### 五、动态原型模式
动态原型模式致力于解决这样一个问题，它把所有信息都封装在了构造函数中，而通过在构造函数中初始化原型（仅在必要的情况下），又保持了同时使用构造函数和原型的优点。换句话说，可以通过检查某个应该存在的方法是否有效，来决定是否需要初始化原型。
```
function Person(name, age, job) {
  //属性
  this.name = name;
  this.age = age;
  this.job = job;
  //方法
  if (typeof this.sayName != "function") {
    Person.prototype.sayName = function () {
      alert(this.name);
    };
  }
}
var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();
```
这里只在sayName()方法不存在的情况下，才会将它添加到原型中。使用动态原型模式时，不能使用对象字面量重写原型。前面已经解释过了，如果在已经创建了实例的情况下重写原型，那么就会切断现有实例与新原型之间的联系。
### 六、寄生构造函数模式
通常，在前述的几种模式都不适用的情况下，可以使用寄生（parasitic）构造函数模式。这种模式的基本思想是创建一个函数，该函数的作用仅仅是封装创建对象的代码，然后再返回新创建的对象；但从表面上看，这个函数又很像是典型的构造函数。
```
function Person(name, age, job) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function () {
    alert(this.name);
  };
  return o;
}
var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName(); //"Nicholas"
```
除了使用new操作符并把使用的包装函数叫做构造函数之外，这个模式跟工厂模式其实是一模一样的。工厂模式为：
```
function createPerson(name, age, job) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function () {
    alert(this.name);
  };
  return o;
}
var person1 = createPerson("Nicholas", 29, "Software Engineer");
var person2 = createPerson("Greg", 27, "Doctor");
```
关于寄生构造函数模式，有一点需要说明：首先，返回的对象与构造函数或者与构造函数的原型属性之间没有关系；也就是说，构造函数返回的对象与在构造函数外部创建的对象没有什么不同。为此，不能依赖 instanceof 操作符来确定对象类型。由于存在上述问题，我们建议在可以使用其他模式的情况下，不要使用这种模式。
### 七、稳妥构造函数模式
所谓稳妥对象，指的是没有公共属性，而且其方法也不引用this的对象。稳妥对象最适合在一些安全的环境中（这些环境中会禁止使用this和new），或者在防止数据被其他应用程序改动时使用。稳妥构造函数遵循与寄生构造函数类似的模式，但有两点不同：
- 1、是新创建对象的实例方法不引用this；
- 2、是不使用new操作符调用构造函数。按照稳妥构造函数的要求，可以将前面的Person构造函数重写如下。

```
function Person(name, age, job){
  //创建要返回的对象
  var o = new Object();
  //可以在这里定义私有变量和函数
  //添加方法
  o.sayName = function(){
    alert(name);
  };
  //返回对象
  return o;
}
var friend = Person("Nicholas", 29, "Software Engineer");
friend.sayName(); //"Nicholas"
```
注意，在以这种模式创建的对象中，除了使用sayName()方法之外，没有其他办法访问name的值。与寄生构造函数模式类似，使用稳妥构造函数模式创建的对象与构造函数之间也没有什么关系，因此instanceof操作符对这种对象也没有意义。