---
title: js 之类型转换
categories:
- note
---
JS 基础常考内容，一定要融会贯通。
<!--more-->
### 一、数据类型
JS 中有 7 中数据类型：
原始数据类型： Null、Boolean、String、Number、Symbol、Undefined
复合数据类型： Object
### 二、显式类型转换
先说明下面几个函数：
valueOf(): 将该对象原始值返回
toString(): 将该对象原始值以字符串返回
- Number 转换

```
// 基本数据转换
Number(1) // 数字： 数字 -> 数字
Number('123') // 字符串： 纯数字 -> 数字，不是纯数字 -> NaN，空字符串 -> 0
Number(true) // 布尔值： true -> 1，false -> 0
Number(undefined) // NaN
Number(null) // 0
// 复合类型数据转换
// 先调用 valueOf，如果输出为基本类型，则调用 Number，如果为复合类型，继续调用 toString，如果还不是基本数据类型就报错，如果是则调用 Number
```
- String 转换

```
// String() 很简单，就是将这些基本类型都变成字符串：
// 123 -> '123'
// true -> 'true'
// undefined -> 'undefined'
// null -> 'null'
// 复合类型转换
// 先调用 toString，如果输出为基本类型，则调用 String，如果为复合类型，继续调用 valueOf，如果还不是基本数据类型就报错，如果是则调用 String
```
- Boolean 转换
+0、-0、undefined、null、''、NaN -> false，其余都为 true

### 三、显式类型转换
- 四则运算
- 判断语句
- Native 调用: 比如说 console.log()、alert()

```
// Boolean()
![]  // false
!![] // true
!{}  // false
!!{} // true
[]+[] // ""，调用 String 处理 []
[]+1 // "1"，调用 String 处理 []
{}+{} // "[object Object][object Object]"，这里 chrome 和 firefox 解释不同
{}+[] // 0，这里 {} 被当前代码块，不做任何处理，然后调用 Number 处理 []
[]+{} // "[object Object]"，调用 String 处理 [] 和 {}
1+{} // "1[object Object]"，调用 {}
```
