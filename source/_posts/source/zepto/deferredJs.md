---
title: zepto 之 deferred.js
categories:
- source
---
zepto 的 deferred 模块，实现了 promise，可使 ajax 请求摆脱回调地狱，使用链式调用。
<!--more-->
### 一、源码
```
    //提供 $.Deferred promises API. 依赖"callbacks" 模块.
    (function ($) {
        var slice = Array.prototype.slice;

        function Deferred(func) {
            var tuples = [
                    // 状态切换方法名、对应状态执行方法名、函数回调列表、最终状态
                    ["resolve", "done", $.Callbacks({once: 1, memory: 1}), "resolved"],
                    ["reject", "fail", $.Callbacks({once: 1, memory: 1}), "rejected"],
                    ["notify", "progress", $.Callbacks({memory: 1})]
                ],
                //Promise的初始状态
                state = "pending",
                /**
                 * promise只包含执行阶段的方法always(),then(),done(),fail(),progress()及辅助方法state()、promise()等。
                 * deferred则在继承promise的基础上，增加切换状态的方法，resolve()/resolveWith(),reject()/rejectWith(),notify()/notifyWith()
                 * 所以称promise是deferred的只读副本
                 * */
                promise = {
                    // 返回promise状态
                    state: function () {
                        return state
                    },
                    //成功/失败均回调调用
                    always: function () {
                        deferred.done(arguments).fail(arguments);
                        return this
                    },
                    then: function (/* fnDone [, fnFailed [, fnProgress]] */) {
                        var fns = arguments;
                        //生成新的deferred，即defer；并为旧deferred添加新的回调函数；返回新的promise对象，then方法后的回调会被添加到新的回调函数列表中
                        return Deferred(function (defer) {
                            $.each(tuples, function (i, tuple) {
                                //i==0:fnDone；i==1:fnFailed；i==2:fnProgress
                                var fn = $.isFunction(fns[i]) && fns[i];
                                //为旧deferred的done/fail/progress方法添加回调函数，回调函数不会立即执行
                                deferred[tuple[1]](function () {
                                    //旧deferred状态切换方法名触发时，调用相应的then函数参数
                                    var returned = fn && fn.apply(this, arguments);
                                    /**
                                     * 如果then的函数参数调用返回了值，而且值存在promise方法，那么执行promise方法，
                                     * 并将defer的resolve/reject/notify添加到promise的done/fail/progress中，
                                     * 如果返回的继承promise的对象状态被切换，那么defer的相应状态切换方法同时被调用
                                     * */
                                    if (returned && $.isFunction(returned.promise)) {
                                        returned.promise()
                                            .done(defer.resolve)
                                            .fail(defer.reject)
                                            .progress(defer.notify)
                                    } else {
                                        /**
                                         * 如果then方法有返回值，则新defer的所有回调函数都使用该值作为参数，否则使用旧deferred回调参数
                                         * 在非严格模式下，this为null或undefined时为被自动替换成全局对象window
                                         * */
                                        var context = this === promise ? defer.promise() : this,
                                            values = fn ? [returned] : arguments;
                                        defer[tuple[0] + "With"](context, values)
                                    }
                                })
                            });
                            fns = null
                        }).promise()
                    },
                    //如果存在obj，则将promise对象的方法赋值给obj；否则返回promise对象
                    promise: function (obj) {
                        return obj != null ? $.extend(obj, promise) : promise
                    }
                },
                deferred = {};
            //给deferred添加切换状态方法
            $.each(tuples, function (i, tuple) {
                var list = tuple[2], //回调函数列表
                    stateString = tuple[3];  //promise最终状态
                //扩展promise的done、fail、progress为Callback的add方法，使其成为回调列表，使用的时候.done(func) func就添加到了回调函数中
                promise[tuple[1]] = list.add;
                /**
                 * 切换的状态是resolve成功/reject失败；添加首组方法做预处理，修改state的值，使成功或失败互斥；
                 * disable后就算上次触发了add时还是不会立即执行，memory被设置为undefined
                 * 锁定progress回调列表，锁定后progress回调列表不能再被触发
                 * */
                if (stateString) {
                    list.add(function () {
                        state = stateString;
                        //i^1  ^异或运算符  0^1=1 1^1=0，成功或失败回调互斥，调用一方，禁用另一方
                    }, tuples[i ^ 1][2].disable, tuples[2][2].lock)
                }
                //添加切换状态方法 resolve()/resolveWith(),reject()/rejectWith(),notify()/notifyWith()
                deferred[tuple[0]] = function () {
                    deferred[tuple[0] + "With"](this === deferred ? promise : this, arguments);
                    return this
                };
                deferred[tuple[0] + "With"] = list.fireWith
            });
            //deferred包装成promise 继承promise对象的方法
            promise.promise(deferred);
            //传递了参数func，执行
            if (func) func.call(deferred, deferred);
            //返回deferred对象
            return deferred
        }

        //主要用于多异步队列处理：多异步队列都成功，执行成功方法，一个失败，执行失败方法。也可以传非异步队列对象
        $.when = function (sub) {
            var resolveValues = slice.call(arguments),
                //队列个数
                len = resolveValues.length,
                i = 0,
                //子deferred计数
                remain = len !== 1 || (sub && $.isFunction(sub.promise)) ? len : 0,
                //主def，如果是1个fn，直接以它为主def，否则建立新的Def
                deferred = remain === 1 ? sub : Deferred(),
                progressValues, progressContexts, resolveContexts,
                updateFn = function (i, ctx, val) {
                    return function (value) {
                        ctx[i] = this;
                        val[i] = arguments.length > 1 ? slice.call(arguments) : value;
                        if (val === progressValues) {
                            //如果是通知，调用主函数的通知，通知可以调用多次
                            deferred.notifyWith(ctx, val)
                        } else if (!(--remain)) {
                            //如果是成功，则需等成功计数为0，即所有子def都成功执行了，remain变为0，再调用主函数的成功
                            deferred.resolveWith(ctx, val)
                        }
                    }
                };

            //如果参数列表长度大于一，那么校验是否为promise对象，如果不是则将remain减一
            if (len > 1) {
                progressValues = new Array(len);
                progressContexts = new Array(len);
                resolveContexts = new Array(len);
                for (; i < len; ++i) {
                    if (resolveValues[i] && $.isFunction(resolveValues[i].promise)) {
                        resolveValues[i].promise()
                            .done(updateFn(i, resolveContexts, resolveValues))
                            .fail(deferred.reject) //直接挂入主def的失败通知函数,当某个子def失败时，调用主def的切换失败状态方法，执行主def的失败函数列表
                            .progress(updateFn(i, progressContexts, progressValues))
                    } else {
                        //非def，直接标记成功，减1
                        --remain
                    }
                }
            }
            //比如无参数，或者所有子队列全为非def，直接切换到成功状态，后面就算返回了promise对象，添加的回调函数也不会被触发
            if (!remain) deferred.resolveWith(resolveContexts, resolveValues);
            return deferred.promise()
        };

        $.Deferred = Deferred
    })(Zepto);
```
### 二、源码分析
#### 1、关于 this
在 apply 方法中，如果 this 的为 null，那么在非严格模式下，this 会被自动替换为全局对象 window(call 方法同)。
#### 2、关于回调函数
所有的 done、fail、always 只是往回调列表数组中添加函数而已，回调列表数组由 callbacks 模块提供。
#### 3、deferred & promise
promise 只包含执行阶段的方法 always(),then(),done(),fail(),progress() 及辅助方法 state()、promise() 等。deferred 继承了 promise 所有方法，同时增加切换状态的方法，resolve()/resolveWith(),reject()/rejectWith(),notify()/notifyWith()，所以说 promise 是 deferred 的只读副本。而且 deferred 切换状态时，会把其他状态的回调函数列表进行禁用或者上锁(callbacks 模块)，成功/失败回调切换都只能调用一次，通知回调可以调用多次，实现了 promise 规范的`只可能从“等待”转到“完成”态或者“拒绝”态，不能逆向转换，同时“完成”态和“拒绝”态不能相互转换`限制。
#### 4、then 方法
如果 then 方法返回一个新的 promise 对象，那么将新 deferred 的状态切换函数添加到返回的 promise 对象的回调函数列表中，这样当返回的 promise 的回调函数被调用时，新的 deferred 对象状态切换函数也会被调用，而且 then 方法后的回调函数同时被执行。如果返回的不是 promise 对象，那么直接调用回调函数。
#### 5、when 方法
用来处理同时多个 promise。新建一个新的 deferred 对象，如果一个 promise 的失败回调函数被执行，那么 deferred 对象直接执行失败回调函数，由于 callbacks 模块的处理，所以失败回调函数只会被调用一次；如果 promise 的成功回调函数被执行，那么将 remain 计数器减一，直到计数器减到 0(即所有的 promise 都被处理了)，deferred 对象才执行成功回调函数。
### 三、promise 规范
它的规范内容大致如下 
* 一个 promise 可能有三种状态：等待（pending）、已完成（fulfilled）、已拒绝（rejected） 
* 一个 promise 的状态只可能从“等待”转到“完成”态或者“拒绝”态，不能逆向转换，同时“完成”态和“拒绝”态不能相互转换 
* promise 必须实现 then 方法（可以说，then 就是 promise 的核心），而且 then 必须返回一个 promise，同一个 promise 的 then 可以调用多次，并且回调的执行顺序跟它们被定义时的顺序一致 
* then 方法接受两个参数，第一个参数是成功时的回调，在 promise 由“等待”态转换到“完成”态时调用，另一个是失败时的回调，在 promise 由“等待”态转换到“拒绝”态时调用。同时，then 可以接受另一个 promise 传入，也接受一个“类 then”的对象或方法，即 thenable 对象

