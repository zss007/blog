---
title: es6 之字符串扩展
categories:
- es6
---
ES6 扩展了字符串对象。
<!--more-->
### 一、codePointAt()
JavaScript 内部，字符以 UTF-16 的格式储存，每个字符固定为2个字节。对于那些需要4个字节储存的字符（Unicode 码点大于0xFFFF的字符），JavaScript 会认为它们是两个字符。ES6 提供了codePointAt方法，能够正确处理 4 个字节储存的字符，返回一个字符的码点；对于那些两个字节储存的常规字符，它的返回结果与charCodeAt方法相同。
```
// 汉字“𠮷”（注意，这个字不是“吉祥”的“吉”）的码点是0x20BB7，UTF-16 编码为0xD842 0xDFB7（十进制为55362 57271），需要4个字节储存。
var s = "𠮷a";
// JavaScript 不能正确处理，字符串长度会误判为2
s.length // 2
// charAt方法无法读取整个字符
s.charAt(0) // ''
s.charAt(1) // ''
// charCodeAt方法只能分别返回前两个字节和后两个字节的值
s.charCodeAt(0) // 55362
s.charCodeAt(1) // 57271
s.codePointAt(2) // 97
    
let s = '𠮷a';
// JavaScript 将“𠮷a”视为三个字符，codePointAt 方法在第一个字符上，正确地识别了“𠮷”，返回了它的十进制码点 134071（即十六进制的20BB7）
s.codePointAt(0) // 134071
// 在第二个字符（即“𠮷”的后两个字节）和第三个字符“a”上，codePointAt方法的结果与charCodeAt方法相同
s.codePointAt(1) // 57271
s.codePointAt(2) // 97
    
let s = '𠮷a';
// codePointAt方法返回的是码点的十进制值，如果想要十六进制的值，可以使用 toString 方法转换一下
s.codePointAt(0).toString(16) // "20bb7"
s.codePointAt(2).toString(16) // "61"
    
// 解决字符 a 在字符串 s 的位置序号问题
let s = '𠮷a';
for (let ch of s) {
  console.log(ch.codePointAt(0).toString(16));
}
// 20bb7
// 61
    
// 测试一个字符由两个字节还是由四个字节组成的最简单方法
function is32Bit(c) {
  return c.codePointAt(0) > 0xFFFF;
}
is32Bit("𠮷") // true
is32Bit("a") // false
```
### 二、String.fromCodePoint()
ES5 提供String.fromCharCode方法，用于从码点返回对应字符，但是这个方法不能识别 32 位的 UTF-16 字符（Unicode 编号大于0xFFFF）。     
ES6 提供了String.fromCodePoint方法，可以识别大于0xFFFF的字符，弥补了String.fromCharCode方法的不足。在作用上，正好与codePointAt方法相反。
注意，fromCodePoint方法定义在String对象上，而codePointAt方法定义在字符串的实例对象上。
```
String.fromCharCode(0x20BB7)
// "ஷ"，String.fromCharCode不能识别大于0xFFFF的码点，所以0x20BB7就发生了溢出，最高位2被舍弃了，最后返回码点U+0BB7对应的字符，而不是码点U+20BB7对应的字符。
    
String.fromCodePoint(0x20BB7)
// "𠮷"
String.fromCodePoint(0x78, 0x1f680, 0x79) === 'x\uD83D\uDE80y'
// true，如果String.fromCodePoint方法有多个参数，则它们会被合并成一个字符串返回。
```
### 三、字符串的遍历器接口
ES6 为字符串添加了遍历器接口，使得字符串可以被for...of循环遍历。
```
for (let codePoint of 'foo') {
  console.log(codePoint)
}
// "f"
// "o"
// "o"
      
// 除了遍历字符串，这个遍历器最大的优点是可以识别大于0xFFFF的码点，传统的for循环无法识别这样的码点
let text = String.fromCodePoint(0x20BB7);
// 字符串text只有一个字符，但是for循环会认为它包含两个字符（都不可打印）
for (let i = 0; i < text.length; i++) {
  console.log(text[i]);
}
// " "
// " "
for (let i of text) {
  console.log(i);
}
// "𠮷"，而for...of循环会正确识别出这一个字符
```
### 四、normalize()
许多欧洲语言有语调符号和重音符号。为了表示它们，Unicode 提供了两种方法。一种是直接提供带重音符号的字符，比如Ǒ（\u01D1）。另一种是提供合成符号（combining character），即原字符与重音符号的合成，两个字符合成一个字符，比如O（\u004F）和ˇ（\u030C）合成Ǒ（\u004F\u030C）。这两种表示方法，在视觉和语义上都等价，但是 JavaScript 不能识别。      
ES6 提供字符串实例的normalize()方法，用来将字符的不同表示方法统一为同样的形式，这称为 Unicode 正规化。
```
'\u01D1'==='\u004F\u030C' //false
'\u01D1'.length // 1
'\u004F\u030C'.length // 2
      
'\u01D1'.normalize() === '\u004F\u030C'.normalize()
// true
```
### 五、includes(), startsWith(), endsWith()
传统上，JavaScript 只有 indexOf 方法，可以用来确定一个字符串是否包含在另一个字符串中。ES6 又提供了三种新方法。   
1、includes()：返回布尔值，表示是否找到了参数字符串。    
2、startsWith()：返回布尔值，表示参数字符串是否在原字符串的头部。   
3、endsWith()：返回布尔值，表示参数字符串是否在原字符串的尾部。   
支持第二个参数，表示开始搜索的位置。   
```
let s = 'Hello world!';
    
s.startsWith('Hello') // true
s.endsWith('!') // true
s.includes('o') // true
    
// 使用第二个参数 n 时，endsWith 的行为与其他两个方法有所不同。它针对前 n 个字符，而其他两个方法针对从第 n 个位置直到字符串结束
s.startsWith('world', 6) // true
s.endsWith('Hello', 5) // true
s.includes('Hello', 6) // false
```
### 六、repeat()
repeat 方法返回一个新字符串，表示将原字符串重复 n 次。
```
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0) // ""
    
// 参数如果是小数，会被取整。
'na'.repeat(2.9) // "nana"
    
// 如果 repeat 的参数是负数或者 Infinity，会报错。
'na'.repeat(Infinity)
// RangeError
'na'.repeat(-1)
// RangeError
    
// 如果参数是 0 到-1 之间的小数，则等同于 0，这是因为会先进行取整运算。0 到-1 之间的小数，取整以后等于 -0，repeat 视同为 0。
'na'.repeat(-0.9) // ""
    
// 参数 NaN 等同于 0。
'na'.repeat(NaN) // ""
    
// 如果 repeat 的参数是字符串，则会先转换成数字。
'na'.repeat('na') // ""
'na'.repeat('3') // "nanana"
```
### 七、padStart()，padEnd()
ES2017 引入了字符串补全长度的功能。如果某个字符串不够指定长度，会在头部或尾部补全。padStart() 用于头部补全，padEnd() 用于尾部补全。一共接受两个参数，第一个参数用来指定字符串的最小长度，第二个参数是用来补全的字符串。
```
'x'.padStart(5, 'ab') // 'ababx'
'x'.padStart(4, 'ab') // 'abax'
'x'.padEnd(5, 'ab') // 'xabab'
'x'.padEnd(4, 'ab') // 'xaba'

// 如果原字符串的长度，等于或大于指定的最小长度，则返回原字符串。
'xxx'.padStart(2, 'ab') // 'xxx'
'xxx'.padEnd(2, 'ab') // 'xxx'
    
// 如果用来补全的字符串与原字符串，两者的长度之和超过了指定的最小长度，则会截去超出位数的补全字符串
'abc'.padStart(10, '0123456789')
// '0123456abc'
    
// 如果省略第二个参数，默认使用空格补全长度
'x'.padStart(4) // '   x'
'x'.padEnd(4) // 'x   '
            
// padStart 的常见用途是为数值补全指定位数。下面代码生成 10 位的数值字符串
'1'.padStart(10, '0') // "0000000001"
'12'.padStart(10, '0') // "0000000012"
'123456'.padStart(10, '0') // "0000123456"
    
// 另一个用途是提示字符串格式
'12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
'09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
```
### 八、模板字符串
模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。
```
// 普通字符串
`In JavaScript '\n' is a line-feed.`
    
// 多行字符串
`In JavaScript this is
 not legal.`
    
console.log(`string text line 1
string text line 2`);
    
// 字符串中嵌入变量
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
    
// 如果在模板字符串中需要使用反引号，则前面要用反斜杠转义
let greeting = `\`Yo\` World!`;
    
