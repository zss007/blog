---
title: es6 之数值扩展
categories:
- note
---
ES6 扩展了数值。
<!--more-->
### 一、Number.isFinite() & Number.isNaN()
ES6 在 Number 对象上，新提供了 Number.isFinite() 和 Number.isNaN() 两个方法。    
Number.isFinite() 用来检查一个数值是否为有限的（finite），即不是 Infinity。注意，如果参数类型不是数值，Number.isFinite 一律返回 false。    
Number.isNaN() 用来检查一个值是否为 NaN。注意，如果参数类型不是数值，Number.isNaN 一律返回 false。
```
Number.isFinite(15); // true
Number.isFinite(0.8); // true
Number.isFinite(NaN); // false
Number.isFinite(Infinity); // false
Number.isFinite(-Infinity); // false
Number.isFinite('foo'); // false
Number.isFinite('15'); // false
Number.isFinite(true); // false
      
Number.isNaN(NaN) // true
Number.isNaN(15) // false
Number.isNaN('15') // false
Number.isNaN(true) // false
Number.isNaN(9/NaN) // true
Number.isNaN('true' / 0) // true
Number.isNaN('true' / 'true') // true
```
它们与传统的全局方法 isFinite() 和 isNaN() 的区别在于，传统方法先调用 Number() 将非数值的值转为数值，再进行判断，而这两个新方法只对数值有效，Number.isFinite() 对于非数值一律返回 false, Number.isNaN() 只有对于 NaN 才返回 true，非 NaN 一律返回 false。
### 二、Number.parseInt(), Number.parseFloat()
ES6 将全局方法 parseInt() 和 parseFloat()，移植到 Number 对象上面，行为完全保持不变。这样做的目的，是逐步减少全局性方法，使得语言逐步模块化。
```
// ES5的写法
parseInt('12.34') // 12
parseFloat('123.45#') // 123.45
    
// ES6的写法
Number.parseInt('12.34') // 12
Number.parseFloat('123.45#') // 123.45
    
Number.parseInt === parseInt // true
Number.parseFloat === parseFloat // true
```
### 三、Number.isInteger()
Number.isInteger() 用来判断一个数值是否为整数。
```
Number.isInteger(25) // true
Number.isInteger(25.1) // false
    
// JavaScript 内部，整数和浮点数采用的是同样的储存方法，所以 25 和 25.0 被视为同一个值
Number.isInteger(25) // true
Number.isInteger(25.0) // true
    
// 如果参数不是数值，Number.isInteger 返回 false。
Number.isInteger() // false
Number.isInteger(null) // false
Number.isInteger('15') // false
Number.isInteger(true) // false
    
// 5E-325 由于值太小，会被自动转为 0，因此返回 true。如果对数据精度的要求较高，不建议使用 Number.isInteger() 判断一个数值是否为整数
Number.isInteger(5E-324) // false
Number.isInteger(5E-325) // true
```
### 四、Number.EPSILON
ES6 在 Number 对象上面，新增一个极小的常量 Number.EPSILON。根据规格，它表示 1 与大于 1 的最小浮点数之间的差。对于 64 位浮点数来说，大于 1 的最小浮点数相当于二进制的1.00..001，小数点后面有连续 51 个零。这个值减去 1 之后，就等于 2 的 -52 次方。Number.EPSILON 实际上是 JavaScript 能够表示的最小精度。误差如果小于这个值，就可以认为已经没有意义了，即不存在误差了。引入一个这么小的量的目的，在于为浮点数计算，设置一个误差范围。我们知道浮点数计算是不精确的，Number.EPSILON 可以用来设置 “能够接受的误差范围”。
```
Number.EPSILON === Math.pow(2, -52)
// true
Number.EPSILON
// 2.220446049250313e-16
    
5.551115123125783e-17 < Number.EPSILON
// true
```
### 五、Number.isSafeInteger()
JavaScript 能够准确表示的整数范围在 -2^53 到 2^53 之间（不含两个端点），超过这个范围，无法精确表示这个值。ES6 引入了 Number.MAX_SAFE_INTEGER 和 
Number.MIN_SAFE_INTEGER 这两个常量，用来表示这个范围的上下限。
```
Number.MAX_SAFE_INTEGER === Math.pow(2, 53) - 1
// true
Number.MAX_SAFE_INTEGER === 9007199254740991
// true
    
Number.MIN_SAFE_INTEGER === -Number.MAX_SAFE_INTEGER
// true
Number.MIN_SAFE_INTEGER === -9007199254740991
// true
    
// Number.isSafeInteger() 则是用来判断一个整数是否落在这个范围之内
Number.isSafeInteger('a') // false
Number.isSafeInteger(null) // false
Number.isSafeInteger(NaN) // false
Number.isSafeInteger(Infinity) // false
Number.isSafeInteger(-Infinity) // false
Number.isSafeInteger(3) // true
Number.isSafeInteger(1.2) // false
Number.isSafeInteger(9007199254740990) // true
Number.isSafeInteger(9007199254740992) // false
Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1) // false
Number.isSafeInteger(Number.MIN_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1) // false
    
// 使用这个函数时，不要只验证运算结果，而要同时验证参与运算的每个值
Number.isSafeInteger(9007199254740993)
// false
Number.isSafeInteger(990)
// true
Number.isSafeInteger(9007199254740993 - 990)
// true
9007199254740993 - 990
// 返回结果 9007199254740002
// 正确答案应该是 9007199254740003
```
### 六、Math 对象的扩展
#### 1、Math.trunc()
Math.trunc 方法用于去除一个数的小数部分，返回整数部分
```
Math.trunc(4.1) // 4
Math.trunc(4.9) // 4
Math.trunc(-4.1) // -4
Math.trunc(-4.9) // -4
Math.trunc(-0.1234) // -0
    
// 对于非数值，Math.trunc 内部使用 Number 方法将其先转为数值
Math.trunc('123.456') // 123
Math.trunc(true) //1
Math.trunc(false) // 0
Math.trunc(null) // 0
    
// 对于空值和无法截取整数的值，返回 NaN
Math.trunc(NaN);      // NaN
Math.trunc('foo');    // NaN
Math.trunc();         // NaN
Math.trunc(undefined) // NaN
```
#### 2、Math.sign()
Math.sign 方法用来判断一个数到底是正数、负数、还是零。对于非数值，会先将其转换为数值。参数为正数，返回 +1；参数为负数，返回 -1；参数为 0，返回 0；参数为 -0，返回 -0；其他值，返回 NaN。
```
Math.sign(-5) // -1
Math.sign(5) // +1
Math.sign(0) // +0
Math.sign(-0) // -0
Math.sign(NaN) // NaN
    
// 如果参数是非数值，会自动转为数值。对于那些无法转为数值的值，会返回 NaN
Math.sign('')  // 0
Math.sign(true)  // +1
Math.sign(false)  // 0
Math.sign(null)  // 0
Math.sign('9')  // +1
Math.sign('foo')  // NaN
Math.sign()  // NaN
Math.sign(undefined)  // NaN
```
#### 3、Math.cbrt()
Math.cbrt 方法用于计算一个数的立方根。
```
Math.cbrt(-1) // -1
Math.cbrt(0)  // 0
Math.cbrt(1)  // 1
Math.cbrt(2)  // 1.2599210498948734
    
// 对于非数值，Math.cbrt方法内部也是先使用Number方法将其转为数值
Math.cbrt('8') // 2
Math.cbrt('hello') // NaN
```
#### 4、Math.clz32()
JavaScript 的整数使用 32 位二进制形式表示，Math.clz32 方法返回一个数的 32 位无符号整数形式有多少个前导 0。
```
Math.clz32(0) // 32
Math.clz32(1) // 31
Math.clz32(1000) // 22
Math.clz32(0b01000000000000000000000000000000) // 1
Math.clz32(0b00100000000000000000000000000000) // 2
    
// 对于小数，Math.clz32方法只考虑整数部分
Math.clz32(3.2) // 30
Math.clz32(3.9) // 30
    
// 对于空值或其他类型的值，Math.clz32方法会将它们先转为数值，然后再计算
Math.clz32() // 32
Math.clz32(NaN) // 32
Math.clz32(Infinity) // 32
Math.clz32(null) // 32
Math.clz32('foo') // 32
Math.clz32([]) // 32
Math.clz32({}) // 32
Math.clz32(true) // 31
```
#### 5、Math.imul()
Math.imul 方法返回两个数以 32 位带符号整数形式相乘的结果，返回的也是一个 32 位的带符号整数。
```
Math.imul(2, 4)   // 8
Math.imul(-1, 8)  // -8
Math.imul(-2, -2) // 4
    
// Math.imul(a, b)与a * b的结果是相同的。因为 JavaScript 有精度限制，超过 2 的 53 次方的值无法精确表示，Math.imul 方法可以返回正确的低位数值。
(0x7fffffff * 0x7fffffff)|0 // 0
// 因为它们的乘积超过了 2 的 53 次方，JavaScript 无法保存额外的精度，就把低位的值都变成了 0。Math.imul 方法可以返回正确的值 1
Math.imul(0x7fffffff, 0x7fffffff) // 1
```
#### 6、Math.fround()
Math.fround 方法返回一个数的32位单精度浮点数形式。Math.fround 方法的主要作用，是将 64 位双精度浮点数转为 32 位单精度浮点数。如果小数的精度超过 24 个二进制位，返回值就会不同于原值，否则返回值不变（即与 64 位双精度值一致）。
```
// 未丢失有效精度
Math.fround(1.125) // 1.125
Math.fround(7.25)  // 7.25
    
// 丢失精度
Math.fround(0.3)   // 0.30000001192092896
Math.fround(0.7)   // 0.699999988079071
Math.fround(1.0000000123) // 1
    
// 对于 NaN 和 Infinity，此方法返回原值
Math.fround(NaN)      // NaN
Math.fround(Infinity) // Infinity
    
// 其它类型的非数值，Math.fround 方法会先将其转为数值，再返回单精度浮点数
Math.fround('5')      // 5
Math.fround(true)     // 1
Math.fround(null)     // 0
Math.fround([])       // 0
Math.fround({})       // NaN
```
#### 7、Math.hypot()
Math.hypot 方法返回所有参数的平方和的平方根。
```
Math.hypot(3, 4);        // 5
Math.hypot(3, 4, 5);     // 7.0710678118654755
Math.hypot();            // 0
// 参数不是数值，Math.hypot 方法会将其转为数值。只要有一个参数无法转为数值，就会返回 NaN
Math.hypot(NaN);         // NaN
Math.hypot(3, 4, 'foo'); // NaN
Math.hypot(3, 4, '5');   // 7.0710678118654755
Math.hypot(-3);          // 3
```
#### 8、Math.expm1(), Math.log1p(), Math.log10(), Math.log2()
```
// Math.expm1(x) 返回 Math.pow(e, x) - 1，即 Math.exp(x) - 1。
Math.expm1(-1) // -0.6321205588285577
Math.expm1(0)  // 0
Math.expm1(1)  // 1.718281828459045
    
// Math.log1p(x) 方法返回 1 + x 的自然对数，即 Math.log(1 + x)。如果 x 小于 -1，返回 NaN
Math.log1p(1)  // 0.6931471805599453
Math.log1p(0)  // 0
Math.log1p(-1) // -Infinity
Math.log1p(-2) // NaN
    
// Math.log10(x) 返回以 10 为底的 x 的对数。如果 x 小于 0，则返回 NaN
Math.log10(2)      // 0.3010299956639812
Math.log10(1)      // 0
Math.log10(0)      // -Infinity
Math.log10(-2)     // NaN
Math.log10(100000) // 5
    
// Math.log2(x) 返回以 2 为底的 x 的对数。如果 x 小于 0，则返回 NaN
Math.log2(3)       // 1.584962500721156
Math.log2(2)       // 1
Math.log2(1)       // 0
Math.log2(0)       // -Infinity
Math.log2(-2)      // NaN
Math.log2(1024)    // 10
Math.log2(1 << 29) // 29
```
#### 9、双曲函数方法
Math.sinh(x) 返回x的双曲正弦（hyperbolic sine）    
Math.cosh(x) 返回x的双曲余弦（hyperbolic cosine）    
Math.tanh(x) 返回x的双曲正切（hyperbolic tangent）     
Math.asinh(x) 返回x的反双曲正弦（inverse hyperbolic sine）    
Math.acosh(x) 返回x的反双曲余弦（inverse hyperbolic cosine）    
Math.atanh(x) 返回x的反双曲正切（inverse hyperbolic tangent）
#### 10、指数运算符
```
2 ** 2 // 4
2 ** 3 // 8
    
// 指数运算符可以与等号结合，形成一个新的赋值运算符（**=）
let a = 1.5;
a **= 2;
// 等同于 a = a * a;
let b = 4;
b **= 3;
// 等同于 b = b * b * b;
    
// 在 V8 引擎中，指数运算符与 Math.pow 的实现不相同，对于特别大的运算结果，两者会有细微的差异
Math.pow(99, 99)
// 3.697296376497263e+197
99 ** 99
// 3.697296376497268e+197
// 两个运算结果的最后一位有效数字是有差异的
```
ES2016 新增了一个指数运算符（**）