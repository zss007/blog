---
title: bootstrap 之 alert
categories:
- bootstrap
---
这节介绍下 alert(警示框)的源码实现。
<!--more-->
### 一、源码
#### 1、alerts.less
```
 //
 // Alerts(警示框)
 // --------------------------------------------------
 
 // Base styles
 // -------------------------
 .alert {
   padding: @alert-padding;
   margin-bottom: @line-height-computed;
   border: 1px solid transparent;
   border-radius: @alert-border-radius;
 
   // Headings for larger alerts
   h4 {
     margin-top: 0;
     // Specified for the h4 to prevent conflicts of changing @headings-color
     color: inherit;
   }
 
   // Provide class for links that match alerts(警示框的链接)
   .alert-link {
     font-weight: @alert-link-font-weight;
   }
 
   // Improve alignment and spacing of inner content
   > p,
   > ul {
     margin-bottom: 0;
   }
 
   > p + p {
     margin-top: 5px;
   }
 }
 
 // Dismissible alerts(可关闭的警示框)
 //
 // Expand the right padding and account for the close button's positioning.
 .alert-dismissable, // The misspelled .alert-dismissable was deprecated in 3.2.0.
 .alert-dismissible {
   padding-right: (@alert-padding + 20);
 
   // Adjust close link position
   .close {
     position: relative;
     top: -2px;
     right: -21px;
     color: inherit;
   }
 }
 
 // Alternate styles(警示框风格)
 //
 // Generate contextual modifier classes for colorizing the alert.
 .alert-success {
   .alert-variant(@alert-success-bg; @alert-success-border; @alert-success-text);
 }
 
 .alert-info {
   .alert-variant(@alert-info-bg; @alert-info-border; @alert-info-text);
 }
 
 .alert-warning {
   .alert-variant(@alert-warning-bg; @alert-warning-border; @alert-warning-text);
 }
 
 .alert-danger {
   .alert-variant(@alert-danger-bg; @alert-danger-border; @alert-danger-text);
 }
```
#### 2、alert.js
```
 /* ========================================================================
  * Bootstrap: alert.js v3.3.7(警告框)
  * http://getbootstrap.com/javascript/#alerts
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * ======================================================================== */
 +function ($) {
   'use strict';
 
   // ALERT CLASS DEFINITION
   // ======================
   var dismiss = '[data-dismiss="alert"]';
   var Alert   = function (el) {
     $(el).on('click', dismiss, this.close)
   };
 
   Alert.VERSION = '3.3.7';
 
   Alert.TRANSITION_DURATION = 150;
 
   Alert.prototype.close = function (e) {
     var $this    = $(this);
     var selector = $this.attr('data-target');
 
     if (!selector) {
       selector = $this.attr('href');
       selector = selector && selector.replace(/.*(?=#[^\s]*$)/, '') // strip for ie7
     }
 
     var $parent = $(selector === '#' ? [] : selector);
 
     if (e) e.preventDefault();
 
     if (!$parent.length) {
       $parent = $this.closest('.alert')
     }
 
     $parent.trigger(e = $.Event('close.bs.alert'));
 
     if (e.isDefaultPrevented()) return;
 
     $parent.removeClass('in');
 
     function removeElement() {
       // detach from parent, fire event then clean up data
       $parent.detach().trigger('closed.bs.alert').remove()
     }
 
     $.support.transition && $parent.hasClass('fade') ?
       $parent
         .one('bsTransitionEnd', removeElement)
         .emulateTransitionEnd(Alert.TRANSITION_DURATION) :
       removeElement()
   };
 
   // ALERT PLUGIN DEFINITION
   // =======================
   function Plugin(option) {
     return this.each(function () {
       var $this = $(this);
       var data  = $this.data('bs.alert');
 
       if (!data) $this.data('bs.alert', (data = new Alert(this)));
       if (typeof option == 'string') data[option].call($this);
     })
   }
 
   var old = $.fn.alert;
   $.fn.alert             = Plugin;
   $.fn.alert.Constructor = Alert;
 
   // ALERT NO CONFLICT
   // =================
   $.fn.alert.noConflict = function () {
     $.fn.alert = old;
     return this
   };
 
   // ALERT DATA-API
   // ==============
   $(document).on('click.bs.alert.data-api', dismiss, Alert.prototype.close);
 }(jQuery);
```
### 二、应用 & 源码分析
#### 1、应用
```
 <div class="alert alert-success">
 	<a href="#" class="close" data-dismiss="alert">&times;</a>
 	<strong>成功！</strong>结果是成功的。
 </div>
 <div class="alert alert-warning">
 	<a href="#" class="close" data-dismiss="alert">&times;</a>
 	<strong>警告！</strong>您的网络连接有问题。
 </div>
```
#### 2、源码分析
##### 2.1、alerts.less
共有四种风格alert-success、alert-info、alert-warning、alert-danger
##### 2.2、alert.js
监听存在属性`data-dismiss="alert"`的元素点击事件，触发close事件，并判断是否存在动画fade。