### 四、案例
```
    $.ajax({
        type: 'GET',
        // type: 'POST',
        url: '/projects.json',
        dataType: 'json',
        timeout: 300,
        success: function(data) {
            console.log(data);
        },
        error: function(xhr, type) {
            alert('Ajax error!');
        }
    }).done(function() {
        console.info("done");
    }).fail(function() {
        console.info("fail");
    }).always(function() {
        console.info("always");
    }) //then 三个参数 第一个是成功后回掉，第二个是失败，第三个是运行中
    .then(function() {
        console.info("then1");
    }, function() {
        console.info("then2");
    }, function() {
        console.info("then3");
    });
    
    //成功回调：done always then1
    //失败回调：alert('Ajax error!')  fail always then2
```
#### 案例一
```
    var wait = function (dtd) {
        var dtd = $.Deferred(); //在函数内部，新建一个Deferred对象　　　　
        var tasks = function () {
            console.log("执行完毕！");
            dtd.resolve(); // 改变Deferred对象的执行状态   　　　　
        };
        setTimeout(tasks, 1000);
        return dtd.promise(); // 返回promise对象
        // 返回dtd.promise  因其没有resolve和reject方法，所以在外面不能该调用这两个方法改变状态
    };
    
    $.when(wait()).done(function () {
        console.log("哈哈，成功了！");
    }).fail(function () {
        console.log("出错啦！");
    });
    
    //执行完毕！
    //哈哈，成功了！
```
#### 案例二