---
title: bootstrap 之 popover
categories:
- bootstrap
---
这节介绍下 popover(弹出框)模块的源码实现。
<!--more-->
### 一、源码
#### 1、popovers.less
```
 //
 // Popovers(弹出框)
 // --------------------------------------------------
 .popover {
   position: absolute;
   top: 0;
   left: 0;
   z-index: @zindex-popover;
   display: none;
   max-width: @popover-max-width;
   padding: 1px;
   // Our parent element can be arbitrary since popovers are by default inserted as a sibling of their target element.
   // So reset our font and text properties to avoid inheriting weird values.
   .reset-text();
   font-size: @font-size-base;
 
   background-color: @popover-bg;
   background-clip: padding-box;
   border: 1px solid @popover-fallback-border-color;
   border: 1px solid @popover-border-color;
   border-radius: @border-radius-large;
   .box-shadow(0 5px 10px rgba(0,0,0,.2));
 
   // Offset the popover to account for the popover arrow
   &.top     { margin-top: -@popover-arrow-width; }
   &.right   { margin-left: @popover-arrow-width; }
   &.bottom  { margin-top: @popover-arrow-width; }
   &.left    { margin-left: -@popover-arrow-width; }
 }
 
 .popover-title {
   margin: 0; // reset heading margin
   padding: 8px 14px;
   font-size: @font-size-base;
   background-color: @popover-title-bg;
   border-bottom: 1px solid darken(@popover-title-bg, 5%);
   border-radius: (@border-radius-large - 1) (@border-radius-large - 1) 0 0;
 }
 
 .popover-content {
   padding: 9px 14px;
 }
 
 // Arrows
 //
 // .arrow is outer, .arrow:after is inner
 
 .popover > .arrow {
   &,
   &:after {
     position: absolute;
     display: block;
     width: 0;
     height: 0;
     border-color: transparent;
     border-style: solid;
   }
 }
 .popover > .arrow {
   border-width: @popover-arrow-outer-width;
 }
 .popover > .arrow:after {
   border-width: @popover-arrow-width;
   content: "";
 }
 
 .popover {
   &.top > .arrow {
     left: 50%;
     margin-left: -@popover-arrow-outer-width;
     border-bottom-width: 0;
     border-top-color: @popover-arrow-outer-fallback-color; // IE8 fallback
     border-top-color: @popover-arrow-outer-color;
     bottom: -@popover-arrow-outer-width;
     &:after {
       content: " ";
       bottom: 1px;
       margin-left: -@popover-arrow-width;
       border-bottom-width: 0;
       border-top-color: @popover-arrow-color;
     }
   }
   &.right > .arrow {
     top: 50%;
     left: -@popover-arrow-outer-width;
     margin-top: -@popover-arrow-outer-width;
     border-left-width: 0;
     border-right-color: @popover-arrow-outer-fallback-color; // IE8 fallback
     border-right-color: @popover-arrow-outer-color;
     &:after {
       content: " ";
       left: 1px;
       bottom: -@popover-arrow-width;
       border-left-width: 0;
       border-right-color: @popover-arrow-color;
     }
   }
   &.bottom > .arrow {
     left: 50%;
     margin-left: -@popover-arrow-outer-width;
     border-top-width: 0;
     border-bottom-color: @popover-arrow-outer-fallback-color; // IE8 fallback
     border-bottom-color: @popover-arrow-outer-color;
     top: -@popover-arrow-outer-width;
     &:after {
       content: " ";
       top: 1px;
       margin-left: -@popover-arrow-width;
       border-top-width: 0;
       border-bottom-color: @popover-arrow-color;
     }
   }
 
   &.left > .arrow {
     top: 50%;
     right: -@popover-arrow-outer-width;
     margin-top: -@popover-arrow-outer-width;
     border-right-width: 0;
     border-left-color: @popover-arrow-outer-fallback-color; // IE8 fallback
     border-left-color: @popover-arrow-outer-color;
     &:after {
       content: " ";
       right: 1px;
       border-right-width: 0;
       border-left-color: @popover-arrow-color;
       bottom: -@popover-arrow-width;
     }
   }
 }
```
#### 2、popover.js
```
 /* ========================================================================
  * Bootstrap: popover.js v3.3.7(弹出框)
  * http://getbootstrap.com/javascript/#popovers
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * ======================================================================== */
 +function ($) {
   'use strict';
 
   // POPOVER PUBLIC CLASS DEFINITION
   // ===============================
   var Popover = function (element, options) {
     this.init('popover', element, options)
   }
 
   if (!$.fn.tooltip) throw new Error('Popover requires tooltip.js')
 
   Popover.VERSION  = '3.3.7'
   Popover.DEFAULTS = $.extend({}, $.fn.tooltip.Constructor.DEFAULTS, {
     placement: 'right',
     trigger: 'click',
     content: '',
     template: '<div class="popover" role="tooltip"><div class="arrow"></div><h3 class="popover-title"></h3><div class="popover-content"></div></div>'
   })
 
   // NOTE: POPOVER EXTENDS tooltip.js
   // ================================
   Popover.prototype = $.extend({}, $.fn.tooltip.Constructor.prototype)
 
   Popover.prototype.constructor = Popover
 
   Popover.prototype.getDefaults = function () {
     return Popover.DEFAULTS
   }
 
   Popover.prototype.setContent = function () {
     var $tip    = this.tip()
     var title   = this.getTitle()
     var content = this.getContent()
 
     $tip.find('.popover-title')[this.options.html ? 'html' : 'text'](title)
     $tip.find('.popover-content').children().detach().end()[ // we use append for html objects to maintain js events
       this.options.html ? (typeof content == 'string' ? 'html' : 'append') : 'text'
     ](content)
 
     $tip.removeClass('fade top bottom left right in')
 
     // IE8 doesn't accept hiding via the `:empty` pseudo selector, we have to do
     // this manually by checking the contents.
     if (!$tip.find('.popover-title').html()) $tip.find('.popover-title').hide()
   }
 
   Popover.prototype.hasContent = function () {
     return this.getTitle() || this.getContent()
   }
 
   Popover.prototype.getContent = function () {
     var $e = this.$element
     var o  = this.options
 
     return $e.attr('data-content')
       || (typeof o.content == 'function' ?
             o.content.call($e[0]) :
             o.content)
   }
 
   Popover.prototype.arrow = function () {
     return (this.$arrow = this.$arrow || this.tip().find('.arrow'))
   }
 
   // POPOVER PLUGIN DEFINITION
   // =========================
   function Plugin(option) {
     return this.each(function () {
       var $this   = $(this)
       var data    = $this.data('bs.popover')
       var options = typeof option == 'object' && option
 
       if (!data && /destroy|hide/.test(option)) return
       if (!data) $this.data('bs.popover', (data = new Popover(this, options)))
       if (typeof option == 'string') data[option]()
     })
   }
 
   var old = $.fn.popover
 
   $.fn.popover             = Plugin
   $.fn.popover.Constructor = Popover
 
   // POPOVER NO CONFLICT
   // ===================
   $.fn.popover.noConflict = function () {
     $.fn.popover = old
     return this
   }
 }(jQuery);
```
### 二、应用 & 源码分析
#### 1、应用
```
 <button type="button" class="btnbtn-default" data-container="body" data-placement="bottom" data-toggle="popover"
         data-original-title="Bootstrap弹出框标题" data-content="Bootstrap弹出框的内容">猛击我吧
 </button>
 <a href="#" class="btnbtn-default" data-container="body" data-placement="right" data-toggle="popover"
    title="Bootstrap弹出框标题" data-content="Bootstrap弹出框的内容">猛击我吧</a>
 // 不能直接通过自定义属性data-来触发，需依赖于JavaScript
 <script type="text/javascript">
   $(function () {
     $('[data-toggle="popover"]').popover();
   });
 </script>
```
#### 2、源码分析
##### 2.1、popovers.less
1、弹出框 popover 模板与 tooltip 不同，不仅有标题，还可以设置内容   
##### 2.2、popover.js  
1、popover 继承了 tooltip 的原型，只不过覆盖了部分布局方法
2、提示框 tooltip 的默认触发事件是 hover 和 focus，而弹出框 popover 是 click