---
title: bootstrap 之 modal
categories:
- bootstrap
---
这节介绍下 modal(模态弹窗)模块的源码实现。
<!--more-->
### 一、源码
#### 1、modals.less
```
 //
 // Modals(静态弹出框)
 // --------------------------------------------------
 
 // .modal-open      - body class for killing the scroll
 // .modal           - container to scroll within
 // .modal-dialog    - positioning shell for the actual modal
 // .modal-content   - actual modal w/ bg and corners and shit
 
 // Kill the scroll on the body(让body不发生滚动)
 .modal-open {
   overflow: hidden;
 }
 
 // Container that the modal scrolls within(容器)
 .modal {
   display: none;
   overflow: hidden;
   position: fixed;
   top: 0;
   right: 0;
   bottom: 0;
   left: 0;
   z-index: @zindex-modal;
   -webkit-overflow-scrolling: touch;
 
   // Prevent Chrome on Windows from adding a focus outline. For details, see
   // https://github.com/twbs/bootstrap/pull/10951.
   outline: 0;
 
   // When fading in the modal, animate it to slide down(模态框动态效果)
   &.fade .modal-dialog {
     .translate(0, -25%);
     .transition-transform(~"0.3s ease-out");
   }
   &.in .modal-dialog { .translate(0, 0) }
 }
 .modal-open .modal {
   overflow-x: hidden;
   overflow-y: auto;
 }
 
 // Shell div to position the modal with bottom padding(模态对话框,水平居中)
 .modal-dialog {
   position: relative;
   width: auto;
   margin: 10px;
 }
 
 // Actual modal(模态框主体容器)
 .modal-content {
   position: relative;
   background-color: @modal-content-bg;
   border: 1px solid @modal-content-fallback-border-color; //old browsers fallback (ie8 etc)
   border: 1px solid @modal-content-border-color;
   border-radius: @border-radius-large;
   .box-shadow(0 3px 9px rgba(0,0,0,.5));
   background-clip: padding-box;
   // Remove focus outline from opened modal
   outline: 0;
 }
 
 // Modal background(模态框遮罩层)
 .modal-backdrop {
   position: fixed;
   top: 0;
   right: 0;
   bottom: 0;
   left: 0;
   z-index: @zindex-modal-background;
   background-color: @modal-backdrop-bg;
   // Fade for backdrop(遮罩层动画效果)
   &.fade { .opacity(0); }
   &.in { .opacity(@modal-backdrop-opacity); }
 }
 
 // Modal header(弹出框头部)
 // Top section of the modal w/ title and dismiss
 .modal-header {
   padding: @modal-title-padding;
   border-bottom: 1px solid @modal-header-border-color;
   &:extend(.clearfix all);
 }
 // Close icon(弹出框头部关闭图标)
 .modal-header .close {
   margin-top: -2px;
 }
 // Title text within header(弹出框头部标题)
 .modal-title {
   margin: 0;
   line-height: @modal-title-line-height;
 }
 
 // Modal body(弹窗框主体)
 // Where all modal content resides (sibling of .modal-header and .modal-footer)
 .modal-body {
   position: relative;
   padding: @modal-inner-padding;
 }
 
 // Footer (for actions)(弹出框脚部)
 .modal-footer {
   padding: @modal-inner-padding;
   text-align: right; // right align buttons
   border-top: 1px solid @modal-footer-border-color;
   &:extend(.clearfix all); // clear it in case folks use .pull-* classes on buttons
 
   // Properly space out buttons
   .btn + .btn {
     margin-left: 5px;
     margin-bottom: 0; // account for input[type="submit"] which gets the bottom margin like all other inputs
   }
   // but override that for button groups
   .btn-group .btn + .btn {
     margin-left: -1px;
   }
   // and override it for block buttons as well
   .btn-block + .btn-block {
     margin-left: 0;
   }
 }
 
 // Measure scrollbar width for padding body during modal show/hide(测量滚动条宽度)
 .modal-scrollbar-measure {
   position: absolute;
   top: -9999px;
   width: 50px;
   height: 50px;
   overflow: scroll;
 }
 
 // Scale up the modal(响应式模态框大小)
 @media (min-width: @screen-sm-min) {
   // Automatically set modal's width for larger viewports
   .modal-dialog {
     width: @modal-md;
     margin: 30px auto;
   }
   .modal-content {
     .box-shadow(0 5px 15px rgba(0,0,0,.5));
   }
 
   // Modal sizes
   .modal-sm { width: @modal-sm; }
 }
 @media (min-width: @screen-md-min) {
   .modal-lg { width: @modal-lg; }
 }
```
#### 2、modal.js
```
 /* ========================================================================
  * Bootstrap: modal.js v3.3.7(模态弹窗)
  * http://getbootstrap.com/javascript/#modals
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * ======================================================================== */
 
 
 +function ($) {
   'use strict';
 
   // MODAL CLASS DEFINITION(模态框构造函数)
   // ======================
   var Modal = function (element, options) {
     this.options             = options
     this.$body               = $(document.body)
     this.$element            = $(element)
     this.$dialog             = this.$element.find('.modal-dialog')
     this.$backdrop           = null
     this.isShown             = null
     this.originalBodyPad     = null
     this.scrollbarWidth      = 0
     this.ignoreBackdropClick = false
 
     if (this.options.remote) {
       this.$element
         .find('.modal-content')
         .load(this.options.remote, $.proxy(function () {
           this.$element.trigger('loaded.bs.modal')
         }, this))
     }
   }
 
   Modal.VERSION  = '3.3.7'
   Modal.TRANSITION_DURATION = 300
   Modal.BACKDROP_TRANSITION_DURATION = 150
 
   // Modules自定义属性，show: 第一次点击时是显示还是隐藏
   Modal.DEFAULTS = {
     backdrop: true,
     keyboard: true,
     show: true
   }
 
   // 模态框显示隐藏切换方法
   Modal.prototype.toggle = function (_relatedTarget) {
     return this.isShown ? this.hide() : this.show(_relatedTarget)
   }
 
   // 模态框显示
   Modal.prototype.show = function (_relatedTarget) {
     var that = this
     // 创建自定义事件 show.bs.modal 并触发
     var e    = $.Event('show.bs.modal', { relatedTarget: _relatedTarget })
     this.$element.trigger(e)
 
     // 判断是否已经成显示状态或者阻止默认事件发生
     if (this.isShown || e.isDefaultPrevented()) return
     this.isShown = true
 
     this.checkScrollbar()
     this.setScrollbar()
     this.$body.addClass('modal-open')
 
     this.escape()
     this.resize()
 
     // 为模态框上的 'X' 添加关闭回调函数
     this.$element.on('click.dismiss.bs.modal', '[data-dismiss="modal"]', $.proxy(this.hide, this))
 
     // 存在在dialog点下按钮然后移动鼠标到backdrop上松开鼠标情况，防止这种情况导致backdrop隐藏
     this.$dialog.on('mousedown.dismiss.bs.modal', function () {
       that.$element.one('mouseup.dismiss.bs.modal', function (e) {
         if ($(e.target).is(that.$element)) that.ignoreBackdropClick = true
       })
     })
 
     // 添加回调函数，等背景加载完全再执行回调函数
     this.backdrop(function () {
       var transition = $.support.transition && that.$element.hasClass('fade')
 
       // 没有父元素(例: 还未append的$("<div></div>"))，则将model附加到body上,
       if (!that.$element.parent().length) {
         that.$element.appendTo(that.$body) // don't move modals dom position
       }
 
       // 显示模态框并设置滚动到头部
       that.$element
         .show()
         .scrollTop(0)
 
       // 调整对话框
       that.adjustDialog()
 
       // 强制回流准备动画
       if (transition) {
         that.$element[0].offsetWidth // force reflow
       }
 
       // 显示模态框
       that.$element.addClass('in')
 
       // 获取焦点
       that.enforceFocus()
 
       var e = $.Event('shown.bs.modal', { relatedTarget: _relatedTarget })
 
       transition ?
         that.$dialog // wait for modal to slide in
           .one('bsTransitionEnd', function () {
             that.$element.trigger('focus').trigger(e)
           })
           .emulateTransitionEnd(Modal.TRANSITION_DURATION) :
         that.$element.trigger('focus').trigger(e)
     })
   }
 
   // 模态框隐藏
   Modal.prototype.hide = function (e) {
     if (e) e.preventDefault()
 
     e = $.Event('hide.bs.modal')
 
     this.$element.trigger(e)
 
     if (!this.isShown || e.isDefaultPrevented()) return
 
     this.isShown = false
 
     this.escape()
     this.resize()
 
     $(document).off('focusin.bs.modal')
 
     this.$element
       .removeClass('in')
       .off('click.dismiss.bs.modal')
       .off('mouseup.dismiss.bs.modal')
 
     this.$dialog.off('mousedown.dismiss.bs.modal')
 
     $.support.transition && this.$element.hasClass('fade') ?
       this.$element
         .one('bsTransitionEnd', $.proxy(this.hideModal, this))
         .emulateTransitionEnd(Modal.TRANSITION_DURATION) :
       this.hideModal()
   }
 
   // 强制让model获取焦点
   Modal.prototype.enforceFocus = function () {
     $(document)
       .off('focusin.bs.modal') // guard against infinite focus loop
       .on('focusin.bs.modal', $.proxy(function (e) {
         if (document !== e.target &&
             this.$element[0] !== e.target &&
             !this.$element.has(e.target).length) {
           this.$element.trigger('focus')
         }
       }, this))
   }
 
   // 按下esc键时，隐藏model功能
   Modal.prototype.escape = function () {
     if (this.isShown && this.options.keyboard) {
       this.$element.on('keydown.dismiss.bs.modal', $.proxy(function (e) {
         e.which == 27 && this.hide()
       }, this))
     } else if (!this.isShown) {
       this.$element.off('keydown.dismiss.bs.modal')
     }
   }
 
   // 窗口大小重新调整时，设置监听函数
   Modal.prototype.resize = function () {
     if (this.isShown) {
       $(window).on('resize.bs.modal', $.proxy(this.handleUpdate, this))
     } else {
       $(window).off('resize.bs.modal')
     }
   }
 
   // 执行模态框隐藏函数，并移除backdrop
   Modal.prototype.hideModal = function () {
     var that = this
     this.$element.hide()
     this.backdrop(function () {
       that.$body.removeClass('modal-open')
       that.resetAdjustments()
       that.resetScrollbar()
       that.$element.trigger('hidden.bs.modal')
     })
   }
 
   // 移除backdrop的dom，并且设置$backdrop为null
   Modal.prototype.removeBackdrop = function () {
     this.$backdrop && this.$backdrop.remove()
     this.$backdrop = null
   }
 
   // backdrop显示隐藏逻辑
   Modal.prototype.backdrop = function (callback) {
     var that = this
     var animate = this.$element.hasClass('fade') ? 'fade' : ''
 
     if (this.isShown && this.options.backdrop) {
       // 判断是否支持动画效果，如果$.support.transition返回false，说明不支持CSS动画
       var doAnimate = $.support.transition && animate
 
       this.$backdrop = $(document.createElement('div'))
         .addClass('modal-backdrop ' + animate)
         .appendTo(this.$body)
 
       // 为模态框添加监听函数
       this.$element.on('click.dismiss.bs.modal', $.proxy(function (e) {
         if (this.ignoreBackdropClick) {
           this.ignoreBackdropClick = false
           return
         }
         // 如果点击模态框而不是遮罩层，不做任何操作
         if (e.target !== e.currentTarget) return
         // 如果backdrop为'static'，则不隐藏模态框而是让模态框获取焦点
         this.options.backdrop == 'static'
           ? this.$element[0].focus()
           : this.hide()
       }, this))
 
       if (doAnimate) this.$backdrop[0].offsetWidth // 强制回流准备动画效果
 
       this.$backdrop.addClass('in')
 
       // 如果没有回调函数则直接返回
       if (!callback) return
 
       // 如果有回调函数，则判断是否支持动画效果，如果支持动画效果则在动画执行完再进行回调，否则直接处理
       doAnimate ?
         this.$backdrop
           .one('bsTransitionEnd', callback)
           .emulateTransitionEnd(Modal.BACKDROP_TRANSITION_DURATION) :
         callback()
 
     } else if (!this.isShown && this.$backdrop) {
       // 移除class in，使得执行遮罩层淡出动画
       this.$backdrop.removeClass('in')
 
       var callbackRemove = function () {
         that.removeBackdrop()
         callback && callback()
       }
       // 遮罩层淡出动画结束后移除遮罩层dom，再执行回调函数
       $.support.transition && this.$element.hasClass('fade') ?
         this.$backdrop
           .one('bsTransitionEnd', callbackRemove)
           .emulateTransitionEnd(Modal.BACKDROP_TRANSITION_DURATION) :
         callbackRemove()
 
     } else if (callback) {
       callback()
     }
   }
 
   // these following methods are used to handle overflowing modals(处理模态框溢出)
   Modal.prototype.handleUpdate = function () {
     this.adjustDialog()
   }
 
   // 主要是处理窗口更新时滚动条出现
   Modal.prototype.adjustDialog = function () {
     // 如果有水平滚动条
     var modalIsOverflowing = this.$element[0].scrollHeight > document.documentElement.clientHeight
 
     this.$element.css({
       paddingLeft:  !this.bodyIsOverflowing && modalIsOverflowing ? this.scrollbarWidth : '',
       paddingRight: this.bodyIsOverflowing && !modalIsOverflowing ? this.scrollbarWidth : ''
     })
   }
 
   // 重置之前为了scrollbar而做的padding处理
   Modal.prototype.resetAdjustments = function () {
     this.$element.css({
       paddingLeft: '',
       paddingRight: ''
     })
   }
 
   // 判断是否有垂直滚动条，以及测量获取滚动条宽度
   Modal.prototype.checkScrollbar = function () {
     var fullWindowWidth = window.innerWidth
     if (!fullWindowWidth) { // workaround for missing window.innerWidth in IE8
       var documentElementRect = document.documentElement.getBoundingClientRect()
       fullWindowWidth = documentElementRect.right - Math.abs(documentElementRect.left)
     }
     // 判断如果body标签的宽度小于window窗口宽度，则bodyIsOverflowing为true
     this.bodyIsOverflowing = document.body.clientWidth < fullWindowWidth
     this.scrollbarWidth = this.measureScrollbar()
   }
 
   // 设置滚动条，为body设置padding-right抵消掉滚动条宽度，因为overflow会被设置为hidden而没有了滚动条
   Modal.prototype.setScrollbar = function () {
     var bodyPad = parseInt((this.$body.css('padding-right') || 0), 10)
     this.originalBodyPad = document.body.style.paddingRight || ''
     if (this.bodyIsOverflowing) this.$body.css('padding-right', bodyPad + this.scrollbarWidth)
   }
 
   // 重置body为原先的padding
   Modal.prototype.resetScrollbar = function () {
     this.$body.css('padding-right', this.originalBodyPad)
   }
 
   // 测量获取滚动条宽度
   Modal.prototype.measureScrollbar = function () { // thx walsh
     var scrollDiv = document.createElement('div')
     scrollDiv.className = 'modal-scrollbar-measure'
     this.$body.append(scrollDiv)
     var scrollbarWidth = scrollDiv.offsetWidth - scrollDiv.clientWidth
     this.$body[0].removeChild(scrollDiv)
     return scrollbarWidth
   }
 
 
   // MODAL PLUGIN DEFINITION(模态框插件定义)
   // =======================
   function Plugin(option, _relatedTarget) {
     return this.each(function () {
       var $this   = $(this)
       var data    = $this.data('bs.modal')
       var options = $.extend({}, Modal.DEFAULTS, $this.data(), typeof option == 'object' && option)
 
       if (!data) $this.data('bs.modal', (data = new Modal(this, options)))
 
       if (typeof option == 'string') data[option](_relatedTarget)
       else if (options.show) data.show(_relatedTarget)
     })
   }
 
   var old = $.fn.modal
 
   $.fn.modal             = Plugin
   $.fn.modal.Constructor = Modal
 
   // MODAL NO CONFLICT(有冲突时，还原$.fn.modal的引用，然后返回自己)
   // =================
   $.fn.modal.noConflict = function () {
     $.fn.modal = old
     return this
   }
 
   // MODAL DATA-API(使用'data-*'自定义属性触发或者使用a标签触发)
   // ==============
   $(document).on('click.bs.modal.data-api', '[data-toggle="modal"]', function (e) {
     var $this   = $(this)
     var href    = $this.attr('href')
     // href.replace(/.*(?=#[^\s]+$)/, ''): 截取'#'号后面的字符，比如 http://www.baidu.com?p1=1&p2=2#ss => #ss
     var $target = $($this.attr('data-target') || (href && href.replace(/.*(?=#[^\s]+$)/, ''))) // strip for ie7
     // !/#/.test(href): 链接中没有'#'
     var option  = $target.data('bs.modal') ? 'toggle' : $.extend({ remote: !/#/.test(href) && href }, $target.data(), $this.data())
 
     if ($this.is('a')) e.preventDefault()
 
     $target.one('show.bs.modal', function (showEvent) {
       if (showEvent.isDefaultPrevented()) return // only register focus restorer if modal will actually get shown
       $target.one('hidden.bs.modal', function () {
         $this.is(':visible') && $this.trigger('focus')
       })
     })
     Plugin.call($target, option, this)
   })
 }(jQuery);
```
### 二、应用 & 源码分析
#### 1、应用
```
 <h2>创建模态框（Modal）</h2>
 <!-- 按钮触发模态框 -->
 <button class="btn btn-primary btn-lg" data-toggle="modal" data-target="#myModal">开始演示模态框</button>
 <!-- 模态框（Modal） -->
 <div class="modal fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
     <div class="modal-dialog">
         <div class="modal-content">
             <div class="modal-header">
                 <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
                 <h4 class="modal-title" id="myModalLabel">模态框（Modal）标题</h4>
             </div>
             <div class="modal-body">在这里添加一些文本</div>
             <div class="modal-footer">
                 <button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>
                 <button type="button" class="btn btn-primary">提交更改</button>
             </div>
         </div><!-- /.modal-content -->
     </div><!-- /.modal-dialog -->
 </div><!-- /.modal-modal -->
```
#### 2、源码分析
##### 2.1、modals.less
1、可在 modal 最外层容器添加 fade 类实现动画效果   
2、modal 为最外层容器，fixed 布局充满屏幕；modal-dialog 为对话框显示内容，水平居中；modal-content 为对话框内容容器；
modal-header 为对话框头部容器，可添加 modal-title 内容标题和 close 图标；modal-body 为内容文本；modal-footer 为对
话框底部容器，可添加 btn 操作按钮 
##### 2.2、modal.js  
1、不依赖于 js 触发，直接通过 data-toggle="modal" 属性来初始化modal，当然也可以通过 js 触发；   
2、data-target 属性值为目标 modal 的 CSS 选择器，同时为 data-dismiss 设置监听函数，点击执行隐藏操作；   
3、如果存在自定义属性 keyboard，那么监听 esc 按键，当模态框弹出时，按键隐藏；   
4、modal 显示时，先加载 backdrop 背景完全(等待动画执行完成)再执行 modal显示(默认使用 translate 移动到 Y 轴上部，显示时设置 y = 0 )动画；隐藏时则相反