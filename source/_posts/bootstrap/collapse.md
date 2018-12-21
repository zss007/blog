---
title: bootstrap 之 collapse
categories:
- bootstrap
---
这节介绍下 collapse(手风琴)模块的源码实现。
<!--more-->
### 一、源码
```
 /* ========================================================================
  * Bootstrap: collapse.js v3.3.7(折叠/手风琴)
  * http://getbootstrap.com/javascript/#collapse
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * ======================================================================== */
 
 /* jshint latedef: false */
 
 +function ($) {
   'use strict';
 
   // COLLAPSE PUBLIC CLASS DEFINITION
   // ================================
   var Collapse = function (element, options) {
     this.$element      = $(element)
     this.options       = $.extend({}, Collapse.DEFAULTS, options)
     this.$trigger      = $('[data-toggle="collapse"][href="#' + element.id + '"],' +
                            '[data-toggle="collapse"][data-target="#' + element.id + '"]')
     // 变量：是否正在过渡动画中
     this.transitioning = null
 
     if (this.options.parent) {
       this.$parent = this.getParent()
     } else {
       this.addAriaAndCollapsedClass(this.$element, this.$trigger)
     }
 
     if (this.options.toggle) this.toggle()
   }
 
   Collapse.VERSION  = '3.3.7'
 
   Collapse.TRANSITION_DURATION = 350
 
   Collapse.DEFAULTS = {
     toggle: true
   }
 
   // 高度还是宽度发生改变
   Collapse.prototype.dimension = function () {
     var hasWidth = this.$element.hasClass('width')
     return hasWidth ? 'width' : 'height'
   }
 
   // 显示
   Collapse.prototype.show = function () {
     // 如果正在动画过渡中，或者已经显示，那么直接返回不做任何处理
     if (this.transitioning || this.$element.hasClass('in')) return
 
     var activesData
     // 展开或者正在展开的手风琴控制面板
     var actives = this.$parent && this.$parent.children('.panel').children('.in, .collapsing')
 
     // 如果手风琴控制面板正在过渡中，则不作任何处理(这里只考虑actives.length为1的处理，即控制面板只有单个展开的情况)
     if (actives && actives.length) {
       activesData = actives.data('bs.collapse')
       if (activesData && activesData.transitioning) return
     }
 
     var startEvent = $.Event('show.bs.collapse')
     this.$element.trigger(startEvent)
     if (startEvent.isDefaultPrevented()) return
 
     if (actives && actives.length) {
       // 将之前拓展开的控制面板隐藏起来
       Plugin.call(actives, 'hide')
       // 存在一种情况：如果一开始有控制面板展开，这时候点击其他面板，展开的面板会被触发隐藏同时初始化bs.collapse对象('hide'参数)存储到
       // data中，下次点击最开始展开的面板时，这时候会从data中读取bs.collapse作为对象而不是重新初始化，然后触发toggle，但是因为并没有将
       // parent传到option中，所以后面点击的面板并不会收拢；这里做的处理是将所有已经展开的面板都隐藏，而且清除掉存在data的bs.collapse数
       // 据，下次点击重新初始化
       // 不过其实将对象存在data的bs.collapse属性中还是有必要的，如果多次点击同一面板触发器，可以避免多次初始化对象
       activesData || actives.data('bs.collapse', null)
     }
 
     var dimension = this.dimension()
 
     // 目标元素移除collapse类，添加collapsing，设置height为0
     this.$element
       .removeClass('collapse')
       .addClass('collapsing')[dimension](0)
       .attr('aria-expanded', true)
 
     // 触发元素移除collapsed类
     this.$trigger
       .removeClass('collapsed')
       .attr('aria-expanded', true)
 
     this.transitioning = 1
 
     var complete = function () {
       // 目标元素移除collapsing类，添加.collapse .in类，并重置height
       this.$element
         .removeClass('collapsing')
         .addClass('collapse in')[dimension]('')
       this.transitioning = 0
       this.$element
         .trigger('shown.bs.collapse')
     }
 
     // 如果设备不支持动画，那么直接执行完成函数
     if (!$.support.transition) return complete.call(this)
 
     var scrollSize = $.camelCase(['scroll', dimension].join('-'))
 
     // 设置函数动画结束监听函数，并设置height为目标元素的scrollHeight
     this.$element
       .one('bsTransitionEnd', $.proxy(complete, this))
       .emulateTransitionEnd(Collapse.TRANSITION_DURATION)[dimension](this.$element[0][scrollSize])
   }
 
   // 隐藏
   Collapse.prototype.hide = function () {
     if (this.transitioning || !this.$element.hasClass('in')) return
 
     var startEvent = $.Event('hide.bs.collapse')
     this.$element.trigger(startEvent)
     if (startEvent.isDefaultPrevented()) return
 
     var dimension = this.dimension()
 
     // 强制回流，为页面重绘做准备
     this.$element[dimension](this.$element[dimension]())[0].offsetHeight
 
     this.$element
       .addClass('collapsing')
       .removeClass('collapse in')
       .attr('aria-expanded', false)
 
     this.$trigger
       .addClass('collapsed')
       .attr('aria-expanded', false)
 
     this.transitioning = 1
 
     var complete = function () {
       this.transitioning = 0
       this.$element
         .removeClass('collapsing')
         .addClass('collapse')
         .trigger('hidden.bs.collapse')
     }
 
     if (!$.support.transition) return complete.call(this)
 
     this.$element
       [dimension](0)
       .one('bsTransitionEnd', $.proxy(complete, this))
       .emulateTransitionEnd(Collapse.TRANSITION_DURATION)
   }
 
   // 切换手风琴显示隐藏
   Collapse.prototype.toggle = function () {
     this[this.$element.hasClass('in') ? 'hide' : 'show']()
   }
 
   // 找到手风琴目标元素并为所有拓展面板执行addAriaAndCollapsedClass方法
   Collapse.prototype.getParent = function () {
     return $(this.options.parent)
       .find('[data-toggle="collapse"][data-parent="' + this.options.parent + '"]')
       .each($.proxy(function (i, element) {
         var $element = $(element)
         this.addAriaAndCollapsedClass(getTargetFromTrigger($element), $element)
       }, this))
       .end()
   }
 
   // 添加无障碍阅读属性和collapsed类
   Collapse.prototype.addAriaAndCollapsedClass = function ($element, $trigger) {
     var isOpen = $element.hasClass('in')
 
     $element.attr('aria-expanded', isOpen)
     $trigger
       .toggleClass('collapsed', !isOpen)
       .attr('aria-expanded', isOpen)
   }
 
   // 获取目标元素
   function getTargetFromTrigger($trigger) {
     var href
     var target = $trigger.attr('data-target')
       || (href = $trigger.attr('href')) && href.replace(/.*(?=#[^\s]+$)/, '') // strip for ie7
 
     return $(target)
   }
 
   // COLLAPSE PLUGIN DEFINITION
   // ==========================
   function Plugin(option) {
     return this.each(function () {
       var $this   = $(this)
       var data    = $this.data('bs.collapse')
       var options = $.extend({}, Collapse.DEFAULTS, $this.data(), typeof option == 'object' && option)
 
       // 如果直接用JS触发手风琴，而且option为show/hide字符串，那么没有不调用toggle方法，而直接调用相应hide/show方法
       if (!data && options.toggle && /show|hide/.test(option)) options.toggle = false
       if (!data) $this.data('bs.collapse', (data = new Collapse(this, options)))
       if (typeof option == 'string') data[option]()
     })
   }
 
   var old = $.fn.collapse
 
   $.fn.collapse             = Plugin
   $.fn.collapse.Constructor = Collapse
 
 
   // COLLAPSE NO CONFLICT
   // ====================
   $.fn.collapse.noConflict = function () {
     $.fn.collapse = old
     return this
   }
 
 
   // COLLAPSE DATA-API
   // =================
   $(document).on('click.bs.collapse.data-api', '[data-toggle="collapse"]', function (e) {
     var $this   = $(this)
 
     if (!$this.attr('data-target')) e.preventDefault()
 
     var $target = getTargetFromTrigger($this)
     // 如果已经初始化了bs.collapse属性对应的对象，那么直接触发toggle方法(即除了第一次点击初始化之外)
     var data    = $target.data('bs.collapse')
     var option  = data ? 'toggle' : $this.data()
 
     Plugin.call($target, option)
   })
 
 }(jQuery);
```
### 二、应用 & 源码分析
#### 1、应用
```
 <div class="panel-group" id="accordion">
   <div class="panel panel-default">
     <div class="panel-heading">
       <h4 class="panel-title">
         <a data-toggle="collapse" data-parent="#accordion"
            href="#collapseOne">
           点击我进行展开，再次点击我进行折叠。第 1 部分
         </a>
       </h4>
     </div>
     <div id="collapseOne" class="panel-collapse collapse in">
       <div class="panel-body">
         Nihil anim keffiyeh helvetica, craft beer labore wes anderson
         cred nesciunt sapiente ea proident. Ad vegan excepteur butcher
         vice lomo.
       </div>
     </div>
   </div>
   <div class="panel panel-default">
     <div class="panel-heading">
       <h4 class="panel-title">
         <a data-toggle="collapse" data-parent="#accordion"
            href="#collapseTwo">
           点击我进行展开，再次点击我进行折叠。第 2 部分
         </a>
       </h4>
     </div>
     <div id="collapseTwo" class="panel-collapse collapse">
       <div class="panel-body">
         Nihil anim keffiyeh helvetica, craft beer labore wes anderson
         cred nesciunt sapiente ea proident. Ad vegan excepteur butcher
         vice lomo.
       </div>
     </div>
   </div>
   <div class="panel panel-default">
     <div class="panel-heading">
       <h4 class="panel-title">
         <a data-toggle="collapse" data-parent="#accordion"
            href="#collapseThree">
           点击我进行展开，再次点击我进行折叠。第 3 部分
         </a>
       </h4>
     </div>
     <div id="collapseThree" class="panel-collapse collapse">
       <div class="panel-body">
         Nihil anim keffiyeh helvetica, craft beer labore wes anderson
         cred nesciunt sapiente ea proident. Ad vegan excepteur butcher
         vice lomo.
       </div>
     </div>
   </div>
 </div>
 
 <script>
   $(function () { $('#collapseFour').collapse({
     toggle: false
   })});
   $(function () { $('#collapseTwo').collapse('show')});
   $(function () { $('#collapseThree').collapse('toggle')});
   $(function () { $('#collapseOne').collapse('hide')});
 </script>
```
#### 2、源码分析
1、data-parent   
如果元素具有相同的 data-parent 属性，那么可以具有点击一个折叠面板，其他所有展开的面板均折叠     
2、show   
开始时每个面板均添加 collapse 类(display:none)，动画准备时移除 collapse 并添加 collapsing 类(height:0;overflow:hidden;)，动画执行时 collapsing 移除添加 .collapse.in(display: block;) 并清空行内 height 达到过渡打开面板的效果    