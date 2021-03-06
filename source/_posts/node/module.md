---
title: node 之 module
categories:
- node
---
JavaScript 自诞生以来，曾经没有人拿它当做一门真正的编程语言，认为它不过是一种网页小脚本而已，在 Web 1.0 时代，这种脚本语言在网络中主要有两个作用广为流传，一个是表单校验，另一个是网页特效。另一方面，由于仓促地被创造出来，所以它自身的各种陷阱和缺点也被各种编程人员广为诟病。直到 Web 2.0 时代，前端工程师利用它大大提升了网页上的用户体验。
长长的后天努力过程中，JavaScript 不断被类聚和抽象，以更好地组织业务逻辑。从另一个角度而言，它也道出了 JavaScript 先天就缺乏的一项功能：模块。经历十多年的发展后，社区也为 JavaScript 制定了相应的规范，其中 CommonJS 规范的提出算是最为重要的里程碑。
<!--more--> 
### 一、模块实现
<img src="/assets/node/commonjs.png">
Node 与浏览器以及 W3C 组织、CommonJS 组织、ECMAScript 之间的关系
Node 在实现中并非完全按照规范实现，而是对模块规范进行了一定的取舍，同时也增加了少许自身需要的特性。在 Node 中引入模块，需要经历如下 3 个步骤：路径分析、文件定位、编译执行。在 Node 中，模块分为两类：一类是 Node 提供的模块，称为核心模块；另一类是用户编写的模块，称为文件模块。
- 核心模块部分在Node源代码的编译过程中，编译进了二进制执行文件。在Node进程启动时，部分核心模块就被直接加载进内存中，所以这部分核心模块引入时，文件定位和编译执行这两个步骤可以省略掉，并且在路径分析中优先判断，所以它的加载速度是最 快的。
- 文件模块则是在运行时动态加载，需要完整的路径分析、文件定位、编译执行过程，速度比核心模块慢。

### 二、优先从缓存加载
与前端浏览器会缓存静态脚本文件以提高性能一样，Node 对引入过的模块都会进行缓存，以减少二次引入时的开销。不同的地方在于，浏览器仅仅缓存文件，而 Node 缓存的是编译和执行之后的对象。
不论是核心模块还是文件模块，require() 方法对相同模块的二次加载都一律采用缓存优先的方式，这是第一优先级的。不同之处在于核心模块的缓存检查先于文件模块的缓存检查。
### 三、路径分析和文件定位
#### 3.1、模块标识符分析
前面提到过，require()方法接受一个标识符作为参数。在Node实现中，正是基于这样一个标识符进行模块查找的。模块标识符在Node中主要分为几类：核心模块，如 http、fs、path 等；. 或 .. 开始的相对路径文件模块；以 / 开始的绝对路径文件模块；非路径形式的文件模块，如自定义的 connect 模块。
- 核心模块
核心模块的优先级仅次于缓存加载，它在Node的源代码编译过程中已经编译为二进制代码，其加载过程最快。如果试图加载一个与核心模块标识符相同的自定义模块，那是不会成功的。如果自己编写了一个 http 用户模块，想要加载成功，必须选择一个不同的标识符或者换用路径的方式。
- 路径形式的文件模块
以 .、.. 和 / 开始的标识符，这里都被当做文件模块来处理。在分析路径模块时，require() 方法会将路径转为真实路径，并以真实路径作为索引，将编译执行后的结果存放到缓存中，以使二次加载时更快。由于文件模块给 Node 指明了确切的文件位置，所以在查找过程中可以节约大量时间，其加载速度慢于核心模块。
- 自定义模块
自定义模块指的是非核心模块，也不是路径形式的标识符。它是一种特殊的文件模块，可能是一个文件或者包的形式。这类模块的查找是最费时的，也是所有方式中最慢的一种。
在介绍自定义模块的查找方式之前，需要先介绍一下模块路径这个概念。
模块路径是Node在定位文件模块的具体文件时制定的查找策略，具体表现为一个路径组成的数组。关于这个路径的生成规则，我们可以手动尝试一番。
  - 创建module_path.js文件，其内容为console.log(module.paths);
  - 将其放到任意一个目录中然后执行node module_path.js
