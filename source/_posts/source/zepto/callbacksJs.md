---
title: zepto 之 callbacks.js
categories:
- source
---
callbacks 用来管理回调函数，也作为 deferred 延迟对象的基础部分。
<!--more-->
### 一、源码
```
    //为"deferred"模块提供 $.Callbacks。
    (function ($) {
        // Create a collection of callbacks to be fired in a sequence, with configurable behaviour
        // Option flags:
        //   - once: 回调只能触发一次
        //   - memory: 记住最近的上下文和参数，如果memory为true则添加的时候会执行一次
        //   - stopOnFalse: 当某个回调函数返回false时中断执行
        //   - unique: 一个回调函数只能被添加一次
        $.Callbacks = function (options) {
            options = $.extend({}, options);

            var memory, // 记录上一次触发回调函数列表时的参数，之后添加的函数都用这参数立即执行
                fired,  // 是否回调过标志量
                firing,  // 回调函数列表是否正在执行中
                firingStart, // 开始回调函数的下标
                firingLength, // 回调函数列表长度
                firingIndex, // 回调列表索引值
                list = [], // 回调数据源:回调列表
                stack = !options.once && [], //回调只能触发一次的时候，stack永远为false；多次触发永远为数组
                //触发回调底层函数
                fire = function (data) {
                    memory = options.memory && data;
                    fired = true;
                    firingIndex = firingStart || 0;
                    firingStart = 0;
                    firingLength = list.length;
                    firing = true;
                    //遍历回调列表，全部回调函数都执行，参数是传递过来的data
                    for (; list && firingIndex < firingLength; ++firingIndex) {
                        //如果 list[ firingIndex ] 为false，且stopOnFalse（中断）模式，则中断回掉执行，设置memory为false
                        if (list[firingIndex].apply(data[0], data[1]) === false && options.stopOnFalse) {
                            memory = false;
                            break
                        }
                    }
                    //回调执行完毕
                    firing = false;
                    if (list) {
                        //stack里还缓存有未执行的回调，则执行stack里的回调
                        if (stack) stack.length && fire(stack.shift());
                        //如果只执行一次而且memory为true(类型转换),那么清空回调函数(不禁用是因为设置了memory，add执行时会调用)
                        else if (memory) list.length = 0;
                        //如果只执行一次而且memory为false(类型转换),那么禁用回调函数
                        else Callbacks.disable();
                    }
                },
                Callbacks = {
                    //添加回调函数
                    add: function () {
                        if (list) {
                            var start = list.length,
                                add = function (args) {
                                    $.each(args, function (_, arg) {
                                        if (typeof arg === "function") {
                                            //非unique，或者是unique，但回调列表未添加过
                                            if (!options.unique || !Callbacks.has(arg)) list.push(arg)
                                        }
                                        //是数组/伪数组，添加，重新遍历
                                        else if (arg && arg.length && typeof arg !== 'string') add(arg)
                                    })
                                };
                            //添加进列表
                            add(arguments);
                            //如果列表正在执行中，修正长度，使得新添加的回调也可以执行
                            if (firing) firingLength = list.length;
                            else if (memory) {
                                //memory 模式下，修正开始下标
                                firingStart = start;
                                //立即执行所有回调
                                fire(memory)
                            }
                        }
                        return this
                    },
                    //从回调列表里删除一个或一组回调函数
                    remove: function () {
                        if (list) {
                            $.each(arguments, function (_, arg) {
                                var index;
                                while ((index = $.inArray(arg, list, index)) > -1) {
                                    list.splice(index, 1);
                                    // Handle firing indexes
                                    if (firing) {
                                        //避免回调列表溢出
                                        if (index <= firingLength) --firingLength;
                                        //如果移除的函数已经执行过了，则将迭代下标减一，否则会漏掉回调函数没执行
                                        if (index <= firingIndex) --firingIndex
                                    }
                                }
                            })
                        }
                        return this
                    },
                    //检查指定的回调函数是否在回调列表中；如果参数为空，则方法用来表明是否存在回调函数
                    has: function (fn) {
                        return !!(list && (fn ? $.inArray(fn, list) > -1 : list.length))
                    },
                    //清空回调函数
                    empty: function () {
                        firingLength = list.length = 0;
                        return this
                    },
                    //禁用回调函数
                    disable: function () {
                        list = stack = memory = undefined;
                        return this
                    },
                    //是否禁用了回调函数
                    disabled: function () {
                        return !list
                    },
                    //锁定回调函数
                    lock: function () {
                        stack = undefined;
                        //非memory模式下，禁用列表
                        if (!memory) Callbacks.disable();
                        return this
                    },
                    //是否是锁定的(一次性调用的时，为true)
                    locked: function () {
                        return !stack
                    },
                    //用上下文、参数执行列表中的所有回调函数
                    fireWith: function (context, args) {
                        //如果调用过一次了fired被设置为true，如果设置once为true的话(可类型转换)，则不执行回调函数
                        if (list && (!fired || stack)) {
                            args = args || [];
                            args = [context, args.slice ? args.slice() : args];
                            //正在回调中,存入stack
                            if (firing) stack.push(args);
                            //否则立即回调,外层fire函数
                            else fire(args);
                        }
                        return this
                    },
                    //执行回调
                    fire: function () {
                        return Callbacks.fireWith(this, arguments)
                    },
                    //回调列表是否被回调过
                    fired: function () {
                        return !!fired
                    }
                };
            return Callbacks
        }
    })(Zepto);
```
### 二、源码解析
#### 1、Option once
设置一个标志量为 fired，第一次调用 fire 方法时设置为 true，下次再次调用时判断 stack(由 once 值决定)决定是否执行回调列表函数；
#### 2、Option memory
记住上次 fire 的回调参数，下次 add 时判断 memory 值，如果为 true(类型转换)则将 add 方法参数里面的回调函数执行一次，参数为 memory；
#### 3、数组移除
使用 $.inArray(arg, list, index)) 判断是否存在数组中，并且记录索引值；如果出现了，则移除并从当前记住索引值开始继续判断。