// 如果使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中。如果你不想要这个换行，可以使用 trim 方法消除它
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`.trim());
    
// 模板字符串中嵌入变量，需要将变量名写在 ${} 之中。大括号内部可以放入任意的 JavaScript 表达式，可以进行运算，以及引用对象属性。
let x = 1;
let y = 2;
`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"
`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"
let obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"
    
// 模板字符串之中还能调用函数
function fn() {
  return "Hello World";
}
`foo ${fn()} bar`
// foo Hello World bar
    
// 如果大括号中的值不是字符串，将按照一般的规则转为字符串。比如，大括号中是一个对象，将默认调用对象的 toString 方法。如果模板字符串中的变量没有声明，将报错。
let msg = `Hello, ${place}`;
// 报错，变量 place 没有声明
    
// 由于模板字符串的大括号内部，就是执行 JavaScript 代码，因此如果大括号内部是一个字符串，将会原样输出。
`Hello ${'World'}`
// "Hello World"
    
// 模板字符串甚至还能嵌套
const tmpl = addrs => `
  <table>
  ${addrs.map(addr => `
    <tr><td>${addr.first}</td></tr>
    <tr><td>${addr.last}</td></tr>
  `).join('')}
  </table>
`;
const data = [
    { first: '<Jane>', last: 'Bond' },
    { first: 'Lars', last: '<Croft>' },
];
console.log(tmpl(data));
// <table>
//
//   <tr><td><Jane></td></tr>
//   <tr><td>Bond</td></tr>
//
//   <tr><td>Lars</td></tr>
//   <tr><td><Croft></td></tr>
//
// </table>
    
// 如果需要引用模板字符串本身，在需要时执行
// 写法一
let str = 'return ' + '`Hello ${name}!`';
let func = new Function('name', str);
func('Jack') // "Hello Jack!"

// 写法二
let str = '(name) => `Hello ${name}!`';
let func = eval.call(null, str);
func('Jack') // "Hello Jack!"
```
### 九、标签模板
模板字符串可以紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串。这被称为“标签模板”功能（tagged template）。
```
// 标签模板其实不是模板，而是函数调用的一种特殊形式。“标签”指的就是函数，紧跟在后面的模板字符串就是它的参数
alert`123`
// 等同于
alert(123)
    
// 如果模板字符里面有变量，就不是简单的调用了，而是会将模板字符串先处理成多个参数，再调用函数
let a = 5;
let b = 10;
tag`Hello ${ a + b } world ${ a * b }`;
// 等同于
tag(['Hello ', ' world ', ''], 15, 50);
// 我们可以按照需要编写 tag 函数的代码
function tag(s, v1, v2) {
  console.log(s[0]);
  console.log(s[1]);
  console.log(s[2]);
  console.log(v1);
  console.log(v2);

  return "OK";
}
// "Hello "
// " world "
// ""
// 15
// 50
// "OK"
    
// “标签模板”的一个重要应用，就是过滤 HTML 字符串，防止用户输入恶意内容。
let message = SaferHTML`<p>${sender} has sent you a message.</p>`;
function SaferHTML(templateData) {
  let s = templateData[0];
  for (let i = 1; i < arguments.length; i++) {
    let arg = String(arguments[i]);

    // Escape special characters in the substitution.
    s += arg.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

    // Don't escape special characters in the template.
    s += templateData[i];
  }
  return s;
}
    
// 模板处理函数的第一个参数（模板字符串数组），还有一个 raw 属性。接受的参数，实际上是一个数组，该数组有一个 raw 属性，保存的是转义后的原字符串
console.log`123`
    
// tag函数的第一个参数 strings，有一个 raw 属性，也指向一个数组，数组中字符串里面的斜杠都被转义了
tag`First line\nSecond line`
function tag(strings) {
  console.log(strings.raw[0]);
  // strings.raw[0] 为 "First line\\nSecond line"
  // 打印输出 "First line\nSecond line"
}
```
### 十、String.raw()
```
String.raw`Hi\n${2+3}!`;
// 返回 "Hi\\n5!"
String.raw`Hi\u000A!`;
// 返回 "Hi\\u000A!"

// 如果原字符串的斜杠已经转义，那么 String.raw 会进行再次转义
String.raw`Hi\\n`
// 返回 "Hi\\\\n"
    
// String.raw 方法也可以作为正常的函数使用。这时，它的第一个参数，应该是一个具有 raw 属性的对象，且 raw 属性的值应该是一个数组
String.raw({ raw: 'test' }, 0, 1, 2);
// 't0e1s2t'
// 等同于
String.raw({ raw: ['t','e','s','t'] }, 0, 1, 2);
```
ES6 还为原生的 String 对象，提供了一个raw方法。String.raw 方法，往往用来充当模板字符串的处理函数，返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，对应于替换变量后的模板字符串。