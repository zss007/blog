---
title: node 之进程管理
categories:
- node
---
JavaScript 运行在单个进程的单个线程上，它带来的好处是：程序状态是单一的，在没有多线程的情况下没有锁、线程同步问题，操作系统在调度时也因为较少上下文的切换，可以很好地提高 CPU 的使用率，但是单进程单线程并非完美的结构。如何充分利用多核CPU服务器、如何保证进程的健壮性和稳定性，前者只是利用率不足的问题，后者对于实际产品化带来一定的顾虑，本文关于进程的介绍和讨论将会解决掉这两个问题。
Node 自身还有一定的 I/O 线程存在，这些 I/O 线程由底层 libuv 处理，这部分线程对于 JavaScript 开发者而言是透明的，本文将围绕 JavaScript 部分展开，屏蔽底层细节的讨论。
<!--more--> 
### 一、服务模型变迁
Web 服务器的架构已经历了几次变迁，服务器处理客户端请求的并发量，就是每个里程碑的见证。
#### 1.1、同步
最早的服务器，其执行模型是同步的，它的服务模式是一次只为一个请求服务，它的处理能力相当低下，假设每次响应服务耗用的时间稳定为 N 秒，这类服务的 QPS 为 1/N。
#### 1.2、复制进程
为了解决同步架构的并发问题，一个简单的改进是通过进程的复制同时服务更多的请求和用户。并把预复制（prefork）被引入服务模型中，即预先复制一定数量的进程，同时将进程复用，避免进程创建、销毁带来的开销。但是这个模型并不具备伸缩性，一旦并发请求过高，内存使用随着进程数的增长将会被耗尽，假设进程数上限为 M，那这类服务的 QPS 为 M/N。
#### 1.3、多线程
为了解决进程复制中的浪费问题，多线程被引入服务模型，让一个线程服务一个请求。线程相对进程的开销要小许多，并且线程之间可以共享数据，内存浪费的问题可以得到解决，并且利用线程池可以减少创建和销毁线程的开销。但是当线程数量过多时，时间将会被耗用在上下文切换中。所以在大并发量时，多线程结构还是无法做到强大的伸缩性，假设线程所占用的资源为进程的 1/L，受资源上限的影响，它的 QPS 则为 M * L/N。
#### 1.4、事件驱动
Apache 就是采用多线程/多进程模型实现的，当并发增长到上万时，内存耗用的问题将会暴露出来。为了解决高并发问题，基于事件驱动的服务模型出现了，像 Node 与 Nginx 均是基于事件驱动的方式实现的，采用单线程避免了不必要的内存开销和上下文切换开销。
基于事件的服务模型存在的问题即是开始时提及的两个问题：CPU 的利用率和进程的健壮性。由于所有处理都在单线程上进行，影响事件驱动服务模型性能的点在于 CPU 的计算能力，它的上限决定这类服务模型的性能上限，可伸缩性远比前两者高。
### 二、多进程架构
理想状态下每个进程各自利用一个 CPU，以此实现多核 CPU 的利用。Node 提供了 child_process 模块，并且也提供了 child_process.fork() 函数供我们实现进程的复制。通过 fork() 复制的进程都是一个独立的进程，这个进程中有着独立而全新的 V8 实例。它需要至少 30 毫秒的启动时间和至少 10MB 的内存。尽管 Node 提供了 fork() 供我们复制进程使每个 CPU 内核都使用上，但是依然要切记 fork() 进程是昂贵的。好在 Node 通过事件驱动的方式在单线程上解决了大并发的问题，这里启动多个进程只是为了充分将 CPU 资源利用起来。
#### 2.1、创建子进程
child_process 模块给予 Node 可以随意创建子进程（child_process）的能力。它提供了4个方法用于创建子进程。
- spawn()：启动一个子进程来执行命令；
- exec()：启动一个子进程来执行命令，有回调函数获知子进程的状况；
- execFile()：启动一个子进程来执行可执行文件；
- fork()：与 spawn() 类似，不同点在于它创建 Node 的子进程只需指定要执行的 JavaScript 文件模块即可。

以一个寻常命令为例，node worker.js 分别用上述 4 种方法实现，如下所示：
```
var cp = require('child_process');
cp.spawn('node', ['worker.js']);
cp.exec('node worker.js', function (err, stdout, stderr) {
// some code
});
cp.execFile('worker.js', function (err, stdout, stderr) {
    // some code
});
cp.fork('./worker.js');
```
以上4个方法在创建子进程之后均会返回子进程对象。它们的差别可以通过表9-1查看：

