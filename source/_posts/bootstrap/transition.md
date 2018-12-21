---
title: bootstrap 之 transition
categories:
- bootstrap
---
最近一段时间研究了下 bootstrap 源码，现在开始总结，先从 transition(动画)开始。
<!--more-->
### 一、源码
```
 /* ========================================================================
  * Bootstrap: transition.js v3.3.7(动画过渡)
  * http://getbootstrap.com/javascript/#transitions
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * ======================================================================== */
 
 
 +function ($) {
   'use strict';
 
   // CSS TRANSITION SUPPORT (Shoutout: http://www.modernizr.com/)
   // ============================================================
   // 判断浏览器是否支持哪种transitionEnd，如果不支持则返回false
   function transitionEnd() {
     var el = document.createElement('bootstrap')
 
     var transEndEventNames = {
       WebkitTransition : 'webkitTransitionEnd',
       MozTransition    : 'transitionend',
       OTransition      : 'oTransitionEnd otransitionend',
       transition       : 'transitionend'
     }
 
     for (var name in transEndEventNames) {
       if (el.style[name] !== undefined) {
         return { end: transEndEventNames[name] }
       }
     }
 
     return false // explicit for ie8 (  ._.)
   }
 
   // http://blog.alexmaccaw.com/css-transitions
   $.fn.emulateTransitionEnd = function (duration) {
     var called = false
     var $el = this
     $(this).one('bsTransitionEnd', function () { called = true })
     // 如果没有监听到了bsTransitionEnd，那么采用定时器触发事件
     var callback = function () { if (!called) $($el).trigger($.support.transition.end) }
     setTimeout(callback, duration)
     return this
   }
 
   // 初始化
   $(function () {
     $.support.transition = transitionEnd()
 
     if (!$.support.transition) return
 
     // 用于传入事件类型，来自定义事件。这样就能自己添加事件$el.one('bsTransitionEnd', function() {})，而不用$el.one($.support.transition.end, function() {})
     $.event.special.bsTransitionEnd = {
       bindType: $.support.transition.end,
       delegateType: $.support.transition.end,
       handle: function (e) {
         if ($(e.target).is(this)) return e.handleObj.handler.apply(this, arguments)
       }
     }
   })
 }(jQuery);
```
### 二、源码分析
#### 1、transitionEnd
遍历各浏览器兼容的transitionEnd，判断哪个是当前浏览器兼容，并返回。
#### 2、emulateTransitionEnd
当监听到bsTransitionEnd设置called标志量为true，并设置定时器，如果bsTransitionEnd没有被触发，那么采用定时器来触发transitionEnd事件。
#### 3、event.special.bsTransitionEnd
采用jQuery处理特殊的事件，将兼容的transitionEnd委托为bsTransitionEnd。