在Linux下，你可能得到的是这样一个数组输出：
```
[ '/home/jackson/research/node_modules',
'/home/jackson/node_modules',
'/home/node_modules',
'/node_modules' ]
```
而在 Windows 下，也许是这样：
```
[ 'c:\\nodejs\\node_modules', 'c:\\node_modules' ]
```
可以看出，模块路径的生成规则如下所示：
  - 当前文件目录下的node_modules目录
  - 父目录下的node_modules目录
  - 父目录的父目录下的node_modules目录
  - 沿路径向上逐级递归，直到根目录下的node_modules目录

它的生成方式与JavaScript的原型链或作用域链的查找方式十分类似。在加载的过程中，Node会逐个尝试模块路径中的路径，直到找到目标文件为止。可以看出，当前文件的路径越深，模块查找耗时会越多，这是自定义模块的加载速度是最慢的原因。
#### 3.2、文件定位
从缓存加载的优化策略使得二次引入时不需要路径分析、文件定位和编译执行的过程，大大提高了再次加载模块时的效率。
但在文件的定位过程中，还有一些细节需要注意，这主要包括文件扩展名的分析、目录和包的处理。
- 文件扩展名分析
require() 在分析标识符的过程中，会出现标识符中不包含文件扩展名的情况。CommonJS 模块规范也允许在标识符中不包含文件扩展名，这种情况下，Node 会按 .js、.json、.node 的次序补足扩展名，依次尝试。
在尝试的过程中，需要调用 fs 模块同步阻塞式地判断文件是否存在。因为 Node 是单线程的，所以这里是一个会引起性能问题的地方。小诀窍是：如果是 .node 和 .json 文件，在传递给 require() 的标识符中带上扩展名，会加快一点速度。
- 目录分析和包
在分析标识符的过程中，require() 通过分析文件扩展名之后，可能没有查找到对应文件，但却得到一个目录，这在引入自定义模块和逐个模块路径进行查找时经常会出现，此时 Node 会将目录当做一个包来处理。
在这个过程中，Node 对 CommonJS 包规范进行了一定程度的支持。首先，Node 在当前目录下查找package.json（CommonJS 包规范定义的包描述文件），通过 JSON.parse() 解析出包描述对象，从中取出 main 属性指定的文件名进行定位。如果文件名缺少扩展名，将会进入扩展名分析的步骤。
而如果 main 属性指定的文件名错误，或者压根没有 package.json 文件，Node 会将 index 当做默认文件名，然后依次查找 index.js、index.node、index.json。
如果在目录分析的过程中没有定位成功任何文件，则自定义模块进入下一个模块路径进行查找。如果模块路径数组都被遍历完毕，依然没有查找到目标文件，则会抛出查找失败的异常。