|    类型    |    回调/异常   |  进程类型 |   执行类型        | 可设置超时 |
|:----------:|:------------:|:--------:|:---------------:|:--------:|
| spawn()    |     N        |   任意    |   命令           |   N    |
| exec()     |     Y        |   任意    |   命令           |   Y    |
| execFile() |     Y        |   任意    |   可执行文件      |   Y    |
| fork()     |     N        |   Node   |  JavaScript文件  |   N    |

这里的可执行文件是指可以直接执行的文件，如果是 JavaScript 文件通过 execFile() 运行，它的首行内容必须添加如下代码：
```
#!/usr/bin/env node
```
尽管4种创建子进程的方式有些差别，但事实上后面 3 种方法都是 spawn() 的延伸应用。
#### 2.2、进程间通信
在前端浏览器中，JavaScript 主线程与 UI 渲染共用同一个线程。执行 JavaScript 的时候 UI 渲染是停滞的，渲染 UI 时，JavaScript 是停滞的，两者互相阻塞。长时间执行 JavaScript 将会造成 UI 停顿不响应。为了解决这个问题，HTML5 提出了 WebWorker API。WebWorker 允许创建工作线程并在后台运行，使得一些阻塞较为严重的计算不影响主线程上的 UI 渲染。
而 Node 在 Master-Worker 模式中，要实现主进程管理和调度工作进程的功能，需要主进程和工作进程之间的通信。对于 child_process 模块，创建好了子进程，然后与父子进程间通信是十分容易的。
```
// parent.js
var cp = require('child_process');
var n = cp.fork(__dirname + '/sub.js');
n.on('message', function (m) {
    console.log('PARENT got message:', m);
});
n.send({ hello: 'world' });
// sub.js
process.on('message', function (m) {
    console.log('CHILD got message:', m);
});
process.send({foo: 'bar'});
```
通过 fork() 或者其他 API，创建子进程之后，为了实现父子进程之间的通信，父进程与子进程之间将会创建 IPC 通道。通过 IPC 通道，父子进程之间才能通过 message 和 send() 传递消息。注意，process.send 用来给父进程发送消息，只有子进程才存在。
#### 2.3、进程间通信原理
IPC 的全称是 Inter-Process Communication，即进程间通信。进程间通信的目的是为了让不同的进程能够互相访问资源并进行协调工作。实现进程间通信的技术有很多，如命名管道、匿名管道、socket、信号量、共享内存、消息队列、Domain Socket 等。Node 中实现 IPC 通道的是管道（pipe）技术。但此管道非彼管道，在 Node 中管道是个抽象层面的称呼，具体细节实现由 libuv 提供，在 Windows 下由命名管道（named pipe）实现，*nix 系统则采用 Unix Domain Socket 实现。表现在应用层上的进程间通信只有简单的 message 事件和 send() 方法，接口十分简洁和消息化，如下为 IPC 创建和实现的示意图。
<img src="/assets/node/process.png">
父进程在实际创建子进程之前，会创建 IPC 通道并监听它，然后才真正创建出子进程，并通过环境变量（NODE_CHANNEL_FD）告诉子进程这个 IPC 通道的文件描述符。子进程在启动的过程中，根据文件描述符去连接这个已存在的 IPC 通道，从而完成父子进程之间的连接，如下为创建 IPC 管道的步骤示意图。
<img src="/assets/node/process2.png">
建立连接之后的父子进程就可以自由地通信了。由于 IPC 通道是用命名管道或 Domain Socket 创建的，它们与网络 socket 的行为比较类似，属于双向通信。不同的是它们在系统内核中就完成了进程间的通信，而不用经过实际的网络层，非常高效。在 Node 中，IPC 通道被抽象为 Stream 对象，在调用 send() 时发送数据（类似于 write()），接收到的消息会通过 message 事件（类似于 data）触发给应用层。
注意，只有启动的子进程是 Node 进程时，子进程才会根据环境变量去连接 IPC 通道，对于其他类型的子进程则无法实现进程间通信，除非其他进程也按约定去连接这个已经创建好的 IPC 通道。
#### 2.4、句柄传递
建立好进程之间的 IPC 后，将启动的服务器分别监听各自的端口么，如果让服务都监听到相同的端口，将会有什么样的结果？示例如下所示：
```
// master.js
var http = require('http');
http.createServer(function (req, res) {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello World\n');
}).listen(8888, '127.0.0.1');
```
再次启动 master.js 文件，如下所示：
```
events.js:183
      throw er; // Unhandled 'error' event
      ^

Error: listen EADDRINUSE 127.0.0.1:8888
```
这时只有一个工作进程能够监听到该端口上，其余的进程在监听的过程中都抛出了 EADDRINUSE 异常，这是端口被占用的情况，新的进程不能继续监听该端口了。这个问题破坏了我们将多个进程监听同一个端口的想法。要解决这个问题，通常的做法是让每个进程监听不同的端口，其中主进程监听主端口（如 80），主进程对外接收所有的网络请求，再将这些请求分别代理到不同的端口的进程上，如下图所示。
<img src="/assets/node/process3.png">
通过代理，可以避免端口不能重复监听的问题，甚至可以在代理进程上做适当的负载均衡，使得每个子进程可以较为均衡地执行任务。由于进程每接收到一个连接，将会用掉一个文件描述符，因此代理方案中客户端连接到代理进程，代理进程连接到工作进程的过程需要用掉两个文件描述符。操作系统的文件描述符是有限的，代理方案浪费掉一倍数量的文件描述符的做法影响了系统的扩展能力。
为了解决上述这样的问题，Node 在版本 v0.5.9 引入了进程间发送句柄的功能。send() 方法除了能通过 IPC 发送数据外，还能发送句柄，第二个可选参数就是句柄，如下所示：
```
child.send(message, [sendHandle])
```
那什么是句柄？句柄是一种可以用来标识资源的引用，它的内部包含了指向对象的文件描述符。比如句柄可以用来标识一个服务器端 socket 对象、一个客户端 socket 对象、一个 UDP 套接字、一个管道等。
发送句柄意味着什么？在前一个问题中，我们可以去掉代理这种方案，使主进程接收到 socket 请求后，将这个 socket 直接发送给工作进程，而不是重新与工作进程之间建立新的 socket 连接来转发数据。文件描述符浪费的问题可以通过这样的方式轻松解决。来看看我们的示例代码。
主进程代码如下所示：
```
var cp = require('child_process');
var child1 = cp.fork('child.js');
var child2 = cp.fork('child.js');

// Open up the server object and send the handle
var server = require('net').createServer();
server.on('connection', function (socket) {
    socket.end('handled by parent\n');
});
server.listen(8001, function () {
    child1.send('server', server);
    child2.send('server', server);
});
```
子进程代码如下所示：
```
process.on('message', function (m, server) {
    if (m === 'server') {
        server.on('connection', function (socket) {
            socket.end('handled by child, pid is ' + process.pid + '\n');
        });
    }
});
```
这个示例中直接将一个 TCP 服务器发送给了子进程。这是看起来不可思议的事情，我们先来测试一番，看看效果如何，如下所示：
```
// 先启动服务器
$ node parent.js
```
然后新开一个命令行窗口，用上 curl 工具，如下所示：
```
$ curl "http://127.0.0.1:8001"
handled by child, pid is 27418
$ curl "http://127.0.0.1:8001"
handled by child, pid is 27419
$ curl "http://127.0.0.1:8001"
handled by child, pid is 27419
$ curl "http://127.0.0.1:8001"
handled by parent
```
命令行中的响应结果也是很不可思议的，这里子进程和父进程都有可能处理我们客户端发起的请求。这是在 TCP 层面上完成的事情，我们尝试将其转化到 HTTP 层面来试试。对于主进程而言，我们甚至想要它更轻量一点，那么是否将服务器句柄发送给子进程之后，就可以关掉服务器的监听，让子进程来处理请求呢？
对主进程进行改动，如下所示：
```
// parent.js
var cp = require('child_process');
var child1 = cp.fork('child.js');
var child2 = cp.fork('child.js');

// Open up the server object and send the handle
var server = require('net').createServer();

server.listen(8001, function () {
    child1.send('server', server);
    child2.send('server', server);

    server.close();
});
```
然后对子进程进行改动，如下所示：
```
var http = require('http');

var server = http.createServer(function (req, res) {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('handled by child, pid is ' + process.pid + '\n');
});

process.on('message', function (m, tcp) {
    if (m === 'server') {
        tcp.on('connection', function (socket) {
            server.emit('connection', socket);
        });
    }
});
```
重新启动 parent.js 后，再次测试，如下所示：
```
$ curl "http://127.0.0.1:8001"
handled by child, pid is 27882
$ curl "http://127.0.0.1:8001"
handled by child, pid is 27883
```
这样一来，所有的请求都是由子进程处理了。整个过程中，服务的过程发生了一次改变，如下图所示。
<img src="/assets/node/process4.png">
主进程发送完句柄并关闭监听之后，成为了如下图所示的结构。
<img src="/assets/node/process5.png">
我们神奇地发现，多个子进程可以同时监听相同端口，再没有 EADDRINUSE 异常发生了。
#### 2.5、句柄发送与还原
上文介绍的虽然是句柄发送，但是仔细看看，句柄发送跟我们直接将服务器对象发送给子进程有没有差别？它是否真的将服务器对象发送给了子进程？为什么它可以发送到多个子进程中？发送给子进程为什么父进程中还存在这个对象？
目前子进程对象 send() 方法可以发送的句柄类型包括如下几种。
- net.Socket。TCP 套接字。
- net.Server。TCP 服务器，任意建立在 TCP 服务上的应用层服务都可以享受到它带来的好处。
- net.Native。C++ 层面的 TCP 套接字或 IPC 管道。
- dgram.Socket。UDP 套接字。
- dgram.Native。C++ 层面的 UDP 套接字。

