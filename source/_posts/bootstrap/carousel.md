---
title: bootstrap 之 carousel
categories:
- bootstrap
---
这节介绍下 carousel(轮播)模块的源码实现。
<!--more-->
### 一、源码
#### 1、carousel.less
```
 //
 // Carousel(图片轮播)
 // --------------------------------------------------
 
 // Wrapper for the slide container and indicators(轮播图容器)
 .carousel {
   position: relative;
 }
 
 // 轮播图片播放区
 .carousel-inner {
   position: relative;
   overflow: hidden;
   width: 100%;
 
   > .item {
     display: none;
     position: relative;
     .transition(.6s ease-in-out left);
 
     // Account for jankitude on images
     > img,
     > a > img {
       &:extend(.img-responsive);
       line-height: 1;
     }
 
     // WebKit CSS3 transforms for supported devices
     @media all and (transform-3d), (-webkit-transform-3d) {
       .transition-transform(~'0.6s ease-in-out');
       .backface-visibility(~'hidden');
       .perspective(1000px);
 
       &.next,
       &.active.right {
         .translate3d(100%, 0, 0);
         left: 0;
       }
       &.prev,
       &.active.left {
         .translate3d(-100%, 0, 0);
         left: 0;
       }
       &.next.left,
       &.prev.right,
       &.active {
         .translate3d(0, 0, 0);
         left: 0;
       }
     }
   }
 
   > .active,
   > .next,
   > .prev {
     display: block;
   }
 
   > .active {
     left: 0;
   }
 
   > .next,
   > .prev {
     position: absolute;
     top: 0;
     width: 100%;
   }
 
   > .next {
     left: 100%;
   }
   > .prev {
     left: -100%;
   }
   > .next.left,
   > .prev.right {
     left: 0;
   }
 
   > .active.left {
     left: -100%;
   }
   > .active.right {
     left: 100%;
   }
 
 }
 
 // Left/right controls for nav(轮播图片控制器)
 // ---------------------------
 .carousel-control {
   position: absolute;
   top: 0;
   left: 0;
   bottom: 0;
   width: @carousel-control-width;
   .opacity(@carousel-control-opacity);
   font-size: @carousel-control-font-size;
   color: @carousel-control-color;
   text-align: center;
   text-shadow: @carousel-text-shadow;
   background-color: rgba(0, 0, 0, 0); // Fix IE9 click-thru bug
   // We can't have this transition here because WebKit cancels the carousel
   // animation if you trip this while in the middle of another animation.
 
   // Set gradients for backgrounds
   &.left {
     #gradient > .horizontal(@start-color: rgba(0,0,0,.5); @end-color: rgba(0,0,0,.0001));
   }
   &.right {
     left: auto;
     right: 0;
     #gradient > .horizontal(@start-color: rgba(0,0,0,.0001); @end-color: rgba(0,0,0,.5));
   }
 
   // Hover/focus state
   &:hover,
   &:focus {
     outline: 0;
     color: @carousel-control-color;
     text-decoration: none;
     .opacity(.9);
   }
 
   // Toggles
   .icon-prev,
   .icon-next,
   .glyphicon-chevron-left,
   .glyphicon-chevron-right {
     position: absolute;
     top: 50%;
     margin-top: -10px;
     z-index: 5;
     display: inline-block;
   }
   .icon-prev,
   .glyphicon-chevron-left {
     left: 50%;
     margin-left: -10px;
   }
   .icon-next,
   .glyphicon-chevron-right {
     right: 50%;
     margin-right: -10px;
   }
   .icon-prev,
   .icon-next {
     width:  20px;
     height: 20px;
     line-height: 1;
     font-family: serif;
   }
 
 
   .icon-prev {
     &:before {
       content: '\2039';// SINGLE LEFT-POINTING ANGLE QUOTATION MARK (U+2039)
     }
   }
   .icon-next {
     &:before {
       content: '\203a';// SINGLE RIGHT-POINTING ANGLE QUOTATION MARK (U+203A)
     }
   }
 }
 
 // Optional indicator pips(轮播图片计数器)
 //
 // Add an unordered list with the following class and add a list item for each
 // slide your carousel holds.
 .carousel-indicators {
   position: absolute;
   bottom: 10px;
   left: 50%;
   z-index: 15;
   width: 60%;
   margin-left: -30%;
   padding-left: 0;
   list-style: none;
   text-align: center;
 
   li {
     display: inline-block;
     width:  10px;
     height: 10px;
     margin: 1px;
     text-indent: -999px;
     border: 1px solid @carousel-indicator-border-color;
     border-radius: 10px;
     cursor: pointer;
 
     // IE8-9 hack for event handling
     //
     // Internet Explorer 8-9 does not support clicks on elements without a set
     // `background-color`. We cannot use `filter` since that's not viewed as a
     // background color by the browser. Thus, a hack is needed.
     // See https://developer.mozilla.org/en-US/docs/Web/Events/click#Internet_Explorer
     //
     // For IE8, we set solid black as it doesn't support `rgba()`. For IE9, we
     // set alpha transparency for the best results possible.
     background-color: #000 \9; // IE8
     background-color: rgba(0,0,0,0); // IE9
   }
   .active {
     margin: 0;
     width:  12px;
     height: 12px;
     background-color: @carousel-indicator-active-bg;
   }
 }
 
 // Optional captions(图片对应标题和描述内容)
 // -----------------------------
 // Hidden by default for smaller viewports
 .carousel-caption {
   position: absolute;
   left: 15%;
   right: 15%;
   bottom: 20px;
   z-index: 10;
   padding-top: 20px;
   padding-bottom: 20px;
   color: @carousel-caption-color;
   text-align: center;
   text-shadow: @carousel-text-shadow;
   & .btn {
     text-shadow: none; // No shadow for button elements in carousel-caption
   }
 }
 
 
 // Scale up controls for tablets and up(对于大屏设备放大一点控制器)
 @media screen and (min-width: @screen-sm-min) {
 
   // Scale up the controls a smidge(一点点)
   .carousel-control {
     .glyphicon-chevron-left,
     .glyphicon-chevron-right,
     .icon-prev,
     .icon-next {
       width: (@carousel-control-font-size * 1.5);
       height: (@carousel-control-font-size * 1.5);
       margin-top: (@carousel-control-font-size / -2);
       font-size: (@carousel-control-font-size * 1.5);
     }
     .glyphicon-chevron-left,
     .icon-prev {
       margin-left: (@carousel-control-font-size / -2);
     }
     .glyphicon-chevron-right,
     .icon-next {
       margin-right: (@carousel-control-font-size / -2);
     }
   }
 
   // Show and left align the captions
   .carousel-caption {
     left: 20%;
     right: 20%;
     padding-bottom: 30px;
   }
 
   // Move up the indicators
   .carousel-indicators {
     bottom: 20px;
   }
 }
```
#### 2、carousel.js
```
 /* ========================================================================
  * Bootstrap: carousel.js v3.3.7(图片轮播)
  * http://getbootstrap.com/javascript/#carousel
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * ======================================================================== */
 +function ($) {
   'use strict';
 
   // CAROUSEL CLASS DEFINITION
   // =========================
   var Carousel = function (element, options) {
     this.$element    = $(element);
     this.$indicators = this.$element.find('.carousel-indicators');
     this.options     = options;
     this.paused      = null;
     this.sliding     = null;  // 判断是否正在轮播项滚动
     this.interval    = null;  // 轮播循环播放定时器
     this.$active     = null;
     this.$items      = null;
 
     // 如果options.keyboard == true，则监听键盘事件
     this.options.keyboard && this.$element.on('keydown.bs.carousel', $.proxy(this.keydown, this));
 
     // 不是移动设备(不存在ontouchstart事件)，那么当options.pause == 'hover'时实现鼠标的悬浮暂停
     this.options.pause == 'hover' && !('ontouchstart' in document.documentElement) && this.$element
       .on('mouseenter.bs.carousel', $.proxy(this.pause, this))
       .on('mouseleave.bs.carousel', $.proxy(this.cycle, this))
   };
 
   Carousel.VERSION  = '3.3.7';
 
   Carousel.TRANSITION_DURATION = 600;
 
   Carousel.DEFAULTS = {
     interval: 5000,
     pause: 'hover',
     wrap: true,
     keyboard: true
   };
 
   // 键盘事件，做出相应操作
   Carousel.prototype.keydown = function (e) {
     if (/input|textarea/i.test(e.target.tagName)) return;
     switch (e.which) {
       case 37: this.prev(); break;
       case 39: this.next(); break;
       default: return
     }
 
     e.preventDefault()
   };
 
   // 循环轮播，清除定时器并重新设置定时器
   Carousel.prototype.cycle = function (e) {
     e || (this.paused = false);
 
     this.interval && clearInterval(this.interval);
 
     this.options.interval
       && !this.paused
       && (this.interval = setInterval($.proxy(this.next, this), this.options.interval));
 
     return this
   };
 
   // 获取轮播项索引值
   Carousel.prototype.getItemIndex = function (item) {
     this.$items = item.parent().children('.item');
     return this.$items.index(item || this.$active)
   };
 
   // 获取方向上的轮播项
   Carousel.prototype.getItemForDirection = function (direction, active) {
     var activeIndex = this.getItemIndex(active);
     var willWrap = (direction == 'prev' && activeIndex === 0)
                 || (direction == 'next' && activeIndex == (this.$items.length - 1));
     if (willWrap && !this.options.wrap) return active;
     var delta = direction == 'prev' ? -1 : 1;
     // 如果itemIndex为负数-1，不需要做处理，因为eq函数参数为负数时从集合最后一个元素开始倒数
     var itemIndex = (activeIndex + delta) % this.$items.length;
     return this.$items.eq(itemIndex)
   };
 
   // 滚动到指定轮播项
   Carousel.prototype.to = function (pos) {
     var that        = this
     var activeIndex = this.getItemIndex(this.$active = this.$element.find('.item.active'));
 
     if (pos > (this.$items.length - 1) || pos < 0) return;
 
     // 如果轮播图正在滚动切换，那么滚动到指定轮播项需要等到滚动切换结束(即监听到slid.bs.carousel)时才能继续操作
     if (this.sliding)       return this.$element.one('slid.bs.carousel', function () { that.to(pos) }); // yes, "slid"
     if (activeIndex == pos) return this.pause().cycle();
 
     return this.slide(pos > activeIndex ? 'next' : 'prev', this.$items.eq(pos))
   };
 
   // 暂停轮播，清除定时器
   Carousel.prototype.pause = function (e) {
     e || (this.paused = true);
 
     // 如果刚好为轮播添加了next/prev类即将开始滚动并且浏览器支持动画，鼠标移入，那么直接触发动画结束自定义事件
     if (this.$element.find('.next, .prev').length && $.support.transition) {
       this.$element.trigger($.support.transition.end);
       this.cycle(true)
     }
 
     this.interval = clearInterval(this.interval);
 
     return this
   };
 
   // 下一个轮播图
   Carousel.prototype.next = function () {
     // 如果轮播图正在滚动切换，那么上下轮播切换不做任何操作
     if (this.sliding) return;
     return this.slide('next')
   };
 
   // 上一个轮播图
   Carousel.prototype.prev = function () {
     if (this.sliding) return;
     return this.slide('prev')
   };
 
   // 滚动函数
   Carousel.prototype.slide = function (type, next) {
     var $active   = this.$element.find('.item.active');
     var $next     = next || this.getItemForDirection(type, $active);
     var isCycling = this.interval;
     var direction = type == 'next' ? 'left' : 'right';
     var that      = this;
 
     if ($next.hasClass('active')) return (this.sliding = false);
 
     var relatedTarget = $next[0];
     // 触发轮播即将开始自定义事件
     var slideEvent = $.Event('slide.bs.carousel', {
       relatedTarget: relatedTarget,
       direction: direction
     });
     this.$element.trigger(slideEvent);
     if (slideEvent.isDefaultPrevented()) return;
 
     this.sliding = true;
 
     isCycling && this.pause();
 
     // 为计数器切换active类
     if (this.$indicators.length) {
       this.$indicators.find('.active').removeClass('active');
       var $nextIndicator = $(this.$indicators.children()[this.getItemIndex($next)]);
       $nextIndicator && $nextIndicator.addClass('active')
     }
 
     var slidEvent = $.Event('slid.bs.carousel', { relatedTarget: relatedTarget, direction: direction }); // yes, "slid"
     if ($.support.transition && this.$element.hasClass('slide')) {
       $next.addClass(type);
       $next[0].offsetWidth // force reflow
       $active.addClass(direction);
       $next.addClass(direction);
       $active
         .one('bsTransitionEnd', function () {
           $next.removeClass([type, direction].join(' ')).addClass('active');
           $active.removeClass(['active', direction].join(' '));
           that.sliding = false;
           setTimeout(function () {
             that.$element.trigger(slidEvent)
           }, 0)
         })
         .emulateTransitionEnd(Carousel.TRANSITION_DURATION)
     } else {
       $active.removeClass('active');
       $next.addClass('active');
       this.sliding = false;
       this.$element.trigger(slidEvent)
     }
 
     isCycling && this.cycle();
 
     return this
   };
 
 
   // CAROUSEL PLUGIN DEFINITION
   // ==========================
   function Plugin(option) {
     return this.each(function () {
       var $this   = $(this);
       var data    = $this.data('bs.carousel');
       var options = $.extend({}, Carousel.DEFAULTS, $this.data(), typeof option == 'object' && option);
       var action  = typeof option == 'string' ? option : options.slide;
 
       if (!data) $this.data('bs.carousel', (data = new Carousel(this, options)));
       // js方法直接触发轮播，跳转到指定轮播页
       if (typeof option == 'number') data.to(option);
       // 在点击上、下一个时触发轮播
       else if (action) data[action]();
       // 初始化加载，在data-ride="carousel"且data-interval==true情况下触发轮播
       else if (options.interval) data.pause().cycle()
     })
   }
 
   var old = $.fn.carousel;
 
   $.fn.carousel             = Plugin;
   $.fn.carousel.Constructor = Carousel;
 
   // CAROUSEL NO CONFLICT
   // ====================
   $.fn.carousel.noConflict = function () {
     $.fn.carousel = old;
     return this
   };
 
   // CAROUSEL DATA-API
   // =================
   var clickHandler = function (e) {
     var href;
     var $this   = $(this);
     var $target = $($this.attr('data-target') || (href = $this.attr('href')) && href.replace(/.*(?=#[^\s]+$)/, '')); // strip for ie7
     // 如果目标元素没有carousel类，说明不是carousel容器，不做任何处理
     if (!$target.hasClass('carousel')) return;
     var options = $.extend({}, $target.data(), $this.data());
     var slideIndex = $this.attr('data-slide-to');
     if (slideIndex) options.interval = false;
 
     Plugin.call($target, options);
 
     if (slideIndex) {
       $target.data('bs.carousel').to(slideIndex)
     }
 
     // 防止使用a标签改变了链接地址等，阻止默认事件发生
     e.preventDefault()
   };
 
   $(document)
     .on('click.bs.carousel.data-api', '[data-slide]', clickHandler)
     .on('click.bs.carousel.data-api', '[data-slide-to]', clickHandler);
 
   $(window).on('load', function () {
     $('[data-ride="carousel"]').each(function () {
       var $carousel = $(this)
       Plugin.call($carousel, $carousel.data())
     })
   })
 
 }(jQuery);
```
### 二、应用 & 源码分析
#### 1、应用
```
 <div id="myCarousel" class="carousel slide">
     <!-- 轮播（Carousel）指标 -->
     <ol class="carousel-indicators">
         <li data-target="#myCarousel" data-slide-to="0" 
             class="active"></li>
         <li data-target="#myCarousel" data-slide-to="1"></li>
         <li data-target="#myCarousel" data-slide-to="2"></li>
     </ol>   
     <!-- 轮播（Carousel）项目 -->
     <div class="carousel-inner">
         <div class="item active">
             <img src="/wp-content/uploads/2014/07/slide1.png" alt="First slide">
         </div>
         <div class="item">
             <img src="/wp-content/uploads/2014/07/slide2.png" alt="Second slide">
         </div>
         <div class="item">
             <img src="/wp-content/uploads/2014/07/slide3.png" alt="Third slide">
         </div>
     </div>
     <!-- 轮播（Carousel）导航 -->
     <a class="carousel-control left" href="#myCarousel" 
         data-slide="prev">&lsaquo;</a>
     <a class="carousel-control right" href="#myCarousel" 
         data-slide="next">&rsaquo;</a>
     <!-- 控制按钮 -->
     <div style="text-align:center;">
         <input type="button" class="btn start-slide" value="Start">
         <input type="button" class="btn pause-slide" value="Pause">
         <input type="button" class="btn prev-slide" value="Previous Slide">
         <input type="button" class="btn next-slide" value="Next Slide">
         <input type="button" class="btn slide-one" value="Slide 1">
         <input type="button" class="btn slide-two" value="Slide 2">            
         <input type="button" class="btn slide-three" value="Slide 3">
     </div>
 </div> 
 <script>
 $(function(){
         // 初始化轮播
         $(".start-slide").click(function(){
             $("#myCarousel").carousel('cycle');
         });
         // 停止轮播
         $(".pause-slide").click(function(){
             $("#myCarousel").carousel('pause');
         });
         // 循环轮播到上一个项目
         $(".prev-slide").click(function(){
             $("#myCarousel").carousel('prev');
         });
         // 循环轮播到下一个项目
         $(".next-slide").click(function(){
             $("#myCarousel").carousel('next');
         });
         // 循环轮播到某个特定的帧 
         $(".slide-one").click(function(){
             $("#myCarousel").carousel(0);
         });
         $(".slide-two").click(function(){
             $("#myCarousel").carousel(1);
         });
         $(".slide-three").click(function(){
             $("#myCarousel").carousel(2);
         });
     });
 </script>
```
#### 2、源码分析
##### 2.1、carousel.less
1、carousel(相对定位) 分为 carousel-indicators(指示器)、carousel-inner(轮播项目)、carousel-control(轮播导航)；   
2、为 carousel-inner 内的 item(轮播项)设置 left、active、right 等样式，操作 left 或者 translate3d(如果支持动画效果的话)设置轮播项的位置；
3、carousel-control(绝对定位) 分为 left、right 两种状态，并设置 hover、focus、icon 等样式；   
4、carousel-indicators、carousel-caption 均为绝对布局，并为大屏设置做了一点调整
##### 2.2、modal.js  
1、通过 data-ride="carousel" 或 $("#myCarousel").carousel() 进行轮播初始化；   
2、监听 data-slide-to 和 data-slide 属性值或者调用 carousel(option) 方法，做出相应滚动操作；   
3、支持键盘操作，监听键盘事件，执行 keydown 函数滚动；   
4、最为核心的方法是 slide，先找到当前页，然后获取下一页和滚动方向，通过设置切换轮播项的 class 达到滚动的目的