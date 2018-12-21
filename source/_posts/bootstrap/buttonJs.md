---
title: bootstrap 之 buttonJs
categories:
- bootstrap
---
上节介绍了 button 模块的样式，这节介绍 button.js。
<!--more-->
### 一、源码
```
 /* ========================================================================
  * Bootstrap: button.js v3.3.7(按钮)
  * http://getbootstrap.com/javascript/#buttons
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * ======================================================================== */
 +function ($) {
   'use strict';
 
   // BUTTON PUBLIC CLASS DEFINITION
   // ==============================
   var Button = function (element, options) {
     this.$element  = $(element);
     this.options   = $.extend({}, Button.DEFAULTS, options);
     this.isLoading = false
   };
 
   Button.VERSION  = '3.3.7';
 
   Button.DEFAULTS = {
     loadingText: 'loading...'
   };
 
   // 处理按钮状态
   Button.prototype.setState = function (state) {
     var d    = 'disabled';
     var $el  = this.$element;
     var val  = $el.is('input') ? 'val' : 'html';
     var data = $el.data();
 
     state += 'Text';
 
     // 存储按钮初始值，为了以后重置
     if (data.resetText == null) $el.data('resetText', $el[val]());
 
     // push to event loop to allow forms to submit
     setTimeout($.proxy(function () {
       $el[val](data[state] == null ? this.options[state] : data[state]);
 
       if (state == 'loadingText') {
         this.isLoading = true;
         $el.addClass(d).attr(d, d).prop(d, true)
       } else if (this.isLoading) {
         this.isLoading = false;
         $el.removeClass(d).removeAttr(d).prop(d, false)
       }
     }, this), 0)
   };
 
   // 处理单选、多选
   Button.prototype.toggle = function () {
     var changed = true;
     var $parent = this.$element.closest('[data-toggle="buttons"]');
 
     if ($parent.length) {
       var $input = this.$element.find('input');
       if ($input.prop('type') == 'radio') {
         if ($input.prop('checked')) changed = false;
         $parent.find('.active').removeClass('active');
         this.$element.addClass('active')
       } else if ($input.prop('type') == 'checkbox') {
         // 处理checkbox默认被checked情况，这个时候再被选中只是改变下样式
         if (($input.prop('checked')) !== this.$element.hasClass('active')) changed = false;
         this.$element.toggleClass('active')
       }
       $input.prop('checked', this.$element.hasClass('active'));
       if (changed) $input.trigger('change')
     } else {
       this.$element.attr('aria-pressed', !this.$element.hasClass('active'));
       this.$element.toggleClass('active')
     }
   };
 
   // BUTTON PLUGIN DEFINITION
   // ========================
   function Plugin(option) {
     return this.each(function () {
       var $this   = $(this);
       var data    = $this.data('bs.button');
       var options = typeof option == 'object' && option;
 
       if (!data) $this.data('bs.button', (data = new Button(this, options)));
 
       if (option == 'toggle') data.toggle();
       else if (option) data.setState(option)
     })
   }
 
   var old = $.fn.button;
 
   $.fn.button             = Plugin;
   $.fn.button.Constructor = Button;
 
   // BUTTON NO CONFLICT
   // ==================
   $.fn.button.noConflict = function () {
     $.fn.button = old;
     return this
   };
 
   // BUTTON DATA-API
   // ===============
   $(document)
     .on('click.bs.button.data-api', '[data-toggle^="button"]', function (e) {
       var $btn = $(e.target).closest('.btn');
       Plugin.call($btn, 'toggle');
       if (!($(e.target).is('input[type="radio"], input[type="checkbox"]'))) {
         // Prevent double click on radios, and the double selections (so cancellation) on checkboxes
         e.preventDefault();
         // The target component still receive the focus
         if ($btn.is('input,button')) $btn.trigger('focus');
         else $btn.find('input:visible,button:visible').first().trigger('focus')
       }
     })
     .on('focus.bs.button.data-api blur.bs.button.data-api', '[data-toggle^="button"]', function (e) {
       $(e.target).closest('.btn').toggleClass('focus', /^focus(in)?$/.test(e.type))
     })
 
 }(jQuery);
```
### 二、应用 & 源码分析
#### 1、应用
```
 // 切换状态
 $("#loaddingBtn").click(function () {
   $(this).button("loading");
 });
 
 // 复选框
 <div class="btn-group" data-toggle="buttons">
 	<label class="btn btn-primary">
 		<input type="checkbox"> 选项 1
 	</label>
 	<label class="btn btn-primary">
 		<input type="checkbox"> 选项 2
 	</label>
 	<label class="btn btn-primary">
 		<input type="checkbox"> 选项 3
 	</label>
 </div>
 
 // 单选框
 <div class="btn-group" data-toggle="buttons">
 	<label class="btn btn-primary">
 		<input type="radio" name="options" id="option1"> 选项 1
 	</label>
 	<label class="btn btn-primary">
 		<input type="radio" name="options" id="option2"> 选项 2
 	</label>
 	<label class="btn btn-primary">
 		<input type="radio" name="options" id="option3"> 选项 3
 	</label>
 </div>	
```
#### 2、源码分析
1、setState   
设置按钮状态，处理按钮状态切换   
2、toggle   
切换单选、多选按钮激活状态   
3、`[data-toggle^="button"]`   
应用正则表达式监听 data-toggle="button" 和 data-toggle="buttons" 元素