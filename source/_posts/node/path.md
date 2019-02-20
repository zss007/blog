---
title: node 之文件路径
categories:
- node
---
node 有好几种表达文件路径的方式，但是他们之间有细微的区别，很容易搞混，现在梳理一遍。
<!--more--> 
### 一、文件路径
Node 中的文件路径大概有 `__dirname`, `__filename`, `process.cwd()`, `./` 或者 `../`，前三个都是绝对路径，为了便于比较，`./` 和 `../` 我们通过 `path.resolve('./')`来转换为绝对路径。
#### 1.1、__dirname
当前模块的目录名，与 `__filename` 的 `path.dirname()` 相同。
示例，从 /Users/songsong.zhang/study/es6test/path 运行 node example.js：
```
console.log(__dirname);
// 打印: /Users/songsong.zhang/study/es6test/path
const path = require('path');
console.log(path.dirname(__filename));
// 打印: /Users/songsong.zhang/study/es6test/path
```
#### 1.2、__filename
当前模块的文件名，这是当前模块文件的已解析的绝对路径。
```
console.log(__filename);
// 打印: /Users/songsong.zhang/study/es6test/path/example.js
```
#### 1.3、process.cwd()
process.cwd() 方法返回 Node.js 进程的当前工作目录。
```
console.log(`当前工作目录是: ${process.cwd()}`);
// 打印: 当前工作目录是: /Users/songsong.zhang/study/es6test/path
```
#### 1.4、./ 与 ../
我们通过 path.resolve() 方法将路径或路径片段的序列解析为绝对路径。
```
const path = require('path');
console.log(path.resolve('./'));
// 打印: /Users/songsong.zhang/study/es6test/path
console.log(path.resolve('../'));
// 打印: /Users/songsong.zhang/study/es6test
```
### 二、综合实例
假如我们有这样的文件结构：
```
path/
    -lib/
        -common.js
    -model
        -task.js
```
在 task.js 里编写如下的代码：
```
var path = require('path');

console.log(__dirname);
console.log(__filename);
console.log(process.cwd());
console.log(path.resolve('./'));
```
在 `model` 目录下运行 `node task.js` 得到的输出是：
```
/Users/songsong.zhang/study/es6test/path/model
/Users/songsong.zhang/study/es6test/path/model/task.js
/Users/songsong.zhang/study/es6test/path/model
/Users/songsong.zhang/study/es6test/path/model
```
然后在 `path` 目录下运行 `node model/task.js`，得到的输出是：
```
/Users/songsong.zhang/study/es6test/path/model
/Users/songsong.zhang/study/es6test/path/model/task.js
/Users/songsong.zhang/study/es6test/path
/Users/songsong.zhang/study/es6test/path
```
可以得出一些**肤浅的结论**了：
* __dirname: 总是返回被执行的 js 所在文件夹的绝对路径
* __filename: 总是返回被执行的 js 的绝对路径
* process.cwd(): 总是返回运行 node 命令时所在的文件夹的绝对路径
* ./: 跟 process.cwd() 貌似一样

但是在 `require('../lib/common')` 里一直都是各种相对路径写，也没见报什么错啊，我们再来个栗子吧，还是上面的结构，`model/task.js` 里的代码改成：
```
var fs = require('fs');
var common = require('../lib/common');

fs.readFile('../lib/common.js', function (err, data) {
    if (err) return console.log(err);
    console.log(data);
});
```
在 model 目录下运行 `node task.js`，一切 Ok，没有报错。然后在 path 目录下运行 `node model/task.js`，然后很果断滴报错了:
```
{ Error: ENOENT: no such file or directory, open '../lib/common.js'
  errno: -2,
  code: 'ENOENT',
  syscall: 'open',
  path: '../lib/common.js' }
```
那么这下问题真的都是来了，按照上面的理论，在 path 下运行时，`../lib/common.js` 会被转成 `/Users/songsong.zhang/study/es6test/lib/common.js`，这个路径显然是不存在的，但是从运行结果可以看出 `require('../lib/common')` 是 OK 的，只是 readFile 时报错了。

那么关于 `./` 正确的结论是：
在 `require()` 中使用是跟 `__dirname` 的效果相同，不会因为启动脚本的目录不一样而改变，在其他情况下跟 `process.cwd()` 效果相同，是相对于启动脚本所在目录的路径。
### 三、总结
只有在 `require()` 时才使用相对路径 `(./, ../)` 的写法，其他地方一律使用绝对路径，如下：
```
// 当前目录下
path.dirname(__filename) + '/test.js';
// 相邻目录下
path.resolve(__dirname, '../lib/common.js');
```