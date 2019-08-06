---
title: zepto 之 touch.js
categories:
- source
---
touch.js 主要提供滑动(swipe)与点击(tap：模拟 click)的事件封装，针对手机常用浏览器(touchstart,touchmove,touchend)和 IE10(msPointDown) 的触摸事件兼容处理以及手势的事件处理。
<!--more-->
### 一、源码
```
    //touch事件 在触摸设备上触发tap–和swipe–相关事件。这适用于所有的`touch`(iOS, Android)和`pointer`事件(Windows Phone)。
    (function ($) {
        var touch = {},
            //延迟触发(ms): 250, 0, 0, 750
            touchTimeout, tapTimeout, swipeTimeout, longTapTimeout,
            longTapDelay = 750,
            gesture;

        //判断滑动方向，返回Left, Right, Up, Down
        function swipeDirection(x1, x2, y1, y2) {
            return Math.abs(x1 - x2) >=
            Math.abs(y1 - y2) ? (x1 - x2 > 0 ? 'Left' : 'Right') : (y1 - y2 > 0 ? 'Up' : 'Down')
        }

        //长按
        function longTap() {
            //定时器执行了没有清空的必要
            longTapTimeout = null;
            if (touch.last) {
                touch.el.trigger('longTap');
                touch = {}
            }
        }

        //取消长按
        function cancelLongTap() {
            if (longTapTimeout) clearTimeout(longTapTimeout);
            longTapTimeout = null
        }

        //取消所有
        function cancelAll() {
            if (touchTimeout) clearTimeout(touchTimeout);
            if (tapTimeout) clearTimeout(tapTimeout);
            if (swipeTimeout) clearTimeout(swipeTimeout);
            if (longTapTimeout) clearTimeout(longTapTimeout);
            touchTimeout = tapTimeout = swipeTimeout = longTapTimeout = null;
            touch = {}
        }

        //判断是否是点击指针是否为主指针(http://www.ayqy.net/blog/html5%E8%A7%A6%E6%91%B8%E4%BA%8B%E4%BB%B6/)
        function isPrimaryTouch(event) {
            return (event.pointerType == 'touch' ||
                event.pointerType == event.MSPOINTER_TYPE_TOUCH)
                && event.isPrimary
        }

        //判断是否为鼠标事件或者ie的点击事件
        function isPointerEventType(e, type) {
            return (e.type == 'pointer' + type ||
            e.type.toLowerCase() == 'mspointer' + type)
        }

        //加载完全时执行，将事件绑定到全局的document
        $(document).ready(function () {
            var now, delta, deltaX = 0, deltaY = 0, firstTouch, _isPointerType;

            //IE的手势
            if ('MSGesture' in window) {
                gesture = new MSGesture();
                gesture.target = document.body
            }

            $(document)
                .bind('MSGestureEnd', function (e) { //处理IE手势结束
                    var swipeDirectionFromVelocity =
                        e.velocityX > 1 ? 'Right' : e.velocityX < -1 ? 'Left' : e.velocityY > 1 ? 'Down' : e.velocityY < -1 ? 'Up' : null;
                    if (swipeDirectionFromVelocity) {
                        touch.el.trigger('swipe');
                        touch.el.trigger('swipe' + swipeDirectionFromVelocity)
                    }
                })
                .on('touchstart MSPointerDown pointerdown', function (e) { //处理手指接触事件
                    //屏蔽掉非触摸设备(非触屏设备鼠标事件isPrimary属性为true)
                    if ((_isPointerType = isPointerEventType(e, 'down')) && !isPrimaryTouch(e)) return;
                    //记录起点坐标(_isPointerType用于ie)
                    firstTouch = _isPointerType ? e : e.touches[0];
                    //重置终点坐标(正常情况下touch会在touchend或cancelAll中清空，除了因为preventDefault等导致touchcancel不被触发)
                    if (e.touches && e.touches.length === 1 && touch.x2) {
                        touch.x2 = undefined;
                        touch.y2 = undefined
                    }
                    //当前时间毫秒数
                    now = Date.now();
                    //距离上次触碰的时间差
                    delta = now - (touch.last || now);
                    //点击元素 如果是正常的dom元素，则赋值于touch.el；若是非正常元素，比如伪元素，则touch源是其父元素
                    touch.el = $('tagName' in firstTouch.target ? firstTouch.target : firstTouch.target.parentNode);
                    //重置touch延迟事件处理器
                    touchTimeout && clearTimeout(touchTimeout);
                    //记录点击起点坐标
                    touch.x1 = firstTouch.pageX;
                    touch.y1 = firstTouch.pageY;
                    //判断是否双击
                    if (delta > 0 && delta <= 250) touch.isDoubleTap = true;
                    //将当期时间毫秒数存储到touch的last属性上
                    touch.last = now;
                    //注册长按延迟事件处理器
                    longTapTimeout = setTimeout(longTap, longTapDelay);
                    //支持IE手势识别
                    if (gesture && _isPointerType) gesture.addPointer(e.pointerId);
                })
                .on('touchmove MSPointerMove pointermove', function (e) { //处理手指滑动
                    if ((_isPointerType = isPointerEventType(e, 'move')) && !isPrimaryTouch(e)) return;
                    firstTouch = _isPointerType ? e : e.touches[0];
                    //取消长按事件处理器(touchstart设置的)
                    cancelLongTap();
                    //设置touch对象的位置
                    touch.x2 = firstTouch.pageX;
                    touch.y2 = firstTouch.pageY;
                    deltaX += Math.abs(touch.x1 - touch.x2);
                    deltaY += Math.abs(touch.y1 - touch.y2);
                })
                .on('touchend MSPointerUp pointerup', function (e) { //处理手指离开
                    if ((_isPointerType = isPointerEventType(e, 'up')) && !isPrimaryTouch(e)) return;
                    //取消长按事件处理器(touchstart设置的)
                    cancelLongTap();
                    //swipe 判定滑动动作（起点 - 终点的横向或者纵向距离超过30px）
                    if ((touch.x2 && Math.abs(touch.x1 - touch.x2) > 30) ||
                        (touch.y2 && Math.abs(touch.y1 - touch.y2) > 30))
                        //延迟执行所以可以在scroll事件触发时取消执行swipe事件
                        swipeTimeout = setTimeout(function () {
                            //根据方向触发滑动事件，并清空touch对象(所有touchend的回调触发自定义事件都作了判断是否touch对象有el属性，屏蔽比如滚动时同时触发了滑动事件等奇怪的现象)
                            if (touch.el) {
                                touch.el.trigger('swipe');
                                touch.el.trigger('swipe' + (swipeDirection(touch.x1, touch.x2, touch.y1, touch.y2)))
                            }
                            touch = {}
                        }, 0);
                    //一般情况下，touch是会存在last属性的；如果长按，那么touch会被赋空值，就不存在last属性了
                    else if ('last' in touch)
                        //当位置发生改变超过30px，不会被触发(比如移动到一个位置然后返回原先的位置)
                        if (deltaX < 30 && deltaY < 30) {
                            //立即准备执行轻触，不立即执行是为了scroll时能取消执行轻触(tap触发在scroll之前)
                            tapTimeout = setTimeout(function () {
                                // 触发全局tap，cancelTouch可以取消singleTap，doubleTap事件，以求更快响应轻触
                                var event = $.Event('tap');
                                event.cancelTouch = cancelAll;
                                // [by paper] fix -> "TypeError: 'undefined' is not an object (evaluating 'touch.el.trigger'), when double tap
                                if (touch.el) touch.el.trigger(event);
                                //立即触发doubleTap
                                if (touch.isDoubleTap) {
                                    if (touch.el) touch.el.trigger('doubleTap');
                                    touch = {}
                                }
                                // 250ms后触发singleTap，因为需要判断tap后250内是否还有tap，如果有则触发doubleTap；否则触发singleTap
                                else {
                                    touchTimeout = setTimeout(function () {
                                        //定时器已经执行了，没有必要clear
                                        touchTimeout = null;
                                        if (touch.el) touch.el.trigger('singleTap');
                                        touch = {}
                                    }, 250)
                                }
                            }, 0)
                        } else {
                            /**
                             * 当位置距离总和deltaX或deltaY大于30，并且起点到终点距离小于30
                             * 即存在这样一种情况，移动到一个位置，然后返回原先的位置
                             * 则直接清空touch对象，并不执行任何操作
                             * */
                            touch = {}
                        }
                    //重置横向，纵向滑动距离
                    deltaX = deltaY = 0;
                })
                //当浏览器窗口失去焦点(如模态框弹窗显示)，取消所有将要发生的事件
                .on('touchcancel MSPointerCancel pointercancel', cancelAll);

            //浏览器窗口滚动时，取消所有将要发生的事件，因为滚动window意味着用户的意图是滚动而不是tap或者swipe
            $(window).on('scroll', cancelAll)
        });

        //绑定自定义回调函数到自定义事件上
        ['swipe', 'swipeLeft', 'swipeRight', 'swipeUp', 'swipeDown',
            'doubleTap', 'tap', 'singleTap', 'longTap'].forEach(function (eventName) {
            $.fn[eventName] = function (callback) {
                return this.on(eventName, callback)
            }
        })
    })(Zepto);
```
### 二、源码分析
#### 1、MSPointerDown/pointerdown
由于 IE 浏览器的触碰事件使用的是 MSPointerDown/pointerdown 等这样的鼠标事件来标识的，所以为了兼容IE浏览器也需要监听此类事件。在 webkit 内核的移动端浏览器上 touch.js 可以正常运行，但是在 chrome 浏览器模拟移动端时会被触发两次，一次是监听到鼠标事件(pointerdown)，一次是监听到 touch 事件
#### 2、longTap
原生 JS 并没有提供 longTap 之类事件，所以 zepto 自己封装实现了此类事件。首先在 touchstart 时设置一个定时器 750ms，如果在此区间没有触发 touchmove、touchend、touchcancel 以及 scroll 事件，那么说明是长按事件，触发相应回调。
#### 3、doubleTap
zepto 同时也封装实现了 doubleTap。在 touchstart 时设置延迟变量为这次与上次点击时间差，如果大于 0(第一次点击为 0)并且小于 250，那么设置touch 对象双击标志量 isDoubleTap 为 true，并在 touchend 触发 doubleTap 事件。
#### 4、tap vs swipe
在 touchend 中判断起点坐标与终点坐标距离如果超过了 30px，那么触发 swipe 事件，并触发 swipe 相应方向事件。tap 需判断累计距离不超过 30px，如果超过 30px 可能存在移动到某个点再返回回来这种情况。
#### 5、singleTap
在 touchend 中设置定时器 250ms，如果在此区间再次点击则清掉定时器，否则触发 singleTap 事件。
#### 6、scroll
窗口滚动时，可以监听到 scroll 事件。