---
title: bootstrap 之 scrollspy
categories:
- bootstrap
---
这节介绍下 scrollspy(滚动侦测)模块的源码实现。
<!--more-->
### 一、源码
```
 /* ========================================================================
  * Bootstrap: scrollspy.js v3.3.7(滚动侦测)
  * http://getbootstrap.com/javascript/#scrollspy
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * ======================================================================== */
 +function ($) {
   'use strict';
 
   // SCROLLSPY CLASS DEFINITION(构造函数，初始化要监听的滚动对象、nav区(selector)，执行滚动事件绑定)
   // ==========================
   function ScrollSpy(element, options) {
     this.$body          = $(document.body)
     this.$scrollElement = $(element).is(document.body) ? $(window) : $(element)
     this.options        = $.extend({}, ScrollSpy.DEFAULTS, options)
     this.selector       = (this.options.target || '') + ' .nav li > a'
     this.offsets        = []
     this.targets        = []
     this.activeTarget   = null
     this.scrollHeight   = 0
 
     this.$scrollElement.on('scroll.bs.scrollspy', $.proxy(this.process, this))
     this.refresh()
     this.process()
   }
 
   ScrollSpy.VERSION  = '3.3.7'
   ScrollSpy.DEFAULTS = {
     offset: 10
   }
 
   // 获取指定元素的scrollHeight(内容高度)，或兼容获取body的内容高度
   ScrollSpy.prototype.getScrollHeight = function () {
     // scrollHeigth是元素内容的高度，包括overflow导致不可见的部分，this.$body[0].scrollHeight和document.body.scrollHeight其实是一样的
     // 在DTD声明和未声明时，document.documentElement.scrollHeight和document.body.scrollHeight有一个会为可视窗口高度，所以用Math.max取得全部内容高度
     return this.$scrollElement[0].scrollHeight || Math.max(this.$body[0].scrollHeight, document.documentElement.scrollHeight)
   }
 
   // 获得content区中对应的锚点（放到targets中）和对应的高度（放到offsets中），及获取滚动容器的内容高度
   ScrollSpy.prototype.refresh = function () {
     var that          = this
     var offsetMethod  = 'offset'
     // offsetBase为滚动容器data-spy="scroll"相对于滚动条顶部的偏移
     var offsetBase    = 0
 
     this.offsets      = []
     this.targets      = []
     this.scrollHeight = this.getScrollHeight()
 
     // 如果监听的滚动对象不是body，则使用position方法来获取offsets值；
     // jquery的offset()方法是获取匹配元素在当前视口的相对偏移，position()方法是获取匹配元素相对父元素的偏移
     if (!$.isWindow(this.$scrollElement[0])) {
       offsetMethod = 'position'
       offsetBase   = this.$scrollElement.scrollTop()
     }
 
     // 找到全部的锚点，返回由[offsets，锚点]组成的数组
     // jquery的map有点奇怪，return值或return[值]得到的都是数组。return [[值]]得到的才是数组组成的数组
     this.$body
       .find(this.selector)
       .map(function () {
         var $el   = $(this)
         // 获取符合格式的锚点$href
         var href  = $el.data('target') || $el.attr('href')
         var $href = /^#./.test(href) && $(href)
 
         // $href[offsetMethod]().top => 相对于父元素(即滚动容器data-spy="scroll")的偏移量
         return ($href
           && $href.length
           && $href.is(':visible')
           && [[$href[offsetMethod]().top + offsetBase, href]]) || null
       })
       .sort(function (a, b) { return a[0] - b[0] })
       .each(function () {
         that.offsets.push(this[0])
         that.targets.push(this[1])
       })
   }
 
   // 根据this.offsets与当前的scrollTop比较，判断是否需要activate
   ScrollSpy.prototype.process = function () {
     // 加上规定offset的，距离顶部的值(this.options.offset：当计算滚动位置时，距离顶部的偏移像素)
     var scrollTop    = this.$scrollElement.scrollTop() + this.options.offset
     // 当前的内容高度
     var scrollHeight = this.getScrollHeight()
     // offset值+内容高度-可视高度得到的最大可滚动高度
     var maxScroll    = this.options.offset + scrollHeight - this.$scrollElement.height()
     var offsets      = this.offsets
     var targets      = this.targets
     var activeTarget = this.activeTarget
     var i
 
     // 当内容高度发生动态变化时调用refresh方法
     if (this.scrollHeight != scrollHeight) {
       this.refresh()
     }
 
     // 超过或等于当前元素的最大可滚动高度，说明滚动到了最底部，直接激活最后一个nav
     if (scrollTop >= maxScroll) {
       return activeTarget != (i = targets[targets.length - 1]) && this.activate(i)
     }
 
     // 没超过第一个offset，清除当前的激活对象（当this.options.offset为负数时存在这种情况）
     if (activeTarget && scrollTop < offsets[0]) {
       this.activeTarget = null
       return this.clear()
     }
 
     // 最精彩的部分，循环判断是否需要激活
     for (i = offsets.length; i--;) {
       activeTarget != targets[i]     // 满足当前遍历的target不是激活对象
         && scrollTop >= offsets[i]   // 满足当前滚动高度大于对应的offset
         && (offsets[i + 1] === undefined || scrollTop < offsets[i + 1])  // 满足当前滚动高度小于下一个滚动高度，或下一个滚动高度未定义
         && this.activate(targets[i]) // 激活该nav
     }
   }
 
   // 激活传进来的dom对象（即为其添加active类）
   ScrollSpy.prototype.activate = function (target) {
     this.activeTarget = target   // 先把该对象存入实例对象中
 
     this.clear()   // 清除当前的激活对象
 
     var selector = this.selector +
       '[data-target="' + target + '"],' +
       this.selector + '[href="' + target + '"]'
 
     // 为对应的a标签的父元素li添加active类
     var active = $(selector)
       .parents('li')
       .addClass('active')
 
     // 为下拉菜单的相应li元素添加active类
     if (active.parent('.dropdown-menu').length) {
       active = active
         .closest('li.dropdown')
         .addClass('active')
     }
 
     // 触发自定义事件
     active.trigger('activate.bs.scrollspy')
   }
 
   // 清除当前的nav中的激活对象
   ScrollSpy.prototype.clear = function () {
     $(this.selector)
       .parentsUntil(this.options.target, '.active')
       .removeClass('active')
   }
 
   // SCROLLSPY PLUGIN DEFINITION
   // ===========================
   function Plugin(option) {
     return this.each(function () {
       var $this   = $(this)
       var data    = $this.data('bs.scrollspy')
       var options = typeof option == 'object' && option
 
       if (!data) $this.data('bs.scrollspy', (data = new ScrollSpy(this, options)))
       if (typeof option == 'string') data[option]()
     })
   }
 
   var old = $.fn.scrollspy
 
   $.fn.scrollspy             = Plugin
   $.fn.scrollspy.Constructor = ScrollSpy
 
   // SCROLLSPY NO CONFLICT
   // =====================
   $.fn.scrollspy.noConflict = function () {
     $.fn.scrollspy = old
     return this
   }
 
   // SCROLLSPY DATA-API
   // ==================
   $(window).on('load.bs.scrollspy.data-api', function () {
     $('[data-spy="scroll"]').each(function () {
       var $spy = $(this)
       Plugin.call($spy, $spy.data())
     })
   })
 }(jQuery);
```
### 二、应用 & 源码分析
#### 1、应用
```
 <nav id="navbar-example" class="navbar navbar-default navbar-static" role="navigation">
   <div class="container-fluid">
     <div class="navbar-header">
       <button class="navbar-toggle" type="button" data-toggle="collapse"
               data-target=".bs-js-navbar-scrollspy">
         <span class="icon-bar"></span><span class="icon-bar"></span><span class="icon-bar"></span>
       </button>
       <a class="navbar-brand" href="#">教程名称</a>
     </div>
     <div class="collapse navbar-collapse bs-js-navbar-scrollspy">
       <ul class="nav navbar-nav">
         <li><a href="#ios">iOS</a></li>
         <li><a href="#svn">SVN</a></li>
         <li class="dropdown">
           <a href="#" id="navbarDrop1" class="dropdown-toggle"
              data-toggle="dropdown">Java
             <b class="caret"></b>
           </a>
           <ul class="dropdown-menu" role="menu"
               aria-labelledby="navbarDrop1">
             <li><a href="#jmeter" tabindex="-1">jmeter</a></li>
             <li><a href="#ejb" tabindex="-1">ejb</a></li>
             <li class="divider"></li>
             <li><a href="#spring" tabindex="-1">spring</a></li>
           </ul>
         </li>
       </ul>
     </div>
   </div>
 </nav>
 <div data-spy="scroll" data-target="#navbar-example" data-offset="-10"
      style="height:200px;overflow:auto; position: relative;">
   <h4 id="ios">iOS</h4>
   <p>iOS 是一个由苹果公司开发和发布的手机操作系统。最初是于 2007 年首次发布 iPhone、iPod Touch 和 Apple
     TV。iOS 派生自 OS X，它们共享 Darwin 基础。OS X 操作系统是用在苹果电脑上，iOS 是苹果的移动版本。
   </p>
   <h4 id="svn">SVN</h4>
   <p>Apache Subversion，通常缩写为 SVN，是一款开源的版本控制系统软件。Subversion 由 CollabNet 公司在 2000 年创建。但是现在它已经发展为 Apache Software
     Foundation 的一个项目，因此拥有丰富的开发人员和用户社区。
   </p>
   <h4 id="jmeter">jMeter</h4>
   <p>jMeter 是一款开源的测试软件。它是 100% 纯 Java 应用程序，用于负载和性能测试。
   </p>
   <h4 id="ejb">EJB</h4>
   <p>Enterprise Java Beans（EJB）是一个创建高度可扩展性和强大企业级应用程序的开发架构，部署在兼容应用程序服务器（比如 JBOSS、Web Logic 等）的 J2EE 上。
   </p>
   <h4 id="spring">Spring</h4>
   <p>Spring 框架是一个开源的 Java 平台，为快速开发功能强大的 Java 应用程序提供了完备的基础设施支持。
   </p>
   <p>Spring 框架最初是由 Rod Johnson 编写的，在 2003 年 6 月首次发布于 Apache 2.0 许可证下。
   </p>
 </div>
```
#### 2、源码分析
1、refresh   
获取导航元素 nav 内各链接对应元素的相对高度(如果不是 body，则相对于父元素)，然后存储各锚点和相对高度     
2、process   
遍历锚点，判断锚点对应元素的相对高度是否位于锚点之间，然后'激活'对应锚点  