send() 方法在将消息发送到 IPC 管道前，将消息组装成两个对象，一个参数是 handle，另一个是 message。message 参数如下所示：
```
{
    cmd: 'NODE_HANDLE', type: 'net.Server', msg: message
}
```
发送到 IPC 管道中的实际上是我们要发送的句柄文件描述符，文件描述符实际上是一个整数值。这个 message 对象在写入到 IPC 管道时也会通过 JSON.stringify() 进行序列化。所以最终发送到 IPC 通道中的信息都是字符串，send() 方法能发送消息和句柄并不意味着它能发送任意对象。
连接了 IPC 通道的子进程可以读取到父进程发来的消息，将字符串通过 JSON.parse() 解析还原为对象后，才触发 message 事件将消息体传递给应用层使用。在这个过程中，消息对象还要被进行过滤处理，message.cmd 的值如果以 NODE_ 为前缀，它将响应一个内部事件 internalMessage。如果 message.cmd 值为 NODE_HANDLE，它将取出 message.type 值和得到的文件描述符一起还原出一个对应的对象。
以发送的 TCP 服务器句柄为例，子进程收到消息后的还原过程如下所示：
```
function(message, handle, emit) {
    var self = this;
    var server = new net.Server();
    server.listen(handle, function () {
        emit(server);
    });
}
```
上面的代码中，子进程根据 message.type 创建对应 TCP 服务器对象，然后监听到文件描述符上。由于底层细节不被应用层感知，所以在子进程中，开发者会有一种服务器就是从父进程中直接传递过来的错觉。值得注意的是，Node 进程之间只有消息传递，不会真正地传递对象，这种错觉是抽象封装的结果。
目前 Node 只支持上述提到的几种句柄，并非任意类型的句柄都能在进程之间传递，除非它有完整的发送和还原的过程。
#### 2.4、端口共同监听
在了解了句柄传递背后的原理后，我们继续探究为何通过发送句柄后，多个进程可以监听到相同的端口而不引起 EADDRINUSE 异常。其答案也很简单，我们独立启动的进程中，TCP 服务器端 socket 套接字的文件描述符并不相同，导致监听到相同的端口时会抛出异常。
Node 底层对每个端口监听都设置了 SO_REUSEADDR 选项，这个选项的涵义是不同进程可以就相同的网卡和端口进行监听，这个服务器端套接字可以被不同的进程复用，如下所示：
```
setsockopt(tcp->io_watcher.fd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on))
```
由于独立启动的进程互相之间并不知道文件描述符，所以监听相同端口时就会失败。但对于 send() 发送的句柄还原出来的服务而言，它们的文件描述符是相同的，所以监听相同端口不会引起异常。
多个应用监听相同端口时，文件描述符同一时间只能被某个进程所用。换言之就是网络请求向服务器端发送时，只有一个幸运的进程能够抢到连接，也就是说只有它能为这个请求进行服务。这些进程服务是抢占式的。
### 三、集群稳定之路
搭建好了集群，充分利用了多核 CPU 资源，似乎就可以迎接客户端大量的请求了。但是我们还有一些细节需要考虑，性能问题、多个工作进程的存活状态管理、工作进程的平滑重启、配置或者静态数据的动态重新载入、其他细节。
虽然我们创建了很多工作进程，但每个工作进程依然是在单线程上执行的，它的稳定性还不能得到完全的保障。我们需要建立起一个健全的机制来保障 Node 应用的健壮性。
#### 3.1、进程事件
再次回归到子进程对象上，除了引人关注的 send() 方法和 message 事件外，子进程还有些什么呢？首先除了 message 事件外，Node 还有如下这些事件。
- error：当子进程无法被复制创建、无法被杀死、无法发送消息时会触发该事件。
- exit：子进程退出时触发该事件，子进程如果是正常退出，这个事件的第一个参数为退出码，否则为 null。如果进程是通过 kill() 方法被杀死的，会得到第二个参数，它表示杀死进程时的信号。
- close：在子进程的标准输入输出流中止时触发该事件，参数与 exit 相同。
- disconnect：在父进程或子进程中调用 disconnect() 方法时触发该事件，在调用该方法时将关闭监听 IPC 通道。

