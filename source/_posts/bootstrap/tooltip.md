---
title: bootstrap 之 tooltip
categories:
- bootstrap
---
这节介绍下 tooltip(提示框)模块的源码实现。
<!--more-->
### 一、源码
#### 1、tooltip.less
```
 //
 // Tooltips(提示框)
 // --------------------------------------------------
 
 // Base class(基础样式)
 .tooltip {
   position: absolute;
   z-index: @zindex-tooltip;
   display: block;
   // Our parent element can be arbitrary since tooltips are by default inserted as a sibling of their target element.
   // So reset our font and text properties to avoid inheriting weird values.
   .reset-text();
   font-size: @font-size-small;
 
   .opacity(0);
 
   &.in     { .opacity(@tooltip-opacity); }
   &.top    { margin-top:  -3px; padding: @tooltip-arrow-width 0; }
   &.right  { margin-left:  3px; padding: 0 @tooltip-arrow-width; }
   &.bottom { margin-top:   3px; padding: @tooltip-arrow-width 0; }
   &.left   { margin-left: -3px; padding: 0 @tooltip-arrow-width; }
 }
 
 // Wrapper for the tooltip content(提示框文本内容)
 .tooltip-inner {
   max-width: @tooltip-max-width;
   padding: 3px 8px;
   color: @tooltip-color;
   text-align: center;
   background-color: @tooltip-bg;
   border-radius: @border-radius-base;
 }
 
 // Arrows(提示框箭头)
 .tooltip-arrow {
   position: absolute;
   width: 0;
   height: 0;
   border-color: transparent;
   border-style: solid;
 }
 // Note: Deprecated .top-left, .top-right, .bottom-left, and .bottom-right as of v3.3.1(提示框不同方向的箭头指向)
 .tooltip {
   &.top .tooltip-arrow {
     bottom: 0;
     left: 50%;
     margin-left: -@tooltip-arrow-width;
     border-width: @tooltip-arrow-width @tooltip-arrow-width 0;
     border-top-color: @tooltip-arrow-color;
   }
   &.top-left .tooltip-arrow {
     bottom: 0;
     right: @tooltip-arrow-width;
     margin-bottom: -@tooltip-arrow-width;
     border-width: @tooltip-arrow-width @tooltip-arrow-width 0;
     border-top-color: @tooltip-arrow-color;
   }
   &.top-right .tooltip-arrow {
     bottom: 0;
     left: @tooltip-arrow-width;
     margin-bottom: -@tooltip-arrow-width;
     border-width: @tooltip-arrow-width @tooltip-arrow-width 0;
     border-top-color: @tooltip-arrow-color;
   }
   &.right .tooltip-arrow {
     top: 50%;
     left: 0;
     margin-top: -@tooltip-arrow-width;
     border-width: @tooltip-arrow-width @tooltip-arrow-width @tooltip-arrow-width 0;
     border-right-color: @tooltip-arrow-color;
   }
   &.left .tooltip-arrow {
     top: 50%;
     right: 0;
     margin-top: -@tooltip-arrow-width;
     border-width: @tooltip-arrow-width 0 @tooltip-arrow-width @tooltip-arrow-width;
     border-left-color: @tooltip-arrow-color;
   }
   &.bottom .tooltip-arrow {
     top: 0;
     left: 50%;
     margin-left: -@tooltip-arrow-width;
     border-width: 0 @tooltip-arrow-width @tooltip-arrow-width;
     border-bottom-color: @tooltip-arrow-color;
   }
   &.bottom-left .tooltip-arrow {
     top: 0;
     right: @tooltip-arrow-width;
     margin-top: -@tooltip-arrow-width;
     border-width: 0 @tooltip-arrow-width @tooltip-arrow-width;
     border-bottom-color: @tooltip-arrow-color;
   }
   &.bottom-right .tooltip-arrow {
     top: 0;
     left: @tooltip-arrow-width;
     margin-top: -@tooltip-arrow-width;
     border-width: 0 @tooltip-arrow-width @tooltip-arrow-width;
     border-bottom-color: @tooltip-arrow-color;
   }
 }
```
#### 2、tooltip.js
```
 /* ========================================================================
  * Bootstrap: tooltip.js v3.3.7(提示框)
  * http://getbootstrap.com/javascript/#tooltip
  * Inspired by the original jQuery.tipsy by Jason Frame
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * ======================================================================== */
 +function ($) {
   'use strict';
 
   // TOOLTIP PUBLIC CLASS DEFINITION(提示框定义)
   // ===============================
   var Tooltip = function (element, options) {
     this.type       = null  // 插件类型
     this.options    = null  // 自定义属性
     this.enabled    = null  // 标志量，控制tooltip是否可以显示
     this.timeout    = null  // 定时器
     this.hoverState = null  // 提示框显示状态
     this.$element   = null  // 对象绑定的元素
     this.inState    = null  // 操作状态
 
     this.init('tooltip', element, options)
   }
 
   Tooltip.VERSION  = '3.3.7'
 
   Tooltip.TRANSITION_DURATION = 150
 
   // 提示框默认设置
   Tooltip.DEFAULTS = {
     animation: true,  // 是否使用动画
     placement: 'top',  // 设置提示框内容位置信息
     selector: false,  // CSS选择器，用于事件委托
     template: '<div class="tooltip" role="tooltip"><div class="tooltip-arrow"></div><div class="tooltip-inner"></div></div>',  // 提示框模板
     trigger: 'hover focus',  // 触发方法
     title: '',  // 提示框标题内容
     delay: 0,  // 提示框显示隐藏延时时间
     html: false,  // 设置'tooltip-inner'内内容html片段还是text纯文本
     container: false,  // 提示框容器
     viewport: {
       selector: 'body',
       padding: 0
     }
   }
 
   // 提示框初始化
   Tooltip.prototype.init = function (type, element, options) {
     this.enabled   = true
     this.type      = type
     this.$element  = $(element)
     // 获取自定义选项
     this.options   = this.getOptions(options)
     // 获取视口，保证提示框在元素的边缘内
     this.$viewport = this.options.viewport && $($.isFunction(this.options.viewport) ? this.options.viewport.call(this, this.$element) : (this.options.viewport.selector || this.options.viewport))
     // 显示条件存储对象
     this.inState   = { click: false, hover: false, focus: false }
 
     // 不能直接为document添加提示框，需要添加CSS选择符来选中document的子元素
     if (this.$element[0] instanceof document.constructor && !this.options.selector) {
       throw new Error('`selector` option must be specified when initializing ' + this.type + ' on the window.document object!')
     }
 
     // 处理不同触发情况
     var triggers = this.options.trigger.split(' ')
     for (var i = triggers.length; i--;) {
       var trigger = triggers[i]
 
       if (trigger == 'click') {
         // 点击触发调用toggle方法
         this.$element.on('click.' + this.type, this.options.selector, $.proxy(this.toggle, this))
       } else if (trigger != 'manual') {
         var eventIn  = trigger == 'hover' ? 'mouseenter' : 'focusin'
         var eventOut = trigger == 'hover' ? 'mouseleave' : 'focusout'
 
         // hover或者focus触发调用enter/leave方法
         this.$element.on(eventIn  + '.' + this.type, this.options.selector, $.proxy(this.enter, this))
         this.$element.on(eventOut + '.' + this.type, this.options.selector, $.proxy(this.leave, this))
       }
     }
 
     this.options.selector ?
       (this._options = $.extend({}, this.options, { trigger: 'manual', selector: '' })) :
       this.fixTitle()
   }
 
   // 返回默认自定义选项
   Tooltip.prototype.getDefaults = function () {
     return Tooltip.DEFAULTS
   }
 
   // 获取自定义选项
   Tooltip.prototype.getOptions = function (options) {
     options = $.extend({}, this.getDefaults(), this.$element.data(), options)
 
     if (options.delay && typeof options.delay == 'number') {
       options.delay = {
         show: options.delay,
         hide: options.delay
       }
     }
 
     return options
   }
 
   // 获取委托的自定义选项
   Tooltip.prototype.getDelegateOptions = function () {
     var options  = {}
     var defaults = this.getDefaults()
 
     this._options && $.each(this._options, function (key, value) {
       if (defaults[key] != value) options[key] = value
     })
 
     return options
   }
 
   // 提示框显示
   Tooltip.prototype.enter = function (obj) {
     // 要不通过toggle方法来触发的enter，要不通过hover/focus触发的，所以obj要不为event对象，要不为tooltip实例对象
     var self = obj instanceof this.constructor ?
       obj : $(obj.currentTarget).data('bs.' + this.type)
 
     // 如果不存在self，说明是hover/focus触发，而且是委托
     if (!self) {
       self = new this.constructor(obj.currentTarget, this.getDelegateOptions())
       $(obj.currentTarget).data('bs.' + this.type, self)
     }
 
     // 如果是obj是event对象，那么切换focus/hover状态
     if (obj instanceof $.Event) {
       self.inState[obj.type == 'focusin' ? 'focus' : 'hover'] = true
     }
 
     // 如果提示框模板元素已经显示，那么不做任何操作直接返回
     if (self.tip().hasClass('in') || self.hoverState == 'in') {
       self.hoverState = 'in'
       return
     }
 
     clearTimeout(self.timeout)
 
     self.hoverState = 'in'
 
     if (!self.options.delay || !self.options.delay.show) return self.show()
 
     // 根据定时器时间来执行提示框显示操作
     self.timeout = setTimeout(function () {
       // 如果定时器执行时，状态已经被切换了，那么需要重新根据状态判断是否继续进行操作
       if (self.hoverState == 'in') self.show()
     }, self.options.delay.show)
   }
 
   // 判断是否存在显示条件
   Tooltip.prototype.isInStateTrue = function () {
     for (var key in this.inState) {
       if (this.inState[key]) return true
     }
 
     return false
   }
 
   // 提示框隐藏
   Tooltip.prototype.leave = function (obj) {
     // 要不通过toggle方法来触发的leave，要不通过hover/focus触发的，所以obj要不为event对象，要不为tooltip实例对象
     var self = obj instanceof this.constructor ?
       obj : $(obj.currentTarget).data('bs.' + this.type)
 
     // 如果不存在self，说明是hover/focus触发，而且是委托
     if (!self) {
       self = new this.constructor(obj.currentTarget, this.getDelegateOptions())
       $(obj.currentTarget).data('bs.' + this.type, self)
     }
 
     // 如果是obj是event对象，那么切换focus/hover状态
     if (obj instanceof $.Event) {
       self.inState[obj.type == 'focusout' ? 'focus' : 'hover'] = false
     }
 
     // 如果存在还显示的状态，那么直接返回
     if (self.isInStateTrue()) return
 
     clearTimeout(self.timeout)
 
     self.hoverState = 'out'
 
     if (!self.options.delay || !self.options.delay.hide) return self.hide()
 
     self.timeout = setTimeout(function () {
       // 如果定时器执行时，状态已经被切换了，那么需要重新根据状态判断是否继续进行操作
       if (self.hoverState == 'out') self.hide()
     }, self.options.delay.hide)
   }
 
   // 显示提示框具体操作
   Tooltip.prototype.show = function () {
     // 自定义事件
     var e = $.Event('show.bs.' + this.type)
 
     // 如果存在显示标题内容而且显示控制标志量enabled为true
     if (this.hasContent() && this.enabled) {
       this.$element.trigger(e)
 
       // 判断对象对应的dom是否在文档内
       var inDom = $.contains(this.$element[0].ownerDocument.documentElement, this.$element[0])
       if (e.isDefaultPrevented() || !inDom) return
 
       var that = this
       var $tip = this.tip()  // 获取提示框DOM元素
       var tipId = this.getUID(this.type)  // 获取不存在于document中的ID
 
       this.setContent()  // 设置标题模板内容
       $tip.attr('id', tipId)  // 设置模板id
       this.$element.attr('aria-describedby', tipId)  // 设置触发元素aria-describedby属性
 
       if (this.options.animation) $tip.addClass('fade')  // 如果使用动画，添加fade类
 
       // 获取提示框位置信息
       var placement = typeof this.options.placement == 'function' ?
         this.options.placement.call(this, $tip[0], this.$element[0]) :
         this.options.placement
 
       // 如果使用auto，尽量会被替换掉，默认替换为top；如果同时存在其他方向，默认使用其他方向
       var autoToken = /\s?auto?\s?/i
       var autoPlace = autoToken.test(placement)
       if (autoPlace) placement = placement.replace(autoToken, '') || 'top'
 
       // 设置提示框样式
       $tip
         .detach()  // detach() 删除匹配的对象；与remove()不同的是，所有绑定的事件、附加的数据等都会保留下来
         .css({ top: 0, left: 0, display: 'block' })
         .addClass(placement)
         .data('bs.' + this.type, this)
 
       // 将提示框添加到文档中
       this.options.container ? $tip.appendTo(this.options.container) : $tip.insertAfter(this.$element)
       // 触发自定义事件'inserted.bs.tooltip'
       this.$element.trigger('inserted.bs.' + this.type)
 
       // 位置对象pos =  {top: , right: , bottom: , left: , width: ,scroll:}
       var pos          = this.getPosition()
       // 提示框自身实际的 width height
       var actualWidth  = $tip[0].offsetWidth
       var actualHeight = $tip[0].offsetHeight
 
       // 如果出现了auto，那么根据viewport动态调整placement
       if (autoPlace) {
         var orgPlacement = placement
         var viewportDim = this.getPosition(this.$viewport)
 
         placement = placement == 'bottom' && pos.bottom + actualHeight > viewportDim.bottom ? 'top'    :
                     placement == 'top'    && pos.top    - actualHeight < viewportDim.top    ? 'bottom' :
                     placement == 'right'  && pos.right  + actualWidth  > viewportDim.width  ? 'left'   :
                     placement == 'left'   && pos.left   - actualWidth  < viewportDim.left   ? 'right'  :
                     placement
 
         $tip
           .removeClass(orgPlacement)
           .addClass(placement)
       }
 
       // 计算提示框的偏移量 {top: , left: }
       var calculatedOffset = this.getCalculatedOffset(placement, pos, actualWidth, actualHeight)
 
       // 应用并设置偏移量
       this.applyPlacement(calculatedOffset, placement)
 
       var complete = function () {
         var prevHoverState = that.hoverState
         that.$element.trigger('shown.bs.' + that.type)
         that.hoverState = null
 
         // 其实这里不太需要，因为就算连续点击两次，后面hide方法会将hoverState设置为null；就算改成prevHoverState==null也只是连续执行两次leave
         if (prevHoverState == 'out') that.leave(that)
       }
 
       $.support.transition && this.$tip.hasClass('fade') ?
         $tip
           .one('bsTransitionEnd', complete)
           .emulateTransitionEnd(Tooltip.TRANSITION_DURATION) :
         complete()
     }
   }
 
   // 应用并设置偏移量
   Tooltip.prototype.applyPlacement = function (offset, placement) {
     var $tip   = this.tip()
     var width  = $tip[0].offsetWidth
     var height = $tip[0].offsetHeight
 
     // manually read margins because getBoundingClientRect includes difference
     // 获得提示框自身的外边距的值
     var marginTop = parseInt($tip.css('margin-top'), 10)
     var marginLeft = parseInt($tip.css('margin-left'), 10)
 
     // we must check for NaN for ie 8/9(是否是非数字)
     if (isNaN(marginTop))  marginTop  = 0
     if (isNaN(marginLeft)) marginLeft = 0
 
     // 计算的偏移量 + 自身的外边距
     offset.top  += marginTop
     offset.left += marginLeft
 
     // $.fn.offset doesn't round pixel values
     // so we use setOffset directly with our own function B-0
     // 应用了offset.setOffset方法，传入了using参数，因为offset设置值的时候，不能四舍五入
     $.offset.setOffset($tip[0], $.extend({
       using: function (props) {
         $tip.css({
           top: Math.round(props.top),
           left: Math.round(props.left)
         })
       }
     }, offset), 0)
 
     // 添加 in class 让提示框显示
     $tip.addClass('in')
 
     // check to see if placing tip in new offset caused the tip to resize itself
     // 获取显示后的提示框的宽和高，检查是否调整了自身的大小
     var actualWidth  = $tip[0].offsetWidth
     var actualHeight = $tip[0].offsetHeight
 
     if (placement == 'top' && actualHeight != height) {
       offset.top = offset.top + height - actualHeight
     }
 
     // 根据viewport调整位置
     var delta = this.getViewportAdjustedDelta(placement, offset, actualWidth, actualHeight)
 
     if (delta.left) offset.left += delta.left
     else offset.top += delta.top
 
     var isVertical          = /top|bottom/.test(placement)
     var arrowDelta          = isVertical ? delta.left * 2 - width + actualWidth : delta.top * 2 - height + actualHeight
     var arrowOffsetPosition = isVertical ? 'offsetWidth' : 'offsetHeight'
 
     $tip.offset(offset)
     this.replaceArrow(arrowDelta, $tip[0][arrowOffsetPosition], isVertical)
   }
 
   // 更新箭头位置
   Tooltip.prototype.replaceArrow = function (delta, dimension, isVertical) {
     this.arrow()
       .css(isVertical ? 'left' : 'top', 50 * (1 - delta / dimension) + '%')
       .css(isVertical ? 'top' : 'left', '')
   }
 
   // 设置内容，并移除上面的
   Tooltip.prototype.setContent = function () {
     var $tip  = this.tip()
     var title = this.getTitle()
 
     $tip.find('.tooltip-inner')[this.options.html ? 'html' : 'text'](title)
     $tip.removeClass('fade in top bottom left right')
   }
 
   // 隐藏提示框具体操作
   Tooltip.prototype.hide = function (callback) {
     var that = this
     var $tip = $(this.$tip)
     var e    = $.Event('hide.bs.' + this.type)
 
     function complete() {
       // 如果在提示框显示时，连续两次点击(假设元素触发方法为click，而且点击快速在动画时间内)，那么不销毁提示框模板元素，如果销毁会有问题
       if (that.hoverState != 'in') $tip.detach()
       if (that.$element) { // TODO: Check whether guarding this code with this `if` is really necessary.
         that.$element
           .removeAttr('aria-describedby')
           .trigger('hidden.bs.' + that.type)
       }
       callback && callback()
     }
 
     this.$element.trigger(e)
 
     if (e.isDefaultPrevented()) return
 
     $tip.removeClass('in')
 
     $.support.transition && $tip.hasClass('fade') ?
       $tip
         .one('bsTransitionEnd', complete)
         .emulateTransitionEnd(Tooltip.TRANSITION_DURATION) :
       complete()
 
     this.hoverState = null
 
     return this
   }
 
   // 默认使用title属性作为提示框内容
   Tooltip.prototype.fixTitle = function () {
     var $e = this.$element
     if ($e.attr('title') || typeof $e.attr('data-original-title') != 'string') {
       $e.attr('data-original-title', $e.attr('title') || '').attr('title', '')
     }
   }
 
   // 是否有显示内容
   Tooltip.prototype.hasContent = function () {
     return this.getTitle()
   }
 
   // 获取位置
   Tooltip.prototype.getPosition = function ($element) {
     // 不传入则为当前点击元素
     $element   = $element || this.$element
 
     // 转换为DOM对象
     var el     = $element[0]
     // 是否是body元素
     var isBody = el.tagName == 'BODY'
     // 获取元素各边与页面上边和左边的距离 left right top bottom height width
     var elRect    = el.getBoundingClientRect()
     //兼容IE8 没有 width height 则计算
     if (elRect.width == null) {
       // width and height are missing in IE8, so compute them manually; see https://github.com/twbs/bootstrap/issues/14093
       elRect = $.extend({}, elRect, { width: elRect.right - elRect.left, height: elRect.bottom - elRect.top })
     }
     var isSvg = window.SVGElement && el instanceof window.SVGElement
     // Avoid using $.offset() on SVGs since it gives incorrect results in jQuery 3.
     // See https://github.com/twbs/bootstrap/issues/20280
     // 当前元素相对于文档的偏移量(因为SVG上得不到正确的offset结果，所以返回null)
     var elOffset  = isBody ? { top: 0, left: 0 } : (isSvg ? null : $element.offset())
     // 垂直滚动条的距离(document.documentElement.scrollTop和document.body.scrollTop 浏览器兼容)
     var scroll    = { scroll: isBody ? document.documentElement.scrollTop || document.body.scrollTop : $element.scrollTop() }
     var outerDims = isBody ? { width: $(window).width(), height: $(window).height() } : null
 
     // 合并
     return $.extend({}, elRect, scroll, outerDims, elOffset)
   }
 
   // 计算偏移量
   Tooltip.prototype.getCalculatedOffset = function (placement, pos, actualWidth, actualHeight) {
     return placement == 'bottom' ? { top: pos.top + pos.height,   left: pos.left + pos.width / 2 - actualWidth / 2 } :
            placement == 'top'    ? { top: pos.top - actualHeight, left: pos.left + pos.width / 2 - actualWidth / 2 } :
            placement == 'left'   ? { top: pos.top + pos.height / 2 - actualHeight / 2, left: pos.left - actualWidth } :
         /* placement == 'right' */ { top: pos.top + pos.height / 2 - actualHeight / 2, left: pos.left + pos.width }
 
   }
 
   // 根据视口调节
   Tooltip.prototype.getViewportAdjustedDelta = function (placement, pos, actualWidth, actualHeight) {
     var delta = { top: 0, left: 0 }
     //默认的话，this.$viewport = 'body'
     if (!this.$viewport) return delta
 
     var viewportPadding = this.options.viewport && this.options.viewport.padding || 0
     //this.$viewport = 'body' width height为window的
     var viewportDimensions = this.getPosition(this.$viewport)
 
     if (/right|left/.test(placement)) {
       var topEdgeOffset    = pos.top - viewportPadding - viewportDimensions.scroll
       var bottomEdgeOffset = pos.top + viewportPadding - viewportDimensions.scroll + actualHeight
       if (topEdgeOffset < viewportDimensions.top) { // top overflow
         delta.top = viewportDimensions.top - topEdgeOffset
       } else if (bottomEdgeOffset > viewportDimensions.top + viewportDimensions.height) { // bottom overflow
         delta.top = viewportDimensions.top + viewportDimensions.height - bottomEdgeOffset
       }
     } else {
       var leftEdgeOffset  = pos.left - viewportPadding
       var rightEdgeOffset = pos.left + viewportPadding + actualWidth
       if (leftEdgeOffset < viewportDimensions.left) { // left overflow
         delta.left = viewportDimensions.left - leftEdgeOffset
       } else if (rightEdgeOffset > viewportDimensions.right) { // right overflow
         delta.left = viewportDimensions.left + viewportDimensions.width - rightEdgeOffset
       }
     }
 
     return delta
   }
 
   // 获取标题内容
   Tooltip.prototype.getTitle = function () {
     var title
     var $e = this.$element
     var o  = this.options
 
     title = $e.attr('data-original-title')
       || (typeof o.title == 'function' ? o.title.call($e[0]) :  o.title)
 
     return title
   }
 
   // 获取'tooltipxxx'格式的id(用do..while得到不存在于document中的ID)
   Tooltip.prototype.getUID = function (prefix) {
     do prefix += ~~(Math.random() * 1000000)
     while (document.getElementById(prefix))
     return prefix
   }
 
   // 获取提示模板元素
   Tooltip.prototype.tip = function () {
     if (!this.$tip) {
       this.$tip = $(this.options.template)
       if (this.$tip.length != 1) {
         throw new Error(this.type + ' `template` option must consist of exactly 1 top-level element!')
       }
     }
     return this.$tip
   }
 
   // 获取提示箭头(tooltip-arrow类)
   Tooltip.prototype.arrow = function () {
     return (this.$arrow = this.$arrow || this.tip().find('.tooltip-arrow'))
   }
 
   // 设置enabled为true，使得提示框能够正常显示
   Tooltip.prototype.enable = function () {
     this.enabled = true
   }
 
   // 与enable相反
   Tooltip.prototype.disable = function () {
     this.enabled = false
   }
 
   // 切换enabled
   Tooltip.prototype.toggleEnabled = function () {
     this.enabled = !this.enabled
   }
 
   // 切换提示框显示状态
   Tooltip.prototype.toggle = function (e) {
     var self = this
     if (e) {
       // 如果是事件委托的话，那么提示框元素并不会存储bs.tooltip属性对象，而是存储到委托元素上去了，所以重新为提示框元素初始化
       self = $(e.currentTarget).data('bs.' + this.type)
       if (!self) {
         // getDelegateOptions: 重置CSS选择器，并将触发器设置为manual，并不设置监听，而是继续使用委托
         self = new this.constructor(e.currentTarget, this.getDelegateOptions())
         $(e.currentTarget).data('bs.' + this.type, self)
       }
     }
 
     if (e) {
       // 切换点击状态，判断是否存在显示条件，并执行相应操作
       self.inState.click = !self.inState.click
       if (self.isInStateTrue()) self.enter(self)
       else self.leave(self)
     } else {
       // 如果不存在参数e(JS调用toggle触发)，那么动态判断当前点击提示框是否显示，并执行相应操作
       self.tip().hasClass('in') ? self.leave(self) : self.enter(self)
     }
   }
 
   // 销毁tooltip实例
   Tooltip.prototype.destroy = function () {
     var that = this
     clearTimeout(this.timeout)
     this.hide(function () {
       that.$element.off('.' + that.type).removeData('bs.' + that.type)
       if (that.$tip) {
         that.$tip.detach()
       }
       that.$tip = null
       that.$arrow = null
       that.$viewport = null
       that.$element = null
     })
   }
 
   // TOOLTIP PLUGIN DEFINITION
   // =========================
   function Plugin(option) {
     return this.each(function () {
       var $this   = $(this)
       var data    = $this.data('bs.tooltip')
       var options = typeof option == 'object' && option
 
       // 如果没有tooltip初始化，那么直接指向destroy|hide方法不进行任何操作
       if (!data && /destroy|hide/.test(option)) return
       if (!data) $this.data('bs.tooltip', (data = new Tooltip(this, options)))
       if (typeof option == 'string') data[option]()
     })
   }
 
   var old = $.fn.tooltip
 
   $.fn.tooltip             = Plugin
   $.fn.tooltip.Constructor = Tooltip
 
   // TOOLTIP NO CONFLICT
   // ===================
   $.fn.tooltip.noConflict = function () {
     $.fn.tooltip = old
     return this
   }
 }(jQuery);
```
### 二、应用 & 源码分析
#### 1、应用
```
 这是一个 <a href="#" class="tooltip-test" data-toggle="tooltip" data-placement="left" title="左侧的 Tooltip">左侧的 Tooltip</a>.
 <button type="button" class="btn btn-default" data-toggle="tooltip" data-placement="left" title="左侧的 Tooltip">左侧的 Tooltip</button>
 // 不能直接通过自定义属性data-来触发，需依赖于JavaScript
 <script>
 	$(function () { $("[data-toggle='tooltip']").tooltip(); });
 </script>
```
#### 2、源码分析
##### 2.1、tooltip.less 
1、容器为绝对布局，默认 opacity 为 0 ，即隐藏   
2、提示框模板内容也很简单，只含有箭头和内容文本，箭头根据提示框出现方向进行调整显示  
##### 2.2、tooltip.js  
1、有多种触发方式，设置 inState = { click: false, hover: false, focus: false } 对象，切换对象中属性，只要对象中有一个属性为true那么显示提示框   
2、含 selector 选择器，如果存在selector，那么使用事件委托进行处理  
3、显示时，动态生成提示框内容并插入到文档中，然后再经过一系列的计算得到并设置提示框的 top、 left(个人觉得插入到触发元素中更简单快捷，但是语义化不符合要求)   
4、通过 while (document.getElementById(prefix)) 可以得到一个文档中没有的ID
5、需要注意如果连续触发而且存在动画的情况，需要判断处理显示隐藏问题，源码中通过hoverState做限制