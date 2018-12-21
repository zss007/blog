---
title: bootstrap 之 tab
categories:
- bootstrap
---
这节介绍下 tab(选项卡)模块的源码实现。
<!--more-->
### 一、源码
```
 /* ========================================================================
  * Bootstrap: tab.js v3.3.7(选项卡)
  * http://getbootstrap.com/javascript/#tabs
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * ======================================================================== */
 +function ($) {
   'use strict';
 
   // TAB CLASS DEFINITION
   // ====================
   var Tab = function (element) {
     // jscs:disable requireDollarBeforejQueryAssignment
     this.element = $(element)
     // jscs:enable requireDollarBeforejQueryAssignment
   }
 
   Tab.VERSION = '3.3.7'
   Tab.TRANSITION_DURATION = 150
 
   Tab.prototype.show = function () {
     var $this    = this.element
     // tab菜单可能存在二级菜单，'ul:not(.dropdown-menu)' => 找到外层的ul元素
     var $ul      = $this.closest('ul:not(.dropdown-menu)')
 
     // 读取data-target属性数据，如果不存在则读取href属性数据并格式化为CSS选择符
     var selector = $this.data('target')
     if (!selector) {
       selector = $this.attr('href')
       selector = selector && selector.replace(/.*(?=#[^\s]*$)/, '') // strip for ie7
     }
 
     // 如果已经被选中则直接返回
     if ($this.parent('li').hasClass('active')) return
 
     // 之前被选中的最后一个tab a触发hide事件，relatedTarget为将要显示的元素；将要显示的tab a触发show事件，relatedTarget为之前被选中的
     var $previous = $ul.find('.active:last a')
     var hideEvent = $.Event('hide.bs.tab', {
       relatedTarget: $this[0]
     })
     var showEvent = $.Event('show.bs.tab', {
       relatedTarget: $previous[0]
     })
 
     $previous.trigger(hideEvent)
     $this.trigger(showEvent)
 
     if (showEvent.isDefaultPrevented() || hideEvent.isDefaultPrevented()) return
 
     var $target = $(selector)
 
     this.activate($this.closest('li'), $ul)
     this.activate($target, $target.parent(), function () {
       $previous.trigger({
         type: 'hidden.bs.tab',
         relatedTarget: $this[0]
       })
       $this.trigger({
         type: 'shown.bs.tab',
         relatedTarget: $previous[0]
       })
     })
   }
 
   Tab.prototype.activate = function (element, container, callback) {
     var $active    = container.find('> .active')
     var transition = callback
       && $.support.transition
       && ($active.length && $active.hasClass('fade') || !!container.find('> .fade').length)
 
     function next() {
       // 如果是tab的下拉菜单的话，那么$active是dropdown的li元素
       $active
         .removeClass('active')
         .find('> .dropdown-menu > .active')
           .removeClass('active')
         .end()
         .find('[data-toggle="tab"]')
           .attr('aria-expanded', false)
 
       element
         .addClass('active')
         .find('[data-toggle="tab"]')
           .attr('aria-expanded', true)
 
       if (transition) {
         element[0].offsetWidth // reflow for transition
         element.addClass('in')
       } else {
         element.removeClass('fade')
       }
 
       if (element.parent('.dropdown-menu').length) {
         element
           .closest('li.dropdown')
             .addClass('active')
           .end()
           .find('[data-toggle="tab"]')
             .attr('aria-expanded', true)
       }
 
       callback && callback()
     }
 
     // 如果是菜单元素直接执行next，否则等$active移除in再执行next方法
     $active.length && transition ?
       $active
         .one('bsTransitionEnd', next)
         .emulateTransitionEnd(Tab.TRANSITION_DURATION) :
       next()
 
     $active.removeClass('in')
   }
 
   // TAB PLUGIN DEFINITION
   // =====================
   function Plugin(option) {
     return this.each(function () {
       var $this = $(this)
       var data  = $this.data('bs.tab')
 
       if (!data) $this.data('bs.tab', (data = new Tab(this)))
       if (typeof option == 'string') data[option]()
     })
   }
 
   var old = $.fn.tab
 
   $.fn.tab             = Plugin
   $.fn.tab.Constructor = Tab
 
   // TAB NO CONFLICT
   // ===============
   $.fn.tab.noConflict = function () {
     $.fn.tab = old
     return this
   }
 
   // TAB DATA-API
   // ============
   var clickHandler = function (e) {
     e.preventDefault()
     Plugin.call($(this), 'show')
   }
 
   $(document)
     .on('click.bs.tab.data-api', '[data-toggle="tab"]', clickHandler)
     .on('click.bs.tab.data-api', '[data-toggle="pill"]', clickHandler)
 }(jQuery);
```
### 二、component-animations.less 源码
```
 //
 // Component animations(组件动画)
 // --------------------------------------------------
 
 // Heads up!
 //
 // We don't use the `.opacity()` mixin here since it causes a bug with text
 // fields in IE7-8. Source: https://github.com/twbs/bootstrap/pull/3552.
 .fade {
   opacity: 0;
   .transition(opacity .15s linear);
   &.in {
     opacity: 1;
   }
 }
 
 // 默认折叠内容隐藏
 .collapse {
   display: none;
   &.in      { display: block; }
   tr&.in    { display: table-row; }
   tbody&.in { display: table-row-group; }
 }
 
 .collapsing {
   position: relative;
   height: 0;
   overflow: hidden;
   .transition-property(~"height, visibility");
   .transition-duration(.35s);
   .transition-timing-function(ease);
 }
```
### 三、应用 & 源码分析
#### 1、应用
```
 <hr>
 <p class="active-tab"><strong>激活的标签页</strong>：<span></span></p>
 <p class="previous-tab"><strong>前一个激活的标签页</strong>：<span></span></p>
 <hr>
 <ul id="myTab" class="nav nav-tabs">
   <li class="active">
     <a href="#home" data-toggle="tab">菜鸟教程</a>
   </li>
   <li><a href="#ios" data-toggle="tab">iOS</a></li>
   <li class="dropdown">
     <a href="#" id="myTabDrop1" class="dropdown-toggle"
        data-toggle="dropdown">Java<b class="caret"></b>
     </a>
     <ul class="dropdown-menu" role="menu" aria-labelledby="myTabDrop1">
       <li><a href="#jmeter" tabindex="-1" data-toggle="tab">jmeter</a></li>
       <li><a href="#ejb" tabindex="-1" data-toggle="tab">ejb</a></li>
     </ul>
   </li>
 </ul>
 <div id="myTabContent" class="tab-content">
   <div class="tab-pane fade in active" id="home">
     <p>菜鸟教程是一个提供最新的web技术站点，本站免费提供了建站相关的技术文档，帮助广大web技术爱好者快速入门并建立自己的网站。菜鸟先飞早入行——学的不仅是技术，更是梦想。</p>
   </div>
   <div class="tab-pane fade" id="ios">
     <p>iOS 是一个由苹果公司开发和发布的手机操作系统。最初是于 2007 年首次发布 iPhone、iPod Touch 和 Apple
       TV。iOS 派生自 OS X，它们共享 Darwin 基础。OS X 操作系统是用在苹果电脑上，iOS 是苹果的移动版本。</p>
   </div>
   <div class="tab-pane fade" id="jmeter">
     <p>jMeter 是一款开源的测试软件。它是 100% 纯 Java 应用程序，用于负载和性能测试。</p>
   </div>
   <div class="tab-pane fade" id="ejb">
     <p>Enterprise Java Beans（EJB）是一个创建高度可扩展性和强大企业级应用程序的开发架构，部署在兼容应用程序服务器（比如 JBOSS、Web Logic 等）的 J2EE 上。
     </p>
   </div>
 </div>
 <script>
   $(function () {
     $('#myTab a:last').tab('show');
 
     $('a[data-toggle="tab"]').on('shown.bs.tab', function (e) {
       // 获取已激活的标签页的名称
       var activeTab = $(e.target).text();
       // 获取前一个激活的标签页的名称
       var previousTab = $(e.relatedTarget).text();
       $(".active-tab span").html(activeTab);
       $(".previous-tab span").html(previousTab);
     });
   })
 </script>
```
#### 2、源码分析
1、show   
tab 插件可能内嵌下拉菜单插件 dropdown，所以需要判断是否是 tab 插件链接还是存在于 dropdown     
2、activate   
核心方法，为点击相应元素添加 active 类，并移除前 active 类元素；如果具有 fade 类(见component-animations.less源码)，则在过渡执行完成后再回调    