上述这些事件是父进程能监听到的与子进程相关的事件。除了 send() 外，还能通过 kill() 方法给子进程发送消息。kill() 方法并不能真正地将通过 IPC 相连的子进程杀死，它只是给子进程发送了一个系统信号。默认情况下，父进程将通过 kill() 方法给子进程发送一个 SIGTERM 信号。它与进程默认的 kill() 方法类似，如下所示：
```
// 子进程
child.kill([signal]);
// 当前进程
process.kill(pid, [signal]);
```
它们一个发给子进程，一个发给目标进程。在 POSIX 标准中，有一套完备的信号系统，在命令行中执行 kill -l 可以看到详细的信号列表，如下所示：
```
$ kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL
 5) SIGTRAP	 6) SIGABRT	 7) SIGEMT	 8) SIGFPE
 9) SIGKILL	10) SIGBUS	11) SIGSEGV	12) SIGSYS
13) SIGPIPE	14) SIGALRM	15) SIGTERM	16) SIGURG
17) SIGSTOP	18) SIGTSTP	19) SIGCONT	20) SIGCHLD
21) SIGTTIN	22) SIGTTOU	23) SIGIO	24) SIGXCPU
25) SIGXFSZ	26) SIGVTALRM	27) SIGPROF	28) SIGWINCH
29) SIGINFO	30) SIGUSR1	31) SIGUSR2
```
Node 提供了这些信号对应的信号事件，每个进程都可以监听这些信号事件。这些信号事件是用来通知进程的，每个信号事件有不同的含义，进程在收到响应信号时，应当做出约定的行为，如SIGTERM是软件终止信号，进程收到该信号时应当退出。示例代码如下所示：
```
process.on('SIGTERM', function () {
    console.log('Got a SIGTERM, exiting...');
    process.exit(1);
});
console.log('server running with PID:', process.pid);
process.kill(process.pid, 'SIGTERM'); 
```
#### 3.2、自动重启
有了父子进程之间的相关事件之后，就可以在这些关系之间创建出需要的机制了。至少能够通过监听子进程的 exit 事件来获知其退出的信息，在主进程上要加入一些子进程管理的机制，比如重新启动一个工作进程来继续服务。
实现代码如下所示：
```
// master.js
var fork = require('child_process').fork;
var cpus = require('os').cpus();
var server = require('net').createServer();
server.listen(1337);
var workers = {};
var createWorker = function () {
    var worker = fork(__dirname + '/worker.js');
    // 退出时重新启动新的进程
    worker.on('exit', function () {
        console.log('Worker ' + worker.pid + ' exited.');
        delete workers[worker.pid];
        createWorker();
    });
    // 句柄转发
    worker.send('server', server);
    workers[worker.pid] = worker;
    console.log('Create worker. pid: ' + worker.pid);
};
for (var i = 0; i < cpus.length; i++) {
    createWorker();
}
// 进程自己退出时，让所有工作进程退出 
process.on('exit', function () {
    for (var pid in workers) {
        workers[pid].kill();
    }
});
```
测试一下上面的代码，如下所示：
```
$ node master.js
Create worker. pid: 28939
Create worker. pid: 28940
Create worker. pid: 28941
Create worker. pid: 28942
Create worker. pid: 28943
Create worker. pid: 28944
Create worker. pid: 28945
Create worker. pid: 28946
```
通过 kill 命令杀死某个进程试试，如下所示：
```
$ kill 28939 
```
结果是 28939 进程退出后，自动启动了一个新的工作进程 28959，总体进程数量并没有发生改变，如下所示：
```
Worker 28939 exited.
Create worker. pid: 28959
```
在这个场景中我们主动杀死了一个进程，在实际业务中，可能有隐藏的 bug 导致工作进程退出，那么我们需要仔细地处理这种异常，如下所示：
```
// worker.js 
var http = require('http');
var server = http.createServer(function (req, res) {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('handled by child, pid is ' + process.pid + '\n');
});
var worker;
process.on('message', function (m, tcp) {
    if (m === 'server') {
        worker = tcp;
        worker.on('connection', function (socket) {
            server.emit('connection', socket);
        });
    }
});
process.on('uncaughtException', function () {
    // 停止接收新的连接  
    worker.close(function () {
        // 所有已有连接断开后，退出进程  
        process.exit(1);
    });
});
```
上述代码的处理流程是，一旦有未捕获的异常出现，工作进程就会立即停止接收新的连接；当所有连接断开后，退出进程。主进程在侦听到工作进程的 exit 后，将会立即启动新的进程服务，以此保证整个集群中总是有进程在为用户服务的。
#### 3.3、自杀信号
当然上述代码存在的问题是要等到已有的所有连接断开后进程才退出，在极端的情况下，所有工作进程都停止接收新的连接，全处在等待退出的状态。但在等到进程完全退出才重启的过程中，所有新来的请求可能存在没有工作进程为新用户服务的情景，这会丢掉大部分请求。
为此需要改进这个过程，不能等到工作进程退出后才重启新的工作进程。当然也不能暴力退出进程，因为这样会导致已连接的用户直接断开。于是在退出的流程中增加一个自杀（suicide）信号。工作进程在得知要退出时，向主进程发送一个自杀信号，然后才停止接收新的连接，当所有连接断开后才退出。主进程在接收到自杀信号后，立即创建新的工作进程服务。代码改动如下所示：
```
// worker.js 
process.on('uncaughtException', function (err) {
    process.send({ act: 'suicide' });
    // 停止接收新的连接   
    worker.close(function () {
        // 所有已有连接断开后，退出进程 
        process.exit(1);
    });
}); 
```
主进程将重启工作进程的任务，从 exit 事件的处理函数中转移到 message 事件的处理函数中，如下所示：
```
var createWorker = function () {
    var worker = fork(__dirname + '/worker.js');
    // 启动新的进程   
    worker.on('message', function (message) {
        if (message.act === 'suicide') {
            createWorker();
        }
    });
    worker.on('exit', function () {
        console.log('Worker ' + worker.pid + ' exited.');
        delete workers[worker.pid];
    });
    worker.send('server', server);
    workers[worker.pid] = worker;
    console.log('Create worker. pid: ' + worker.pid);
};
```
为了模拟未捕获的异常，我们将工作进程的处理代码改为抛出异常，一旦有用户请求，就会有一个可怜的工作进程退出，如下所示：
```
var server = http.createServer(function (req, res) {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('handled by child, pid is ' + process.pid + '\n');
    throw new Error('throw exception');
}); 
```
然后启动所有进程，如下所示：
```
$ node master.js
Create worker. pid: 29428
Create worker. pid: 29429
Create worker. pid: 29430
Create worker. pid: 29431
Create worker. pid: 29432
Create worker. pid: 29433
Create worker. pid: 29434
Create worker. pid: 29435
```
用curl工具测试效果，如下所示：
```
$ curl http://127.0.0.1:8001
handled by child, pid is 29433 
```
再回头看重启信息，如下所示：
```
Create worker. pid: 29472
Worker 29462 exited.
```
与前一种方案相比，创建新工作进程在前，退出异常进程在后。在这个可怜的异常进程退出之前，总是有新的工作进程来替上它的岗位。至此我们完成了进程的平滑重启，一旦有异常出现，主进程会创建新的工作进程来为用户服务，旧的进程一旦处理完已有连接就自动断开。整个过程使得我们的应用的稳定性和健壮性大大提高。
这里存在问题的是有可能我们的连接是长连接，不是 HTTP 服务的这种短连接，等待长连接断开可能需要较久的时间。为此为已有连接的断开设置一个超时时间是必要的，在限定时间里强制退出的设置如下所示：
```
process.on('uncaughtException', function (err) {
    process.send({ act: 'suicide' });
    // 停止接收新的连接
    worker.close(function () {
        // 所有已有连接断开后，退出进程   
        process.exit(1);
    });
    // 5秒后退出进程 
    setTimeout(function () {
        process.exit(1);
    }, 5000);
}); 
```
进程中如果出现未能捕获的异常，就意味着有那么一段代码在健壮性上是不合格的。为此退出进程前，通过日志记录下问题所在是必须要做的事情，它可以帮我们很好地定位和追踪代码异常出现的位置，如下所示：
```
process.on('uncaughtException', function (err) {
    // 记录日志  
    logger.error(err);
    // 发送自杀信号   
    process.send({ act: 'suicide' });
    // 停止接收新的连接  
    worker.close(function () {
        // 所有已有连接断开后，退出进程    
        process.exit(1);
    });
    // 5秒后退出进程 
    setTimeout(function () {
        process.exit(1);
    }, 5000);
}); 
```
#### 3.4、限量重启
通过自杀信号告知主进程可以使得新连接总是有进程服务，但是依然还是有极端的情况。工作进程不能无限制地被重启，如果启动的过程中就发生了错误，或者启动后接到连接就收到错误，会导致工作进程被频繁重启，这种频繁重启不属于我们捕捉未知异常的情况，因为这种短时间内频繁重启已经不符合预期的设置，极有可能是程序编写的错误。
为了消除这种无意义的重启，在满足一定规则的限制下，不应当反复重启。比如在单位时间内规定只能重启多少次，超过限制就触发 giveup 事件，告知放弃重启工作进程这个重要事件。
为了完成限量重启的统计，我们引入一个队列来做标记，在每次重启工作进程之间进行打点并判断重启是否太过频繁，如下所示：
```
// 重启次数 
var limit = 10;
// 时间单位 
var during = 60000;
var restart = [];
var isTooFrequently = function () {
    // 记录重启时间   
    var time = Date.now();
    var length = restart.push(time);
    if (length > limit) {
        // 取出最后 10 个记录     
        restart = restart.slice(limit * -1);
    }
    // 最后一次重启到前 10 次重启之间的时间间隔  
    return restart.length >= limit && restart[restart.length - 1] - restart[0] < during;
};
var workers = {};
var createWorker = function () {
    // 检查是否太过频繁  
    if (isTooFrequently()) {
        // 触发 giveup 事件后，不再重启  
        process.emit('giveup', length, during);
        return;
    }
    var worker = fork(__dirname + '/worker.js');
    worker.on('exit', function () {
        console.log('Worker ' + worker.pid + ' exited.');
        delete workers[worker.pid];
    });
    // 重新启动新的进程   
    worker.on('message', function (message) {
        if (message.act === 'suicide') {
            createWorker();
        }
    });
    // 句柄转发   
    worker.send('server', server);
    workers[worker.pid] = worker;
    console.log('Create worker. pid: ' + worker.pid);
}; 
```
giveup 事件是比 uncaughtException 更严重的异常事件。uncaughtException 只代表集群中某个工作进程退出，在整体性保证下，不会出现用户得不到服务的情况，但是这个 giveup 事件则表示集群中没有任何进程服务了，十分危险。为了健壮性考虑，我们应在 giveup 事件中添加重要日志，并让监控系统监视到这个严重错误，进而报警等。
#### 3.5、负载均衡
在多进程之间监听相同的端口，使得用户请求能够分散到多个进程上进行处理，这带来的好处是可以将 CPU 资源都调用起来。这犹如饭店将客人的点单分发给多个厨师进行餐点制作，这种保证多个处理单元工作量公平的策略叫负载均衡。
Node 默认提供的机制是采用操作系统的抢占式策略。所谓的抢占式就是在一堆工作进程中，闲着的进程对到来的请求进行争抢，谁抢到谁服务。
一般而言，这种抢占式策略对大家是公平的，各个进程可以根据自己的繁忙度来进行抢占。但是对于 Node 而言，需要分清的是它的繁忙是由 CPU、I/O 两个部分构成的，影响抢占的是 CPU 的繁忙度。对不同的业务，可能存在 I/O 繁忙，而 CPU 较为空闲的情况，这可能造成某个进程能够抢到较多请求，形成负载不均衡的情况。
为此 Node 在 v0.11 中提供了一种新的策略使得负载均衡更合理，这种新的策略叫 Round-Robin，又叫轮叫调度。轮叫调度的工作方式是由主进程接受连接，将其依次分发给工作进程。分发的策略是在 N 个工作进程中，每次选择第 i = (i + 1) mod n 个进程来发送连接。在 cluster 模块中启用它的方式如下：
```
// 启用 Round-Robin  
cluster.schedulingPolicy = cluster.SCHED_RR
// 不启用 Round-Robin
cluster.schedulingPolicy = cluster.SCHED_NONE
```
或者在环境变量中设置 NODE_CLUSTER_SCHED_POLICY 的值，如下所示：
```
export NODE_CLUSTER_SCHED_POLICY = rr 
export NODE_CLUSTER_SCHED_POLICY = none 
```
Round-Robin 非常简单，可以避免 CPU 和 I/O 繁忙差异导致的负载不均衡。Round-Robin 策略也可以通过代理服务器来实现，但是它会导致服务器上消耗的文件描述符是平常方式的两倍。
#### 3.6、状态共享
在 Node 进程中不宜存放太多数据，因为它会加重垃圾回收的负担，进而影响性能。同时，Node 也不允许在多个进程之间共享数据。但在实际的业务中，往往需要共享一些数据，譬如配置数据，这在多个进程中应当是一致的。为此，在不允许共享数据的情况下，我们需要一种方案和机制来实现数据在多个进程之间的共享。
- 第三方数据存储
解决数据共享最直接、简单的方式就是通过第三方来进行数据存储，比如将数据存放到数据库、磁盘文件、缓存服务（如Redis）中，所有工作进程启动时将其读取进内存中。但这种方式存在的问题是如果数据发生改变，还需要一种机制通知到各个子进程，使得它们的内部状态也得到更新。
实现状态同步的机制有两种，一种是各个子进程去向第三方进行定时轮询。
定时轮询带来的问题是轮询时间不能过密，如果子进程过多，会形成并发处理，如果数据没有发生改变，这些轮询会没有意义，白白增加查询状态的开销。如果轮询时间过长，数据发生改变时，不能及时更新到子进程中，会有一定的延迟。
- 主动通知
一种改进的方式是当数据发生更新时，主动通知子进程。当然，即使是主动通知，也需要一种机制来及时获取数据的改变。这个过程仍然不能脱离轮询，但我们可以减少轮询的进程数量，我们将这种用来发送通知和查询状态是否更改的进程叫做通知进程。为了不混合业务逻辑，可以将这个进程设计为只进行轮询和通知，不处理任何业务逻辑。
这种推送机制如果按进程间信号传递，在跨多台服务器时会无效，是故可以考虑采用 TCP 或 UDP 的方案。进程在启动时从通知服务处除了读取第一次数据外，还将进程信息注册到通知服务处。一旦通过轮询发现有数据更新后，根据注册信息，将更新后的数据发送给工作进程。由于不涉及太多进程去向同一地方进行状态查询，状态响应处的压力不至于太过巨大，单一的通知服务轮询带来的压力并不大，所以可以将轮询时间调整得较短，一旦发现更新，就能实时地推送到各个子进程中。