### 四、模块编译
在 Node 中，每个文件模块都是一个对象，编译和执行是引入文件模块的最后一个阶段。定位到具体的文件后，Node 会新建一个模块对象，然后根据路径载入并编译。每一个编译成功的模块都会将其文件路径作为索引缓存在 Module._cache 对象上，以提高二次引入的性能。对于不同的文件扩展名，其载入方法也有所不同:
#### 4.1、JavaScript 模块的编译
回到 CommonJS 模块规范，我们知道每个模块文件中存在着 require、exports、module 这 3 个变量，但是它们在模块文件中并没有定义，那么从何而来呢？
事实上，在编译的过程中，Node对获取的JavaScript文件内容进行了头尾包装。一个正常的JavaScript文件会被包装成如下的样子：
```
(function (exports, require, module, __filename, __dirname) {
    var math = require('math');
    exports.area = function (radius) {
        return Math.PI * radius * radius;
    };
});
```
这样每个模块文件之间都进行了作用域隔离。包装之后的代码会通过 vm 原生模块的 runInThisContext() 方法执行（类似 eval，只是具有明确上下文，不污染全局），返回一个具体的 function 对象。最后，将当前模块对象的 exports 属性、require() 方法、module（模块对象自身），以及在文件定位中得到的完整文件路径和文件目录作为参数传递给这个 function() 执行。
这就是这些变量并没有定义在每个模块文件中却存在的原因。在执行之后，模块的 exports 属性被返回给了调用方。exports 属性上的任何方法和属性都可以被外部调用到，但是模块中的其余变量或属性则不可直接被调用。此外，许多初学者都曾经纠结过为何存在 exports 的情况下，还存在 module.exports。其原因在于，exports 对象是通过形参的方式传入的，直接赋值形参会改变形参的引用，但并不能改变作用域外的值。
#### 4.2、C/C++模块的编译
Node 调用 process.dlopen() 方法进行加载和执行。在 Node 的架构下，dlopen() 方法在 Windows 和`*nix`平台下分别有不同的实现，通过 libuv 兼容层进行了封装。
实际上，.node 的模块文件并不需要编译，因为它是编写 C/C++ 模块之后编译生成的，所以这里只有加载和执行的过程。在执行的过程中，模块的 exports 对象与 .node 模块产生联系，然后返回给调用者。
#### 4.3、JSON文件的编译
.json 文件的编译是 3 种编译方式中最简单的。Node 利用 fs 模块同步读取 JSON 文件的内容之后，调用 JSON.parse() 方法得到对象，然后将它赋给模块对象的 exports，以供外部调用。
JSON 文件在用作项目的配置文件时比较有用。如果你定义了一个 JSON 文件作为配置，那就不必调用 fs 模块去异步读取和解析，直接调用 require() 引入即可。此外，你还可以享受到模块缓存的便利，并且二次引入时也没有性能影响。
### 五、核心模块及模块调用栈
#### 5.1、核心模块
前面提到，Node 的核心模块在编译成可执行文件的过程中被编译进了二进制文件。核心模块其实分为 C/C++ 编写的和 JavaScript 编写的两部分，其中 C/C++ 文件存放在 Node 项目的 src 目录下，JavaScript 文件存放在 lib 目录下。
在核心模块中，有些模块全部由 C/C++ 编写，有些模块则由 C/C++ 完成核心部分，其他部分则由 JavaScript 实现包装或向外导出，以满足性能需求。这种 C++ 模块主内完成核心，JavaScript 主外实现封装的模式是 Node 能够提高性能的常见方式。通常，脚本语言的开发速度优于静态语言，但是其性能则弱于静态语言。而 Node 的这种复合模式可以在开发速度和性能之间找到平衡点。
#### 5.2、模块调用栈
C/C++ 内建模块属于最底层的模块，它属于核心模块，主要提供 API 给 JavaScript 核心模块和第三方 JavaScript 文件模块调用。如果你不是非常了解要调用的 C/C++ 内建模块，请尽量避免通过 process.binding() 方法直接调用，这是不推荐的。
JavaScript 核心模块主要扮演的职责有两类：一类是作为 C/C++ 内建模块的封装层和桥接层，供文件模块调用；一类是纯粹的功能模块，它不需要跟底层打交道，但是又十分重要。
<img src="/assets/node/module.png">
模块之间的调用关系
文件模块通常由第三方编写，包括普通 JavaScript 模块和 C/C++ 扩展模块，主要调用方向为普通 JavaScript 模块调用扩展模块。
### 六、包与NPM
Node组织了自身的核心模块，也使得第三方文件模块可以有序地编写和使用。但是在第三方模块中，模块与模块之间仍然是散列在各地的，相互之间不能直接引用。而在模块之外，包和NPM则是将模块联系起来的一种机制，详情见 [node 之 npm](/node/npm)。
