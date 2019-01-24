---
title: node 之 Buffer
categories:
- node
---
JavaScript 对于字符串的操作十分友好，无论是多字节字符还是单字节字符，都被认为是一个字符。在体验过 JavaScript 友好的字符串操作后，有些开发者可能会形成思维定势，将 Buffer 当做字符串来理解。但字符串与 Buffer 之间有实质上的差异，即 Buffer 是二进制数据，字符串与 Buffer 之间存在编码关系。
在 Node 中，应用需要处理网络协议、操作数据库、处理图片、接收上传文件等，在网络流和文件的操作，还要处理大量二进制数据，JavaScript 自有的字符串远远不能满足这些需求，于是 Buffer 对象应运而生。
<!--more--> 
### 一、结构
#### 1.1、模块结构
Buffer 是一个像 Array 的对象，但它主要用于操作字节。
#### 1.2、Buffer 对象
Buffer 是一个典型的 JavaScript 与 C++ 结合的模块，它将性能相关部分用 C++ 实现，将非性能相关的部分用 JavaScript 实现。Buffer 所占用的内存不是通过 V8 分配的，属于堆外内存。由于 Buffer 太过常见，Node 在进程启动时就已经加载了它，并将其放在全局对象上。所以在使用 Buffer 时，无须通过 require() 即可直接使用。
Buffer 对象类似于数组，它的元素为 16 进制的两位数，即 0 到 255 的数值，如：
```
var str = '深入浅出node.js'
var buff = new Buffer(str, 'utf-8')

// <Buffer e6 b7 b1 e5 85 a5 e6 b5 85 e5 87 ba 6e 6f 64 65 2e 6a 73>
console.log(buff)
```
由上可见，不同编码的字符串占用的元素个数各不相同，上面代码中的中文字在 UTF-8 编码下占用 3 个元素，字母和半角标点符号占用 1 个元素。Buffer 受 Array 类型的影响很大，可以访问 length 属性得到长度，也可以通过下标访问元素，在构造对象时也十分相似：
```
var buff = new Buffer(100)
// 100
console.log(buff.length)
```
可以通过下标对 Buffer 进行赋值：
```
buff[10] = 100
// 100
console.log(buff[10])
```
如果给元素赋值不是 0-255 而是小数时：
```
buff[20] = -100
// 156
console.log(buff[20])
buff[21] = 300
// 44
console.log(buff[21])
buff[22] = 3.1415
// 3
console.log(buff[22])
```
给元素的赋值如果小于 0，就将该值逐次加 256，直到得到一个 0 到 255 之间的整数。如果得到的数值大于 255，将逐次减 256，直到 0-255 区间内的数值。如果是小数，舍弃小数部分，只保留整数部分。
### 二、Buffer 的转换
Buffer 对象可以与字符串之间相互转换。
#### 2.1、字符串转 Buffer
字符串转 Buffer 对象主要是通过构造函数完成的：
```
new Buffer(str, [encoding])
```
通过构造函数转换的 Buffer 对象，存储的只能是一种编码类型。encoding 参数不传递时，默认按 UTF-8 编码进行转码和存储。一个 Buffer 对象可以存储不同编码类型的字符串转码的值，调用 write() 方法可以实现该目的，代码如下：
```
buff.write(string, [offset], [length], [encoding])
```
由于可以不断写入内容到 Buffer 对象中，并且每次写入可以指定编码，所以 Buffer 对象中可以存在多种编码转化后的内容。需要小心的是，每种编码所用的字节长度不同，将 Buffer 反转回字符串时需要谨慎处理。
#### 2.2、Buffer 转字符串
实现 Buffer 向字符串的转换也十分简单，Buffer 对象的 toString() 可以将 Buffer 对象转换为字符串，代码如下：
```
buff.toString([encoding], [start], [end])
```
比较精巧的是，可以设置 encoding（默认为 UTF-8）、start、end 这 3 个参数实现整体或局部的转换。如果 Buffer 对象由多种编码写入，就需要在局部指定不同的编码，才能转换回正常的编码。
### 三、Buffer 的拼接
Buffer 在使用场景中，通常是以一段一段的方式传输，以下是常见的从输入流中读取内容的示例代码：
```
var fs = require('fs')

var rs = fs.createReadStream('test.md')
var data = ''
rs.on('data', function (chunk) {
    data += chunk
})
rs.on('end', function (chunk) {
    console.log(data)
})
```
上面这段代码常见于国外，用于流读取的示范，data 事件中获取的 chunk 对象即是 Buffer 对象。一旦输入流中有多字节编码时，问题就会暴露出来。这里潜在的问题在于如下这句代码：
```
data += chunk
```
这句代码里隐藏了 toString() 操作，它等价于如下的代码：
```
data += data.toString() + chunk.toString()
```
值得注意的是，外国人语境通常是指英文环境，在他们的场景下，这个 toString() 不会造成任何问题。但对于多字节的中文，却会形成问题。为了重现这个问题，下面模拟近似的场景，将文件可读流的每次读取 Buffer 长度限制为 11，代码如下：
```
var rs = fs.createReadStream('test.md', {highWaterMark: 11})
```
搭配该代码的测试数据为李白的《静夜思》。执行该程序，将会得到以下输入：
```
窗前明��光，疑���地上霜。举头��明月，���头思故乡
```
#### 3.1、乱码产生原因
上面的诗歌中，“月”、“是”、“望”、“低” 4个字没有被正常输出，取而代之的是�，产生这个输出结果的原因在于文件可读流在读取时会逐个读取 Buffer。这首歌原始 Buffer 应存储为：
```
<Buffer e5 ba 8a e5 89 8d e6 98 8e e6 9c 88 e5 85 89 ef bc 8c e7 96 91 e6 98 af e5 9c b0 e4 b8 8a e9 9c 9c ef bc 9b e4 b8 be e5 a4 b4 e6 9c 9b e6 98 8e e6 9c 88...>
```
由于限定了 Buffer 对象的长度为 11，因此只读流需要读取 7 次才能完成完成的读取，结果是以下几个 Buffer 对象依次输出：
```
<Buffer e5 ba 8a e5 89 8d e6 98 8e e6 9c>
<Buffer 88 e5 85 89 ef bc 8c e7 96 91 e6>
...
```
上文提到 buff.toString() 方法默认以 UTF-8 为编码，中文字在 UTF-8 下占 3 个字节。所以第一个 Buffer 对象在输出时，只能显示 3 个字符，Buffer 中剩下的 2 个字节（e6 9c）将会以乱码的形式显示。第二个 Buffer 对象的第一个字节也不能形成文字，只能显示乱码，于是形成一些文字无法正常显示的问题。
在这个示例中，我们构造了 11 这个限制，但是对于任意长度的 Buffer 而言，多字节字符都有可能存在被截断的情况，只不过 Buffer 的长度越大出现的概率越低而已，但该问题依然不可忽视。
#### 3.2、setEncoding()
在看过上述的示例后，也许我们忘记了可读流还有一个设置编码的方法 setEncoding()，示例如下：
```
readable.setEncoding(encoding)
```
改方法的作用是让 data 事件中传递不再是一个 Buffer 对象，而是编码后的字符串。为此，我们继续改进前面的诗歌的程序，添加 setEncoding() 的步骤如下：
```
var rs = fs.createReadStream('test.md', { highWaterMark: 11 })
rs.setEncoding('utf8')
```
重新执行程序，得到输出：
```
窗前明月光，疑是地上霜。举头望明月，低头思故乡
```
那 Node 是如何实现这个输出结果的呢？要知道，无论如何设置编码，触发 data 事件的次数依旧相同，这意味着设置编码并未改变按段读取的基本方式。
事实上，在用 setEncoding() 时，可读流对象在内部设置了一个 decoder，它来自于 string_decoder 模块 StringDecoder 的实例对象。下面以代码来说明：
```
var StringDecoder = require('string_decoder').StringDecoder
var decoder = new StringDecoder('utf8')

var buf1 = new Buffer([0xE5, 0xBA, 0x8A, 0xE5, 0x89, 0x8D, 0xE6, 0x98, 0x8E, 0xE6, 0x9C])
console.log(decoder.write(buf1))
// => 床前月

var buf2 = new Buffer([0x88, 0xE5, 0x85, 0x89, 0xEF, 0xBC, 0x8C, 0xE7, 0x96, 0x91, 0xE6])
console.log(decoder.write(buf2))
// => 月光，疑
```
StringDecoder 在得到编码后，知道多字节字符在 UTF-8 编码下以 3 个字节的方式存储，所以第一次 write() 时，只输出前 9 个字节转码形成的字符，“月”字的前两个字节被保留在 StringDecoder 实例内部，第二次 write() 时，会将 2 个剩余字节和后续 11 个字节组合在一起，再次用 3 的整数倍字节进行转码。于是乱码问题通过这种中间形式被解决了。
虽然 string_decoder 模块很奇妙，但是它只能处理几种编码，如 UTF-8、Base64、UCS-2/UTF-16LE。所以通过 setEncoding() 可以解决大部分的乱码问题，但并不能从根本上解决问题。
#### 3.3、正确拼接 Buffer
淘汰掉 setEncoding() 方法后，剩下的解决方案只有将多个小 Buffer 对象拼接为一个 Buffer 对象，然后通过 iconv-lite 一类的模块来转码这种方式。
```
var chunks = []
var size = 0
res.on('data', function (chunk) {
  chunks.push(chunk)
  size += chunk.length
})
res.on('end', function () {
  var buf = Buffer.concat(chunks, size)
  var str = iconv.decode(buf, 'utf8')
  console.log(str)
})
```
正确的拼接方式是用一个数组来存储接收到的所有 Buffer 片段，并记录下总长度，然后调用 Buffer.concat() 方法来生成一个合并的 Buffer 对象。Buffer.concat() 方法封装了从小 Buffer 对象向大 Buffer 对象的复制过程，实现十分细腻，值得学习：
```
Buffer.concat = function(list, length) {
  if (!Array.isArray(list)) {
    throw new Error('Usage: Buffer.concat(list, [length])');
  }

  if (list.length === 0) {
    return new Buffer(0);
  } else if (list.length === 1) {
    return list[0];
  }

  if (typeof length !== 'number') {
    length = 0;
    for (var i = 0; i < list.length; i++) {
      var buf = list[i];
      length += buf.length; }
  }

  var buffer = new Buffer(length);
  varpos = 0;
  for (var i = 0; i < list.length; i++) {
    var buf = list[i];
    buf.copy(buffer, pos);
    pos += buf.length;
  }
  return buffer;
};
```