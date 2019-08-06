---
title: zepto 之 event.js
categories:
- source
---
event.js 主要用于提供注册自定义事件和手动触发事件等功能。
<!--more-->
### 一、源码
```
    //event事件(事件处理)
    (function ($) {
        var _zid = 1, undefined,
            slice = Array.prototype.slice,
            isFunction = $.isFunction,
            isString = function (obj) {
                return typeof obj == 'string'
            },
            handlers = {},//_zid: events    事件缓存池
            specialEvents = {},
            //是否支持即将获取焦点时触发函数 onfocusin 事件类似于 onfocus 事件。 主要的区别是 onfocus 事件不支持冒泡。
            focusinSupported = 'onfocusin' in window,
            focus = {focus: 'focusin', blur: 'focusout'},
            //mouseenter、mouseleave不冒泡
            hover = {mouseenter: 'mouseover', mouseleave: 'mouseout'};

        //此处标准浏览器，click、mousedown、mouseup、mousemove抛出的就是MouseEvents，应该也是对低版本IE等某些浏览器的修正
        specialEvents.click = specialEvents.mousedown = specialEvents.mouseup = specialEvents.mousemove = 'MouseEvents';

        //取元素标识符，没有设置一个返回
        function zid(element) {
            return element._zid || (element._zid = _zid++)
        }

        //根据形参条件 查找元素上事件响应函数集合
        function findHandlers(element, event, fn, selector) {
            event = parse(event);
            if (event.ns) var matcher = matcherFor(event.ns);
            /**
             * 若有event.e ,则判断事件类型是否相同，否则直接走下一步
             * 若有event.e,则判断事件命名空间是否相同 RegExp.prototype.test = function(String) {};
             * zid(handler.fn)返回handler.fn的标识，没有加一个，判断fn标识符是否相同
             * 若有selector 则判断selector是否相同
             * */
            return (handlers[zid(element)] || []).filter(function (handler) {
                return handler
                    && (!event.e || handler.e == event.e)
                    && (!event.ns || matcher.test(handler.ns))
                    && (!fn || zid(handler.fn) === zid(fn))
                    && (!selector || handler.sel == selector)
            })
        }

        /**
         * 解析事件类型 parse("click.zhutao.xiaoyu"); => Object {e: "click", ns: "xiaoyu zhutao"}
         * slice取从索引为1之后的所有项，sort对数组进行排序，join(" ")将数组变为字符串，中间插入空格
         * */
        function parse(event) {
            var parts = ('' + event).split('.');
            return {e: parts[0], ns: parts.slice(1).sort().join(' ')}
        }

        //生成命名空间的正则对象 matcherFor("xiaoyu zhutao"); => /(?:^| )xiaoyu.* ?zhutao(?: |$)/
        function matcherFor(ns) {
            return new RegExp('(?:^| )' + ns.replace(' ', ' .* ?') + '(?: |$)')
        }

        //addEventListener 的第三个参数，true - 事件句柄在捕获阶段执行，false - 默认,事件句柄在冒泡阶段执行
        function eventCapture(handler, captureSetting) {
            return handler.del &&
                (!focusinSupported && (handler.e in focus)) || !!captureSetting
        }

        //修正事件类型 focus->focusIn blur->focusOut mouseenter->mouseover  mouseleave->mouseout
        function realEvent(type) {
            return hover[type] || (focusinSupported && focus[type]) || type
        }

        //增加事件底层方法; add(element, event, callback, data, selector, delegator || autoRemove)
        function add(element, events, fn, data, selector, delegator, capture) {
            var id = zid(element), set = (handlers[id] || (handlers[id] = []));
            events.split(/\s/).forEach(function (event) {
                if (event == 'ready') return $(document).ready(fn);
                var handler = parse(event);
                handler.fn = fn;
                handler.sel = selector;
                //如果事件是mouseenter, mouseleave，模拟mouseover mouseout事件处理
                if (handler.e in hover) fn = function (e) {
                    /**
                     * relatedTarget 事件属性返回与事件的目标节点相关的节点。
                     * 对于 mouseover 事件来说，该属性是鼠标指针移到目标节点上时所离开的那个节点。
                     * 对于 mouseout 事件来说，该属性是离开目标时，鼠标指针进入的节点。
                     * 对于其他类型的事件来说，这个属性没有用。
                     * */
                    var related = e.relatedTarget;
                    //当related不在事件对象event内   表示事件已触发完成，不是在move过程中，需要执行响应函数
                    if (!related || (related !== this && !$.contains(this, related)))
                        return handler.fn.apply(this, arguments)
                };
                handler.del = delegator;
                var callback = delegator || fn;
                handler.proxy = function (e) {
                    e = compatible(e);
                    //如果某个监听函数执行了event.stopImmediatePropagation()方法,则除了该事件的冒泡行为被阻止之外(event.stopPropagation方法的作用),该元素绑定的后序事件的监听函数的执行也将被阻止.
                    if (e.isImmediatePropagationStopped()) return;
                    e.data = data;
                    //执行回调函数，context：element，arguments：event,e._args(默认是undefind，trigger()时传递的参数）
                    var result = callback.apply(element, e._args == undefined ? [e] : [e].concat(e._args));
                    //当事件响应函数返回false时，阻止浏览器默认操作和冒泡
                    if (result === false) e.preventDefault(), e.stopPropagation();
                    return result
                };
                //设置事件响应函数的索引,删除事件时，根据它来删除  delete handlers[id][handler.i]
                handler.i = set.length;
                set.push(handler);
                if ('addEventListener' in element)
                    element.addEventListener(realEvent(handler.e), handler.proxy, eventCapture(handler, capture))
            })
        }

        //删除事件,对应add
        function remove(element, events, fn, selector, capture) {
            var id = zid(element);
            (events || '').split(/\s/).forEach(function (event) {
                findHandlers(element, event, fn, selector).forEach(function (handler) {
                    // delete删除掉数组中的元素后，会把该下标出的值置为undefined,数组的长度不会变
                    delete handlers[id][handler.i]
                    if ('removeEventListener' in element)
                        element.removeEventListener(realEvent(handler.e), handler.proxy, eventCapture(handler, capture))
                })
            })
        }

        //此处不清楚要干嘛，将事件两个核心底层方法封装到event对象里，方便做Zepto插件事件扩展吧
        $.event = {add: add, remove: remove};

        //接受一个函数，然后返回一个新函数，并且这个新函数始终保持了特定的上下文(context)语境，新函数中this指向context参数
        $.proxy = function (fn, context) {
            //如果传了第3个参数，取到第3个参数以后（包含第3个参数）所有的参数数组
            var args = (2 in arguments) && slice.call(arguments, 2);
            if (isFunction(fn)) {
                //args.concat(slice.call(arguments))将代理函数的参数与$.proxy的第三个及后面可选参数合并
                var proxyFn = function () {
                    return fn.apply(context, args ? args.concat(slice.call(arguments)) : arguments)
                };
                //标记函数
                proxyFn._zid = zid(fn);
                return proxyFn
            } else if (isString(context)) { //另外一种形式，原始的function是从上下文(context)对象的特定属性读取
                if (args) {
                    args.unshift(fn[context], fn);
                    return $.proxy.apply(null, args)
                } else {
                    return $.proxy(fn[context], fn)
                }
            } else {
                throw new TypeError("expected function")
            }
        };

        $.fn.bind = function (event, data, callback) {
            return this.on(event, data, callback)
        };
        $.fn.unbind = function (event, callback) {
            return this.off(event, callback)
        };
        //添加一个处理事件到元素，当第一次执行事件以后，该事件将自动解除绑定，保证处理函数在每个元素上最多执行一次。
        $.fn.one = function (event, selector, data, callback) {
            return this.on(event, selector, data, callback, 1)
        };

        var returnTrue = function () {
                return true
            },
            returnFalse = function () {
                return false
            },
            ignoreProperties = /^([A-Z]|returnValue$|layer[XY]$|webkitMovement[XY]$)/,
            eventMethods = {
                preventDefault: 'isDefaultPrevented',
                stopImmediatePropagation: 'isImmediatePropagationStopped',
                stopPropagation: 'isPropagationStopped'
            };

        /**
         * 修正event对象
         * @param event   代理的event对象|原生event对象
         * @param source  原生event对象
         * @returns {*}
         */
        function compatible(event, source) {
            if (source || !event.isDefaultPrevented) {
                source || (source = event);
                /**
                 * 遍历，给事件添加isDefaultPrevented、isImmediatePropagationStopped、isPropagationStopped方法
                 * isDefaultPrevented:如果preventDefault()被该事件的实例调用，那么返回true。这可作为跨平台的替代原生的defaultPrevented属性，如果defaultPrevented缺失或在某些浏览器下不可靠的时候。
                 * isImmediatePropagationStopped:如果stopImmediatePropagation()被该事件的实例调用，那么返回true。Zepto在不支持该原生方法的浏览器中实现它，（例如老版本的Android）。
                 * isPropagationStopped:如果stopPropagation()被该事件的实例调用，那么返回true。
                 **/
                $.each(eventMethods, function (name, predicate) {
                    var sourceMethod = source[name];
                    event[name] = function () {
                        this[predicate] = returnTrue;
                        return sourceMethod && sourceMethod.apply(source, arguments)
                    };
                    event[predicate] = returnFalse
                });

                event.timeStamp || (event.timeStamp = Date.now());
                //如果浏览器支持defaultPrevented DOM3 EVENT提出的能否取消默认行为
                //source.defaultPrevented:判断默认事件是否已被阻止,与preventDefault()相对应,这是对各种情况的兼容
                if (source.defaultPrevented !== undefined ? source.defaultPrevented :
                        'returnValue' in source ? source.returnValue === false :
                            source.getPreventDefault && source.getPreventDefault())
                    event.isDefaultPrevented = returnTrue
            }
            //返回修正对象
            return event
        }

        //新建一个对象 封装event，创建代理对象
        function createProxy(event) {
            var key, proxy = {originalEvent: event};
            //复制event属性至proxy，ignoreProperties里包含的属性除外
            for (key in event)
                if (!ignoreProperties.test(key) && event[key] !== undefined) proxy[key] = event[key];

            return compatible(proxy, event)
        }

        $.fn.delegate = function (selector, event, callback) {
            return this.on(event, selector, callback)
        };
        $.fn.undelegate = function (selector, event, callback) {
            return this.off(event, selector, callback)
        };

        //冒泡到document.body绑定事件
        $.fn.live = function (event, callback) {
            $(document.body).delegate(this.selector, event, callback);
            return this
        };
        //在doument.body解绑事件
        $.fn.die = function (event, callback) {
            $(document.body).undelegate(this.selector, event, callback);
            return this
        };

        /**
         * 多个事件可以通过空格的字符串方式添加，或者以事件类型为键、以函数为值的对象方式。
         * 如果给定css选择器，当事件在匹配该选择器的元素上发起时，事件才会被触发
         * 如果给定data参数，这个值将在事件处理程序执行期间被作为有用的 event.data 属性
         * */
        $.fn.on = function (event, selector, data, callback, one) {
            var autoRemove, delegator, $this = this;
            //event是对象{ type: handler, type2: handler2, ... }
            if (event && !isString(event)) {
                $.each(event, function (type, fn) {
                    $this.on(type, selector, data, fn, one)
                });
                return $this
            }

            //校验调整函数参数
            if (!isString(selector) && !isFunction(callback) && callback !== false)
                callback = data, data = selector, selector = undefined;
            if (callback === undefined || data === false)
                callback = data, data = undefined;

            //如果false在回调函数的位置上作为参数传递给这个方法，它相当于传递一个函数，这个函数直接返回false。
            if (callback === false) callback = returnFalse;

            return $this.each(function (_, element) {
                if (one) autoRemove = function (e) {
                    remove(element, e.type, callback);
                    return callback.apply(this, arguments)
                };

                if (selector) delegator = function (e) {
                    //closest 从元素本身开始，逐级向上级元素匹配，并返回最先匹配selector的元素。如果给定context节点参数，那么只匹配该节点的后代元素。
                    var evt, match = $(e.target).closest(selector, element).get(0)
                    //其实还是在父元素上进行监听，只不过如果事件触发的元素不是匹配的话，不调用函数回调
                    if (match && match !== element) {
                        evt = $.extend(createProxy(e), {currentTarget: match, liveFired: element})
                        return (autoRemove || callback).apply(match, [evt].concat(slice.call(arguments, 1)))
                    }
                };

                add(element, event, callback, data, selector, delegator || autoRemove)
            })
        };
        $.fn.off = function (event, selector, callback) {
            var $this = this;
            if (event && !isString(event)) {
                $.each(event, function (type, fn) {
                    $this.off(type, selector, fn)
                });
                return $this
            }

            if (!isString(selector) && !isFunction(callback) && callback !== false)
                callback = selector, selector = undefined;

            if (callback === false) callback = returnFalse;

            return $this.each(function () {
                remove(this, event, callback, selector)
            })
        };

        //在对象集合的元素上触发指定的事件。事件可以是一个字符串类型，也可以是一个 通过$.Event 定义的事件对象。如果给定args参数，它会作为参数传递给事件函数。
        $.fn.trigger = function (event, args) {
            event = (isString(event) || $.isPlainObject(event)) ? $.Event(event) : compatible(event);
            event._args = args;
            return this.each(function () {
                // handle focus(), blur() by calling them directly
                if (event.type in focus && typeof this[event.type] == "function") this[event.type]();
                // items in the collection might not be DOM elements
                else if ('dispatchEvent' in this) this.dispatchEvent(event);
                //可能不是dom元素上触发指定事件
                else $(this).triggerHandler(event, args)
            })
        };

        // triggers event handlers on current element just as if an event occurred,
        // doesn't trigger an actual event, doesn't bubble
        $.fn.triggerHandler = function (event, args) {
            var e, result;
            this.each(function (i, element) {
                e = createProxy(isString(event) ? $.Event(event) : event);
                e._args = args;
                e.target = element;
                //找到此元素上此事件类型上的事件响应函数集，遍历，触发
                $.each(findHandlers(element, event.type || event), function (i, handler) {
                    //调用 handler.proxy执行事件
                    result = handler.proxy(e);
                    //如果event调用了immediatePropagationStopped()，终止后续事件的响应
                    if (e.isImmediatePropagationStopped()) return false
                })
            });
            return result
        };

        //给常用事件生成便捷方法
        ('focusin focusout focus blur load resize scroll unload click dblclick ' +
        'mousedown mouseup mousemove mouseover mouseout mouseenter mouseleave ' +
        'change select keydown keypress keyup error').split(' ').forEach(function (event) {
            //有callback回调，是绑定事件，否则，触发事件
            $.fn[event] = function (callback) {
                return (0 in arguments) ?
                    this.bind(event, callback) :
                    this.trigger(event)
            }
        });

        //创建Event对象
        $.Event = function (type, props) {
            //当type是个对象时 保证type为对象的属性字符串，props为对象
            if (!isString(type)) props = type, type = props.type;
            //创建自定义事件，如果是click,mousedown,mouseup mousemove创建为MouseEvent对象,bubbles设为冒泡
            var event = document.createEvent(specialEvents[type] || 'Events'), bubbles = true;
            //bubbles = !!props[name]冒泡判断；event[name] = props[name] props属性扩展到event对象上
            if (props) for (var name in props) (name == 'bubbles') ? (bubbles = !!props[name]) : (event[name] = props[name])
            //初始化event对象，type为事件名称，如click，bubbles为是否冒泡，第三个参数表示是否可以用preventDefault方法来取消默认操作
            event.initEvent(type, bubbles, true);
            return compatible(event)
        };
    })(Zepto);
```
### 二、源码分析
#### $.Event
注册自定义事件核心流程:  
1、document.createEvent(type) 创建一个新的自定义事件  
2、event.initEvent(type, bubbles, cancelable) 初始化自定义事件  
3、compatible(event) 给事件添加 isDefaultPrevented、isImmediatePropagationStopped、isPropagationStopped 方法
#### $.fn.on
on 方法主要调用原生的 target.addEventListener(type, listener[, useCapture]); 方法。
1、$.fn.one 方法是在 on 方法的回调函数中调用 remove 方法移除事件监听
2、如果 on 方法中有选择器，虽然是必须在子元素上点击才有效果，但是事件监听其实还是在父元素上，只不过在回调函数中设置了如果不是匹配的子元素触发不进行操作
#### $.fn.off
off 方法主要调用原生 target.removeEventListener(type, listener[, useCapture]); 方法，与 $.fn.on 相对
#### $.fn.trigger
trigger 方法判断元素如果是 dom 元素触发的，则调用原生 target.dispatchEvent(event) 方法；如果不是 dom 元素上触发的，那么直接手动调用回调函数。
#### handlers
事件缓存池，用于移除或者触发监听函数找到相应的回调函数，然后进行相应操作。