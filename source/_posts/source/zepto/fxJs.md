---
title: zepto 之 fx.js
categories:
- source
---
zepto 的 fx 模块，封装了 CSS 的过渡和动画。
<!--more-->
### 一、源码
```
    //fx The animate()方法
    (function ($, undefined) {
        //prefix:样式前缀(-webkit-、-moz-、-o-)；eventPrefix事件前缀(webkit、''、o)
        var prefix = '', eventPrefix,
            //内核厂商
            vendors = {Webkit: 'webkit', Moz: '', O: 'o'},
            testEl = document.createElement('div'),
            //支持的过渡、动画效果
            supportedTransforms = /^((translate|rotate|scale)(X|Y|Z|3d)?|matrix(3d)?|perspective|skew(X|Y)?)$/i,
            transform,
            //过渡
            transitionProperty, transitionDuration, transitionTiming, transitionDelay,
            //动画
            animationName, animationDuration, animationTiming, animationDelay,
            cssReset = {};

        //将字符串转成css属性，如aB-->a-b
        function dasherize(str) {
            return str.replace(/([A-Z])/g, '-$1').toLowerCase()
        }

        //修正事件名(如果不支持css3标准语法，则为事件添加前缀)
        function normalizeEvent(name) {
            return eventPrefix ? eventPrefix + name : name.toLowerCase()
        }

        //如果不支持css3标准，那么为样式加上内核厂商前缀
        if (testEl.style.transform === undefined) $.each(vendors, function (vendor, event) {
            if (testEl.style[vendor + 'TransitionProperty'] !== undefined) {
                prefix = '-' + vendor.toLowerCase() + '-';
                eventPrefix = event;
                return false
            }
        });

        //将transform设置为兼容性写法
        transform = prefix + 'transform';
        cssReset[transitionProperty = prefix + 'transition-property'] =
            cssReset[transitionDuration = prefix + 'transition-duration'] =
                cssReset[transitionDelay = prefix + 'transition-delay'] =
                    cssReset[transitionTiming = prefix + 'transition-timing-function'] =
                        cssReset[animationName = prefix + 'animation-name'] =
                            cssReset[animationDuration = prefix + 'animation-duration'] =
                                cssReset[animationDelay = prefix + 'animation-delay'] =
                                    cssReset[animationTiming = prefix + 'animation-timing-function'] = '';

        $.fx = {
            //判断是否支持css3的过渡及动画，如果即不支持css3的标准语法同时不支持带前缀的兼容形式，那么判断为不支持
            off: (eventPrefix === undefined && testEl.style.transitionProperty === undefined),
            speeds: {_default: 400, fast: 200, slow: 600},
            cssPrefix: prefix,
            transitionEnd: normalizeEvent('TransitionEnd'),
            animationEnd: normalizeEvent('AnimationEnd')
        };

        /**
         * 自定义动画
         * properties:属性对象
         * duration:过渡的时间，_default/fast/slow/数字
         * ease:变化的速率曲线，ease、linear、ease-in / ease-out、ease-in-out
         * callback:回调函数
         * delay:延迟时间
         * 并不是完全意义上的校验函数参数
         * 只有'function(properties,callback)'、'function(properties,duration,callback))'、'function(properties,duration,easing,callback,delay))'
         * 'function(properties,{duration: msec, easing: type, complete: fn }'、'function(animationName, {}'几种格式
         *
         * */
        $.fn.animate = function (properties, duration, ease, callback, delay) {
            // 传参为function(properties,callback)
            if ($.isFunction(duration))
                callback = duration, ease = undefined, duration = undefined;
            // 传参为function(properties,duration，callback)
            if ($.isFunction(ease))
                callback = ease, ease = undefined;
            //传参为function(properties, {})
            if ($.isPlainObject(duration))
                ease = duration.easing, callback = duration.complete, delay = duration.delay, duration = duration.duration;
            if (duration) duration = (typeof duration == 'number' ? duration :
                    ($.fx.speeds[duration] || $.fx.speeds._default)) / 1000;
            if (delay) delay = parseFloat(delay) / 1000;
            return this.anim(properties, duration, ease, callback, delay)
        };

        $.fn.anim = function (properties, duration, ease, callback, delay) {
            var key, cssValues = {}, cssProperties, transforms = '',
                that = this, wrappedCallback, endEvent = $.fx.transitionEnd,
                fired = false;

            //修正好时间
            if (duration === undefined) duration = $.fx.speeds._default / 1000;
            //修正好延迟
            if (delay === undefined) delay = 0;
            //如果浏览器不支持动画，持续时间设为0，直接跳动画结束
            if ($.fx.off) duration = 0;
            //css3动画:keyframe animation
            if (typeof properties == 'string') {
                cssValues[animationName] = properties;
                cssValues[animationDuration] = duration + 's';
                cssValues[animationDelay] = delay + 's';
                cssValues[animationTiming] = (ease || 'linear');
                endEvent = $.fx.animationEnd;
            } else {
                cssProperties = [];
                // CSS3的过渡
                for (key in properties)
                    //用于传参时{"rotateX": "120deg"}这种形式的properties，正常情况下是"transform": "rotateX(120deg)"
                    if (supportedTransforms.test(key)) transforms += key + '(' + properties[key] + ') ';
                    else cssValues[key] = properties[key], cssProperties.push(dasherize(key));
                if (transforms) cssValues[transform] = transforms, cssProperties.push(transform);
                if (duration > 0 && typeof properties === 'object') {
                    cssValues[transitionProperty] = cssProperties.join(', ');
                    cssValues[transitionDuration] = duration + 's';
                    cssValues[transitionDelay] = delay + 's';
                    cssValues[transitionTiming] = (ease || 'linear')
                }
            }
            //动画完成后的响应函数
            wrappedCallback = function (event) {
                event = undefined;
                //动画完成后移除监听函数
                if (typeof event !== 'undefined') {
                    //如果监听函数不是设置在事件发生元素上，而是元素的祖先元素，通过向上冒泡触发的，那么不能移除
                    if (event.target !== event.currentTarget) return; // makes sure the event didn't bubble from "below"
                    $(event.target).unbind(endEvent, wrappedCallback)
                } else
                    $(this).unbind(endEvent, wrappedCallback); // triggered by setTimeout

                fired = true;
                //动画完成后，将过渡和动画样式重置为空
                $(this).css(cssReset);
                callback && callback.call(this);
            };
            //处理动画结束事件
            if (duration > 0) {
                this.bind(endEvent, wrappedCallback);
                // transitionEnd is not always firing on older Android phones
                // so make sure it gets fired
                //延时ms后执行动画，注意这里加了25ms，保持endEvent，动画先执行完。
                //旧的android手机不一定总是会调用transitionEnd回调函数，所以在动画执行完后进行判断，如果没有执行则自己手动调用
                setTimeout(function () {
                    if (fired) return;
                    wrappedCallback.call(that)
                }, ((duration + delay) * 1000) + 25)
            }
            /**
             * 主动触发页面回流，刷新DOM，让接下来设置的动画可以正确播放
             * 更改 offsetTop、offsetLeft、offsetWidth、offsetHeight；
             * scrollTop、scrollLeft、scrollWidth、scrollHeight；
             * clientTop、clientLeft、clientWidth、clientHeight；
             * getComputedStyle()、currentStyle()，这些都会触发回流。
             * 回流导致DOM重新渲染，平时要尽可能避免。
             * 但这里，为了动画即时生效播放，则主动触发回流，刷新DOM
             * */
            this.size() && this.get(0).clientLeft
            //设置样式，启动动画
            this.css(cssValues);
            //duration为0，即浏览器不支持动画的情况，直接执行动画结束，执行回调。
            if (duration <= 0) setTimeout(function () {
                that.each(function () {
                    wrappedCallback.call(this)
                })
            }, 0);
            return this
        };

        testEl = null;
    })(Zepto);
```
### 二、源码分析
#### 1、添加前缀
首先判断是否支持标准语法`testEl.style.transform === undefined`，如果不支持则遍历 Webkit、Moz、O 前缀判断是否有相应前缀语法`testEl.style[vendor + 'TransitionProperty'] !== undefined`，为属性添加相应内核前缀。如果既不支持标准语法，也不支持兼容的加前缀语法，那么设置过渡时间为 0，直接跳转到结束。
#### 2、transitionEnd & animationEnd
通过监听 transitionEnd/animationEnd 来执行过渡/动画结束回调函数。有个奇怪的现象：测试对象是google浏览器，支持标准语法，设置的是标准浏览器 css 语法，那么可以同时监听多个 animationend 事件，如果既有 animationend 也有 webkitAnimationEnd，webkitAnimationEnd 无效，单独监听 webkitAnimationEnd 是有效果的。如果浏览器设置的是兼容浏览器版本 css 语法，那么 webkitAnimationEnd 和 animationend事件均有效。案例如下:
```
    var div1 = $('#foo1');
    div1.animate({
            "width": "300px",
            "height": "300px"
        },
        'slow', 'ease-in-out',
        function () {
            console.log('div1 animate callback');
        }, '2000');
    div1[0].addEventListener('transitionend', function () {
        console.log('transitionend');
    });
    div1[0].addEventListener('webkitTransitionEnd', function () {
        console.log('webkitTransitionEnd');
    });
```
有的老 android 手机不一定总是会触发 transitionEnd/animationEnd，可以用定时器来触发，如果触发了设置标志量 fired 为 true，然后定时器设置过渡和动画都执行完再调用，执行时判断 fired，如果为 true 则不操作，否则手动调用完成回调函数。在设置的监听回调函数中，效果执行完成后会重置过渡/动画样式为空，同时移除监听。有个情况需要特别注意，如果是子元素触发冒泡上来的，那么不能移除监听和重置样式。
#### 3、页面回流
在设置了新的样式后，主动触发页面回流，刷新 DOM，让接下来设置的过渡/动画可以正常执行。