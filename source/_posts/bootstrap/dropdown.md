---
title: bootstrap 之 dropdown
categories:
- bootstrap
---
这节介绍下 dropdown(下拉菜单)模块的源码实现。
<!--more-->
### 一、源码
#### 1、dropdowns.less
```
 //
 // Dropdown menus(下拉菜单)
 // --------------------------------------------------
 
 // Dropdown arrow/caret(下拉菜单箭头)
 .caret {
   display: inline-block;
   width: 0;
   height: 0;
   margin-left: 2px;
   vertical-align: middle;
   border-top:   @caret-width-base dashed;
   border-top:   @caret-width-base solid ~"\9"; // IE8
   border-right: @caret-width-base solid transparent;
   border-left:  @caret-width-base solid transparent;
 }
 
 // The dropdown wrapper (div: 下拉菜单容器)
 .dropup,
 .dropdown {
   position: relative;
 }
 
 // Prevent the focus on the dropdown toggle when closing dropdowns
 .dropdown-toggle:focus {
   outline: 0;
 }
 
 // The dropdown menu (ul: 下拉菜单)
 .dropdown-menu {
   position: absolute;
   top: 100%;
   left: 0;
   z-index: @zindex-dropdown;
   display: none; // none by default, but block on "open" of the menu
   float: left;
   min-width: 160px;
   padding: 5px 0;
   margin: 2px 0 0; // override default ul
   list-style: none;
   font-size: @font-size-base;
   text-align: left; // Ensures proper alignment if parent has it changed (e.g., modal footer)
   background-color: @dropdown-bg;
   border: 1px solid @dropdown-fallback-border; // IE8 fallback
   border: 1px solid @dropdown-border;
   border-radius: @border-radius-base;
   .box-shadow(0 6px 12px rgba(0,0,0,.175));
   background-clip: padding-box;
 
   // Aligns the dropdown menu to right(下拉菜单右对齐)
   //
   // Deprecated as of 3.1.0 in favor of `.dropdown-menu-[dir]`
   &.pull-right {
     right: 0;
     left: auto;
   }
 
   // Dividers (basically an hr) within the dropdown(下拉分割线)
   .divider {
     .nav-divider(@dropdown-divider-bg);
   }
 
   // Links within the dropdown menu(下拉菜单链接)
   > li > a {
     display: block;
     padding: 3px 20px;
     clear: both;
     font-weight: normal;
     line-height: @line-height-base;
     color: @dropdown-link-color;
     white-space: nowrap; // prevent links from randomly breaking onto new lines
   }
 }
 
 // Hover/Focus state(菜单项状态: 悬浮状态、焦点状态)
 .dropdown-menu > li > a {
   &:hover,
   &:focus {
     text-decoration: none;
     color: @dropdown-link-hover-color;
     background-color: @dropdown-link-hover-bg;
   }
 }
 
 // Active state(当前状态)
 .dropdown-menu > .active > a {
   &,
   &:hover,
   &:focus {
     color: @dropdown-link-active-color;
     text-decoration: none;
     outline: 0;
     background-color: @dropdown-link-active-bg;
   }
 }
 
 // Disabled state(禁用状态)
 //
 // Gray out text and ensure the hover/focus state remains gray
 .dropdown-menu > .disabled > a {
   &,
   &:hover,
   &:focus {
     color: @dropdown-link-disabled-color;
   }
 
   // Nuke hover/focus effects
   &:hover,
   &:focus {
     text-decoration: none;
     background-color: transparent;
     background-image: none; // Remove CSS gradient
     .reset-filter();
     cursor: @cursor-disabled;
   }
 }
 
 // Open state for the dropdown(展开下拉菜单)
 .open {
   // Show the menu
   > .dropdown-menu {
     display: block;
   }
 
   // Remove the outline when :focus is triggered
   > a {
     outline: 0;
   }
 }
 
 // Menu positioning(下拉菜单对齐方式)
 //
 // Add extra class to `.dropdown-menu` to flip the alignment of the dropdown
 // menu with the parent.(右对齐)
 .dropdown-menu-right {
   left: auto; // Reset the default from `.dropdown-menu`
   right: 0;
 }
 // With v3, we enabled auto-flipping if you have a dropdown within a right
 // aligned nav component. To enable the undoing of that, we provide an override
 // to restore the default dropdown menu alignment.
 //
 // This is only for left-aligning a dropdown menu within a `.navbar-right` or
 // `.pull-right` nav component.(左对齐)
 .dropdown-menu-left {
   left: 0;
   right: auto;
 }
 
 // Dropdown section headers(下拉菜单标题)
 .dropdown-header {
   display: block;
   padding: 3px 20px;
   font-size: @font-size-small;
   line-height: @line-height-base;
   color: @dropdown-header-color;
   white-space: nowrap; // as with > li > a
 }
 
 // Backdrop to catch body clicks on mobile, etc.(背景遮罩层)
 .dropdown-backdrop {
   position: fixed;
   left: 0;
   right: 0;
   bottom: 0;
   top: 0;
   z-index: (@zindex-dropdown - 10);
 }
 
 // Right aligned dropdowns(右对齐下拉菜单)
 .pull-right > .dropdown-menu {
   right: 0;
   left: auto;
 }
 
 // Allow for dropdowns to go bottom up (aka, dropup-menu)
 //
 // Just add .dropup after the standard .dropdown class and you're set, bro.
 // TODO: abstract this so that the navbar fixed styles are not placed here?
 
 .dropup,
 .navbar-fixed-bottom .dropdown {
   // Reverse the caret
   .caret {
     border-top: 0;
     border-bottom: @caret-width-base dashed;
     border-bottom: @caret-width-base solid ~"\9"; // IE8
     content: "";
   }
   // Different positioning for bottom up menu
   .dropdown-menu {
     top: auto;
     bottom: 100%;
     margin-bottom: 2px;
   }
 }
 
 
 // Component alignment
 //
 // Reiterate per navbar.less and the modified component alignment there.
 
 @media (min-width: @grid-float-breakpoint) {
   .navbar-right {
     .dropdown-menu {
       .dropdown-menu-right();
     }
     // Necessary for overrides of the default right aligned menu.
     // Will remove come v4 in all likelihood.
     .dropdown-menu-left {
       .dropdown-menu-left();
     }
   }
 }
```
#### 2、dropdown.js
```
 /* ========================================================================
  * Bootstrap: dropdown.js v3.3.7(下拉菜单)
  * http://getbootstrap.com/javascript/#dropdowns
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * ======================================================================== */
 
 
 +function ($) {
   'use strict';
 
   // DROPDOWN CLASS DEFINITION
   // =========================
   var backdrop = '.dropdown-backdrop'
   var toggle   = '[data-toggle="dropdown"]'
   var Dropdown = function (element) {
     $(element).on('click.bs.dropdown', this.toggle)
   }
 
   Dropdown.VERSION = '3.3.7'
 
   // 获取触发按钮或链接的目标元素
   function getParent($this) {
     var selector = $this.attr('data-target')
 
     // 如果不存在data-target，那么获取href属性并将格式处理为CSS选择器
     if (!selector) {
       selector = $this.attr('href')
       selector = selector && /#[A-Za-z]/.test(selector) && selector.replace(/.*(?=#[^\s]*$)/, '') // strip for ie7
     }
 
     // 如果存在目标元素，那么直接选中
     var $parent = selector && $(selector)
 
     // 否则默认使用触发元素的父元素
     return $parent && $parent.length ? $parent : $this.parent()
   }
 
   // 隐藏下拉菜单
   function clearMenus(e) {
     // 如果鼠标右键点击不做任何处理
     if (e && e.which === 3) return
     // 移除背景
     $(backdrop).remove()
     $(toggle).each(function () {
       var $this         = $(this)
       var $parent       = getParent($this)
       var relatedTarget = { relatedTarget: this }
 
       // 如果不是下拉菜单已经展开菜单了，那么不做任何处理
       if (!$parent.hasClass('open')) return
 
       // 如果事件发生在目标元素中的input/textarea元素中，那么不做任何处理
       if (e && e.type == 'click' && /input|textarea/i.test(e.target.tagName) && $.contains($parent[0], e.target)) return
 
       // 触发开始隐藏菜单事件
       $parent.trigger(e = $.Event('hide.bs.dropdown', relatedTarget))
 
       if (e.isDefaultPrevented()) return
 
       $this.attr('aria-expanded', 'false')
       // 触发已经隐藏菜单事件
       $parent.removeClass('open').trigger($.Event('hidden.bs.dropdown', relatedTarget))
     })
   }
 
   // 切换下拉菜单显示状态
   Dropdown.prototype.toggle = function (e) {
     var $this = $(this)
 
     if ($this.is('.disabled, :disabled')) return
 
     var $parent  = getParent($this)
     var isActive = $parent.hasClass('open')
 
     clearMenus()
 
     // 显示下拉菜单
     if (!isActive) {
       // 如果是在移动设备上，那么点击后添加遮罩层
       if ('ontouchstart' in document.documentElement && !$parent.closest('.navbar-nav').length) {
         // if mobile we use a backdrop because click events don't delegate
         $(document.createElement('div'))
           .addClass('dropdown-backdrop')
           .insertAfter($(this))
           .on('click', clearMenus)
       }
 
       var relatedTarget = { relatedTarget: this }
       $parent.trigger(e = $.Event('show.bs.dropdown', relatedTarget))
 
       if (e.isDefaultPrevented()) return
 
       $this
         .trigger('focus')
         .attr('aria-expanded', 'true')
 
       $parent
         .toggleClass('open')
         .trigger($.Event('shown.bs.dropdown', relatedTarget))
     }
 
     // 返回false，阻止冒泡和默认事件发生；如果是用JS直接触发，而且JS绑定的元素是document的子元素就可以避免绑定触发多次的问题，因为这里返回了false
     return false
   }
 
   // 键盘事件处理函数
   Dropdown.prototype.keydown = function (e) {
     // 38:Up Arrow; 40:Down Arrow; 27:Esc; 32:Spacebar
     if (!/(38|40|27|32)/.test(e.which) || /input|textarea/i.test(e.target.tagName)) return
 
     var $this = $(this)
 
     e.preventDefault()
     e.stopPropagation()
 
     if ($this.is('.disabled, :disabled')) return
 
     var $parent  = getParent($this)
     var isActive = $parent.hasClass('open')
 
     if (!isActive && e.which != 27 || isActive && e.which == 27) {
       // 如果菜单处于展开状态而且点击esc键盘按钮，那么让触发元素获取焦点
       if (e.which == 27) $parent.find(toggle).trigger('focus')
       // 触发元素上触发点击事件
       return $this.trigger('click')
     }
 
     var desc = ' li:not(.disabled):visible a'
     var $items = $parent.find('.dropdown-menu' + desc)
 
     // 如果$items.length == false，说明拓展菜单隐藏，不执行下面操作；否则执行下拉菜单选择，选中菜单触发focus事件
     if (!$items.length) return
 
     var index = $items.index(e.target)
 
     if (e.which == 38 && index > 0)                 index--         // up
     if (e.which == 40 && index < $items.length - 1) index++         // down
     if (!~index)                                    index = 0
 
     $items.eq(index).trigger('focus')
   }
 
 
   // DROPDOWN PLUGIN DEFINITION
   // ==========================
   function Plugin(option) {
     return this.each(function () {
       var $this = $(this)
       var data  = $this.data('bs.dropdown')
 
       if (!data) $this.data('bs.dropdown', (data = new Dropdown(this)))
       if (typeof option == 'string') data[option].call($this)
     })
   }
 
   var old = $.fn.dropdown
 
   $.fn.dropdown             = Plugin
   $.fn.dropdown.Constructor = Dropdown
 
 
   // DROPDOWN NO CONFLICT
   // ====================
   $.fn.dropdown.noConflict = function () {
     $.fn.dropdown = old
     return this
   }
 
 
   // APPLY TO STANDARD DROPDOWN ELEMENTS
   // ===================================
   $(document)
     // 如果点击发生在document上，那么隐藏拓展开的菜单
     .on('click.bs.dropdown.data-api', clearMenus)
     // 如果点击事件发生在下拉菜单元素上，那么阻止冒泡到document上
     .on('click.bs.dropdown.data-api', '.dropdown form', function (e) { e.stopPropagation() })
     // 如果点击在触发元素上，触发Dropdown.prototype.toggle事件
     .on('click.bs.dropdown.data-api', toggle, Dropdown.prototype.toggle)
     // 在触发元素上设置键盘监听函数
     .on('keydown.bs.dropdown.data-api', toggle, Dropdown.prototype.keydown)
     // 如果是展开状态(只有展开状态才有.dropdown-menu)，那么则在展开菜单上也设置监听函数
     .on('keydown.bs.dropdown.data-api', '.dropdown-menu', Dropdown.prototype.keydown)
 }(jQuery);
```
### 二、应用 & 源码分析
#### 1、应用
```
 <nav class="navbar navbar-default" role="navigation">
     <div class="container-fluid"> 
     <div class="navbar-header">
         <a class="navbar-brand" href="#">菜鸟教程</a>
     </div>
     <div>
         <ul class="nav navbar-nav">
             <li class="active"><a href="#">iOS</a></li>
             <li><a href="#">SVN</a></li>
             <li class="dropdown">
                 <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                     Java 
                     <b class="caret"></b>
                 </a>
                 <ul class="dropdown-menu pull-right">
                     <li role="presentation" class="dropdown-header">下拉菜单标题</li>
                     <li><a href="#">jmeter</a></li>
                     <li><a href="#">EJB</a></li>
                     <li><a href="#">Jasper Report</a></li>
                     <li class="divider"></li>
                     <li><a href="#">分离的链接</a></li>
                     <li class="divider"></li>
                     <li><a href="#">另一个分离的链接</a></li>
                 </ul>
             </li>
         </ul>
     </div>
     </div>
 </nav>
```
#### 2、源码分析
##### 2.1、dropdowns.less 
1、可通过向 .dropdown-menu 添加 class .pull-right 或 .dropdown-menu-right 来向右对齐下拉菜单，默认左对齐   
2、可使用 class dropdown-header 向下拉菜单的标签区域添加标题  
3、可使用 class disabled 向下拉菜单的标签区域设置禁用项  
4、可使用 class divider 向下拉菜单的标签区域设置分割线 
##### 2.2、dropdown.js  
1、如果点击事件发生在 document ，那么执行 clearMenus 隐藏下拉框   
2、监听属性 data-toggle="dropdown" 的点击事件，触发下拉菜单显示隐藏切换，并返回false，阻止事件冒泡到 document 上  
3、在 data-toggle="dropdown" 和 class dropdown-menu 元素上监听键盘事件，执行相应操作