### 四、Cluster 模块
Node v0.8 时直接引入了 cluster 模块，用以解决多核 CPU 的利用率问题，同时也提供了较完善的 API，用以处理进程的健壮性问题，cluster 实现创建 Node 进程集群也是很轻松的事情，如下所示：
```
// cluster.js
var cluster = require('cluster');
cluster.setupMaster({
    exec: "./worker.js"
});

var cpus = require('os').cpus(), works = [];
for (var i = 0; i < cpus.length; i++) {
    works[i] = cluster.fork();
}
```
执行 node cluster.js 将会得到与前文创建子进程集群的效果相同。就官方的文档而言，它更喜欢如下的形式作为示例：
```
var cluster = require('cluster');
var http = require('http');
var numCPUs = require('os').cpus().length;
if (cluster.isMaster) {
    // Fork workers  
    for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    cluster.on('exit', function (worker, code, signal) {
        console.log('worker ' + worker.process.pid + ' died');
    });
} else {
    // Workers can share any TCP connection  
    // In this case its a HTTP server   
    http.createServer(function (req, res) {
        res.writeHead(200);
        res.end("hello world\n");
    }).listen(8000);
} 
```
在进程中判断是主进程还是工作进程，主要取决于环境变量中是否有 NODE_UNIQUE_ID，如下所示：
```
cluster.isWorker = ('NODE_UNIQUE_ID' in process.env);
cluster.isMaster = (cluster.isWorker === false);
```
但是官方示例中忽而判断 cluster.isMaster、忽而判断 cluster.isWorker，对于代码的可读性十分差。建议用 cluster.setupMaster() 这个 API，将主进程和工作进程从代码上完全剥离，如同 send() 方法看起来直接将服务器从主进程发送到子进程那样神奇，剥离代码之后，甚至都感觉不到主进程中有任何服务器相关的代码。
通过 cluster.setupMaster() 创建子进程而不是使用 cluster.fork()，程序结构不再凌乱，逻辑分明，代码的可读性和可维护性较好。
#### 4.1、Cluster 工作原理
事实上 cluster 模块就是 child_process 和 net 模块的组合应用。cluster 启动时，会在内部启动 TCP 服务器，在 cluster.fork() 子进程时，将这个 TCP 服务器端 socket 的文件描述符发送给工作进程。如果进程是通过 cluster.fork() 复制出来的，那么它的环境变量里就存在 NODE_UNIQUE_ID，如果工作进程中存在 listen() 侦听网络端口的调用，它将拿到该文件描述符，通过 SO_REUSEADDR 端口重用，从而实现多个子进程共享端口。对于普通方式启动的进程，则不存在文件描述符传递共享等事情。
在 cluster 内部隐式创建 TCP 服务器的方式对使用者来说十分透明，但也正是这种方式使得它无法如直接使用 child_process 那样灵活。在 cluster 模块应用中，一个主进程只能管理一组工作进程。
对于自行通过 child_process 来操作时，则可以更灵活地控制工作进程，甚至控制多组工作进程。其原因在于自行通过 child_process 操作子进程时，可以隐式地创建多个 TCP 服务器，使得子进程可以共享多个的服务器端 socket。
#### 4.2、Cluster事件
对于健壮性处理，cluster 模块也暴露了相当多的事件。
- fork：复制一个工作进程后触发该事件。
- online：复制好一个工作进程后，工作进程主动发送一条 online 消息给主进程，主进程收到消息后，触发该事件。
- listening：工作进程中调用 listen() 后，发送一条 listening 消息给主进程，主进程收到消息后，触发该事件。
- disconnect：主进程和工作进程之间 IPC 通道断开后会触发该事件。
- exit：有工作进程退出时触发该事件。
- setup：cluster.setupMaster() 执行后触发该事件。

这些事件大多跟 child_process 模块的事件相关，在进程间消息传递的基础上完成的封装。这些事件对于增强应用的健壮性已经足够了。