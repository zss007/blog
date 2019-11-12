---
title: js 之数组方法
categories:
- note
---
最近研究zepto源码的时候发现一些非常好用的数组方法，这些就是es5新增的数组方法，而w3c这些网站并没有这些方法介绍，可能是太久没更新维护了。总结了一下，给这些方法分了类，大体如下：
- 5个迭代方法：forEach()、map()、filter()、some()、every()；
- 2个索引方法：indexOf() 和 lastIndexOf()；
- 2个归并方法：reduce()、reduceRight()；
<!--more-->

### 一、迭代方法
这些方法都接收两个参数，第一个参数是一个函数，它接收三个参数，数组当前项的值、当前项在数组中的索引、数组对象本身。第二个参数是执行第一个函数参数的作用域对象，也就是上面说的函数中this所指向的值。注意，这几种方法都不会改变原数组。
#### 1.1、forEach(即遍历)
```
var arr = [1, 2, 3, 4];
[4, 3, 2, 1].forEach(function (item, index, array) {
  console.log('item', item);    //打印当前值(4,3,2,1)
  console.log('index', index);  //打印索引值(0,1,2,3)
  console.log('array', array);  //[4, 3, 2, 1]
  console.log('this', this);    //如果有arr参数值的话，打印[1, 2, 3, 4]，否则打印window对象
}, arr);

var arr2 = [5, 6, 7, 8];
[].forEach.call(arr, function (item, index, array) {
  console.log('item', item);    //打印当前值(1,2,3,4)
  console.log('index', index);  //打印索引值(0,1,2,3)
  console.log('array', array);  //[1, 2, 3, 4]
  console.log('this', this);    //如果有arr2参数值的话，打印[5, 6, 7, 8]，否则打印window对象
}, arr2);
```
这里把方法的作用域对象也讲下，后面的例子就不重复了。如果这第2个可选参数不指定，则使用全局对象代替（在浏览器是为window），严格模式下甚至是undefined。
而且删除数组中某项后，forEach遍历数组也有点不同。
```
var array = [1, 2, 3];

delete array[1]; // 移除 2
console.log(array); // 打印"1,,3"
console.log(array[1]);//打印undefined

console.log(array.length); //长度仍然是3

array.forEach(alert); // 弹出的仅仅是1和3
array.forEach(console.log); //打印1 0 [1,,3]、3 2 [1,,3]
```
#### 1.2、map(映射)
```
var data = [1, 2, 3, 4];

var arrayOfSquares = data.map(function (item) {
    return item * item;
});

console.log(arrayOfSquares); //打印[1, 4, 9, 16]
```
map是一一对应的关系，如果没有return的话，则返回一个跟原数组相同长度的undefined数组。在实际使用的时候，我们可以利用map方法方便提取对象数组中的特定属性值们。
```
var users = [
  {name: "张三", "age": "16"},
  {name: "李四", "age": "17"},
  {name: "王五", "age": "18"}
];

var names = users.map(function (user) {
  return user.name;
});

console.log('names', names); //打印names ["张三", "李四", "王五"]
```
#### 1.3、filter(过滤)
```
var users = [
  {name: "张三", "age": "16"},
  {name: "李四", "age": "17"},
  {name: "王五", "age": "18"}
];

var fullAgeUser = users.filter(function (user) {
  if (user.age >= 17) {
      return user;
  }
});

console.log('fullAgeUser', JSON.stringify(fullAgeUser)); //打印fullAgeUser [{"name":"李四","age":"17"},{"name":"王五","age":"18"}]
```
过滤同map相似，不过没有return的话，返回的数组就没有该项，常用于筛选部分符合条件的数组。
#### 1.4、some(某项满足)
```
var users = [
  {name: "张三", "age": "16"},
  {name: "李四", "age": "17"},
  {name: "王五", "age": "18"}
];

var fullAgeResult = users.some(function (user) {
  console.log(JSON.stringify(user));
  return user.age >= 17;
});

console.log('fullAgeResult', fullAgeResult); //打印true
```
some遍历数组，如果有某项满足条件，则跳出循环不再执行，返回true；否则返回false。
#### 1.5、every(所有项满足)
```
var users = [
  {name: "张三", "age": "16"},
  {name: "李四", "age": "17"},
  {name: "王五", "age": "18"}
];

var fullAgeResult = users.every(function (user) {
  console.log(JSON.stringify(user));
  return user.age >= 16;
});

console.log('fullAgeResult', fullAgeResult); //打印true
```
every遍历数组，如果某项没有满足条件，则跳出循环不再执行，返回false；如果所有项都满足则返回true。
### 二、索引方法
#### 2.1、indexOf(索引)
indexOf方法在字符串中一直都用，数组这里的indexOf方法很类似:array.indexOf(searchElement[, fromIndex])。
```
var array = [1, 4, 6, 2, 4];
console.log(array.indexOf(4, "x")); // 1 ("x"被忽略)
console.log(array.indexOf(4, "2")); // 4 (从2号位开始搜索)

console.log(array.indexOf(3)); // -1 (未找到)
console.log(array.indexOf("4")); // -1 (未找到，因为4 !== "4")
```
返回整数索引值，如果没有匹配（进行严格匹配），返回-1。fromIndex可选，表示从这个位置开始搜索，若缺省或格式不合要求，使用默认值0，fromIndex使用字符串数值也是可以的，例如"3"和3都可以，没有进行严格匹配。
#### 2.2、lastIndexOf(倒序查找)
```
var array = [1, 6, 4, 2, 4];
console.log(array.lastIndexOf(4));  //4
console.log(array.lastIndexOf(4, 1));  //-1
```
倒序查找，如果制定了fromIndex，则从fromIndex往前查找。
### 三、归并方法
#### 3.1、reduce(合并)
```
var result = [1, 2, 3, 4].reduce(function (previous, current, index, array) {
  console.log('previous', previous); //1,3,6
  return previous + current;
});
console.log(result);  //10

result = [1, 2, 3, 4].reduce(function (previous, current, index, array) {
  console.log('previous', previous); //3,4,6,9
  return previous + current;
}, 3);
console.log(result);  //13
```
array.reduce(callback[, initialValue])，如果不存在initialValue的话，则第一个previous是数组第一项。最后两个参数为索引值index以及数组本身。
执行过程：
```
// 初始设置
previous = initialValue = 3, current = 1

// 第一次迭代
previous = (3 + 1) =  4, current = 2

// 第二次迭代
previous = (4 + 2) =  6, current = 3

// 第三次迭代(退出循环)
previous = (6 + 3) =  9, current = 4
```
实际应用当中，我们可以实现二维数组的扁平化(即转化为一维数组)。
```
var matrix = [
  [1, 2],
  [3, 4],
  [5, 6]
];

console.log(matrix.reduce(function (prev, curr) {   //[1, 2, 3, 4, 5, 6]
  return prev.concat(curr);
}));
```
#### 3.2、reduceRight(倒序合并)
```
var matrix = [
  [1, 2],
  [3, 4],
  [5, 6]
];

console.log(matrix.reduceRight(function (prev, curr) {   //[5, 6, 3, 4, 1, 2]
  return prev.concat(curr);
}));
```
与reduce极为相似。