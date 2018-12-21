---
title: bootstrap 之 affix
categories:
- bootstrap
---
这节介绍下 affix(自动定位浮标)模块的源码实现。
<!--more-->
### 一、源码
```
 /* ========================================================================
  * Bootstrap: affix.js v3.3.7(自动定位浮标)
  * http://getbootstrap.com/javascript/#affix
  * ========================================================================
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  * bottom使用时会有bug(见107行)
  * ======================================================================== */
 
 +function ($) {
   'use strict';
 
   // AFFIX CLASS DEFINITION
   // ======================
   var Affix = function (element, options) {
     this.options = $.extend({}, Affix.DEFAULTS, options)
 
     this.$target = $(this.options.target)
       .on('scroll.bs.affix.data-api', $.proxy(this.checkPosition, this))
       .on('click.bs.affix.data-api',  $.proxy(this.checkPositionWithEventLoop, this))
 
     this.$element     = $(element)
     this.affixed      = null
     this.unpin        = null
     this.pinnedOffset = null
 
     this.checkPosition()
   }
 
   Affix.VERSION  = '3.3.7'
 
   Affix.RESET    = 'affix affix-top affix-bottom'
 
   Affix.DEFAULTS = {
     offset: 0,
     target: window
   }
 
   // 获取滚动状态
   Affix.prototype.getState = function (scrollHeight, height, offsetTop, offsetBottom) {
     var scrollTop    = this.$target.scrollTop()  // 滚动元素(目标元素|Window)的滚动距离
     var position     = this.$element.offset()    // 元素相对于视口的距离
     var targetHeight = this.$target.height()     // 滚动元素(目标元素|Window)的可视高度
 
     // 后续滚动的话，如果滚动元素的滚动距离少于自定义向上偏移量offsetTop，返回'top'
     if (offsetTop != null && this.affixed == 'top') return scrollTop < offsetTop ? 'top' : false
 
     if (this.affixed == 'bottom') {
       // target的top＋getpinnedOffset的值>粘住元素当前定位到top的值, 返回'bottom'
       if (offsetTop != null) return (scrollTop + this.unpin <= position.top) ? false : 'bottom'
       // target的top＋target元素的高度>文档高度–粘住元素距离底部的高度
       return (scrollTop+targetHeight<=scrollHeight-offsetBottom) ? false : 'bottom'
     }
 
     // 初次运行执行代码
     var initializing   = this.affixed == null
     var colliderTop    = initializing ? scrollTop : position.top
     var colliderHeight = initializing ? targetHeight : height
 
     // 初始时，如果滚动元素的滚动距离少于等于自定义向上偏移量offsetTop，返回'top'
     if (offsetTop != null && scrollTop <= offsetTop) return 'top'
     // 初始时，如果滚动元素的滚动距离+滚动元素的高度+自定义向下偏移量offsetBottom大于内容高度，返回'bottom'
     // 滚动时，如果元素的offsetTop(距顶部距离)+元素的高度+自定义向下偏移量offsetBottom大于内容高度，返回'bottom'
     if (offsetBottom != null && (colliderTop + colliderHeight >= scrollHeight - offsetBottom)) return 'bottom'
 
     return false
   }
 
   // 获取粘住元素top–target滚动条的top
   Affix.prototype.getPinnedOffset = function () {
     if (this.pinnedOffset) return this.pinnedOffset
     this.$element.removeClass(Affix.RESET).addClass('affix')
     var scrollTop = this.$target.scrollTop()
     var position  = this.$element.offset()
     return (this.pinnedOffset = position.top - scrollTop)
   }
 
   // 如果点击目标元素，那么滚动元素会通过锚点滚动，然后执行checkPosition，判断菜单位置
   Affix.prototype.checkPositionWithEventLoop = function () {
     setTimeout($.proxy(this.checkPosition, this), 1)
   }
 
   // 检查位置信息
   Affix.prototype.checkPosition = function () {
     // 如果data-spy="affix"元素不可见，那么直接返回
     if (!this.$element.is(':visible')) return
 
     var height       = this.$element.height()
     var offset       = this.options.offset
     var offsetTop    = offset.top
     var offsetBottom = offset.bottom
     // 文档内容高度
     var scrollHeight = Math.max($(document).height(), $(document.body).height())
 
     // 如果offset不是对象，那么offsetBottom、offsetTop均为offset
     if (typeof offset != 'object')         offsetBottom = offsetTop = offset
     // 如果offsetTop、offsetBottom为function，那么offsetTop、offsetBottom将执行并以this.$element为参数得到结果作为值
     if (typeof offsetTop == 'function')    offsetTop    = offset.top(this.$element)
     if (typeof offsetBottom == 'function') offsetBottom = offset.bottom(this.$element)
 
     var affix = this.getState(scrollHeight, height, offsetTop, offsetBottom)
 
     if (this.affixed != affix) {
       // 从affix-bottom到其他状态，设置top样式为0
       if (this.unpin != null) this.$element.css('top', '')
       // 去掉行内样式position，不然会覆盖掉position为fixed的affix类样式，affix的bug
         // , this.$element.css('position', '')
 
       var affixType = 'affix' + (affix ? '-' + affix : '')
 
       var e         = $.Event(affixType + '.bs.affix')
       this.$element.trigger(e)
       if (e.isDefaultPrevented()) return
 
       this.affixed = affix
       this.unpin = affix == 'bottom' ? this.getPinnedOffset() : null
 
       this.$element
         .removeClass(Affix.RESET)
         .addClass(affixType)
         .trigger(affixType.replace('affix', 'affixed') + '.bs.affix')
     }
 
     // 到达底部时，去掉fixed后，改成relative，设置top属性为刚为bottom时的位置
     if (affix == 'bottom') {
       this.$element.offset({
         top: scrollHeight - height - offsetBottom
       })
     }
   }
 
 
   // AFFIX PLUGIN DEFINITION
   // =======================
   function Plugin(option) {
     return this.each(function () {
       var $this   = $(this)
       var data    = $this.data('bs.affix')
       var options = typeof option == 'object' && option
 
       if (!data) $this.data('bs.affix', (data = new Affix(this, options)))
       if (typeof option == 'string') data[option]()
     })
   }
 
   var old = $.fn.affix
 
   $.fn.affix             = Plugin
   $.fn.affix.Constructor = Affix
 
 
   // AFFIX NO CONFLICT
   // =================
   $.fn.affix.noConflict = function () {
     $.fn.affix = old
     return this
   }
 
 
   // AFFIX DATA-API
   // ==============
   $(window).on('load', function () {
     $('[data-spy="affix"]').each(function () {
       var $spy = $(this)
       var data = $spy.data()
 
       data.offset = data.offset || {}
 
       if (data.offsetBottom != null) data.offset.bottom = data.offsetBottom
       if (data.offsetTop    != null) data.offset.top    = data.offsetTop
 
       Plugin.call($spy, data)
     })
   })
 }(jQuery);
```
### 二、utilities.less 源码
```
 //
 // Utility classes
 // --------------------------------------------------
 
 
 // Floats
 // -------------------------
 
 .clearfix {
   .clearfix();
 }
 .center-block {
   .center-block();
 }
 .pull-right {
   float: right !important;
 }
 .pull-left {
   float: left !important;
 }
 
 
 // Toggling content
 // -------------------------
 
 // Note: Deprecated .hide in favor of .hidden or .sr-only (as appropriate) in v3.0.1
 .hide {
   display: none !important;
 }
 .show {
   display: block !important;
 }
 .invisible {
   visibility: hidden;
 }
 .text-hide {
   .text-hide();
 }
 
 
 // Hide from screenreaders and browsers
 //
 // Credit: HTML5 Boilerplate
 
 .hidden {
   display: none !important;
 }
 
 
 // For Affix plugin(固定定位插件)
 // -------------------------
 .affix {
   position: fixed;
 }
```
### 三、应用 & 源码分析
#### 1、应用
```
 <style>
    /* Custom Styles */
    ul.nav-tabs {
      width: 140px;
      margin-top: 20px;
      border-radius: 4px;
      border: 1px solid #ddd;
      box-shadow: 0 1px 4px rgba(0, 0, 0, 0.067);
    }

    ul.nav-tabs li {
      margin: 0;
      border-top: 1px solid #ddd;
    }

    ul.nav-tabs li:first-child {
      border-top: none;
    }

    ul.nav-tabs li a {
      margin: 0;
      padding: 8px 16px;
      border-radius: 0;
    }

    ul.nav-tabs li.active a, ul.nav-tabs li.active a:hover {
      color: #fff;
      background: #0088cc;
      border: 1px solid #0088cc;
    }

    ul.nav-tabs li:first-child a {
      border-radius: 4px 4px 0 0;
    }

    ul.nav-tabs li:last-child a {
      border-radius: 0 0 4px 4px;
    }

    ul.nav-tabs.affix {
      top: 30px; /* Set the top position of pinned element */
    }
 </style>
 <body data-spy="scroll" data-target="#myScrollspy">
 <script>
   $(document).ready(function () {
     $("#myNav").affix({
       offset: {
         top: 125,
         bottom: function () {
           return (this.bottom = $('#comments').outerHeight(true) + $('#footer').outerHeight(true) + 40)
         }
       }
     });
   });
 </script>
 <div class="container">
   <div class="jumbotron">
     <h1>Bootstrap Affix</h1>
   </div>
   <div class="row">
     <div class="col-xs-3" id="myScrollspy">
       <ul class="nav nav-tabs nav-stacked" id="myNav">
         <li class="active"><a href="#section-1">第一部分</a></li>
         <li><a href="#section-2">第二部分</a></li>
         <li><a href="#section-3">第三部分</a></li>
         <li><a href="#section-4">第四部分</a></li>
         <li><a href="#section-5">第五部分</a></li>
       </ul>
     </div>
     <div class="col-xs-9">
       <h2 id="section-1">第一部分</h2>
       <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam eu sem tempor, varius quam at, luctus dui. Mauris
         magna metus, dapibus nec turpis vel, semper malesuada ante. Vestibulum id metus ac nisl bibendum scelerisque non
         non purus. Suspendisse varius nibh non aliquet sagittis. In tincidunt orci sit amet elementum vestibulum.
         Vivamus fermentum in arcu in aliquam. Quisque aliquam porta odio in fringilla. Vivamus nisl leo, blandit at
         bibendum eu, tristique eget risus. Integer aliquet quam ut elit suscipit, id interdum neque porttitor. Integer
         faucibus ligula.</p>
       <p>Vestibulum quis quam ut magna consequat faucibus. Pellentesque eget nisi a mi suscipit tincidunt. Ut tempus
         dictum risus. Pellentesque viverra sagittis quam at mattis. Suspendisse potenti. Aliquam sit amet gravida nibh,
         facilisis gravida odio. Phasellus auctor velit at lacus blandit, commodo iaculis justo viverra. Etiam vitae est
         arcu. Mauris vel congue dolor. Aliquam eget mi mi. Fusce quam tortor, commodo ac dui quis, bibendum viverra
         erat. Maecenas mattis lectus enim, quis tincidunt dui molestie euismod. Curabitur et diam tristique, accumsan
         nunc eu, hendrerit tellus.</p>
       <hr>
       <h2 id="section-2">第二部分</h2>
       <p>Nullam hendrerit justo non leo aliquet imperdiet. Etiam in sagittis lectus. Suspendisse ultrices placerat
         accumsan. Mauris quis dapibus orci. In dapibus velit blandit pharetra tincidunt. Quisque non sapien nec lacus
         condimentum facilisis ut iaculis enim. Sed viverra interdum bibendum. Donec ac sollicitudin dolor. Sed fringilla
         vitae lacus at rutrum. Phasellus congue vestibulum ligula sed consequat.</p>
       <p>Vestibulum consectetur scelerisque lacus, ac fermentum lorem convallis sed. Nam odio tortor, dictum quis
         malesuada at, pellentesque vitae orci. Vivamus elementum, felis eu auctor lobortis, diam velit egestas lacus,
         quis fermentum metus ante quis urna. Sed at facilisis libero. Cum sociis natoque penatibus et magnis dis
         parturient montes, nascetur ridiculus mus. Vestibulum bibendum blandit dolor. Nunc orci dolor, molestie nec nibh
         in, hendrerit tincidunt ante. Vivamus sem augue, hendrerit non sapien in, mollis ornare augue.</p>
       <hr>
       <h2 id="section-3">第三部分</h2>
       <p>Integer pulvinar leo id risus pellentesque vestibulum. Sed diam libero, sodales eget sapien vel, porttitor
         bibendum enim. Donec sed nibh vitae lorem porttitor blandit in nec ante. Pellentesque vitae metus ipsum.
         Phasellus sed nunc ac sem malesuada condimentum. Etiam in aliquam lectus. Nam vel sapien diam. Donec pharetra id
         arcu eget blandit. Proin imperdiet mattis augue in porttitor. Quisque tempus enim id lobortis feugiat.
         Suspendisse tincidunt risus quis dolor fringilla blandit. Ut sed sapien at purus lacinia porttitor. Nullam
         iaculis, felis a pretium ornare, dolor nisl semper tortor, vel sagittis lacus est consequat eros. Sed id pretium
         nisl. Curabitur dolor nisl, laoreet vitae aliquam id, tincidunt sit amet mauris.</p>
       <p>Phasellus vitae suscipit justo. Mauris pharetra feugiat ante id lacinia. Etiam faucibus mauris id tempor
         egestas. Duis luctus turpis at accumsan tincidunt. Phasellus risus risus, volutpat vel tellus ac, tincidunt
         fringilla massa. Etiam hendrerit dolor eget ante rutrum adipiscing. Cras interdum ipsum mattis, tempus mauris
         vel, semper ipsum. Duis sed dolor ut enim lobortis pellentesque ultricies ac ligula. Pellentesque convallis elit
         nisi, id vulputate ipsum ullamcorper ut. Cras ac pulvinar purus, ac viverra est. Suspendisse potenti. Integer
         pellentesque neque et elementum tempus. Curabitur bibendum in ligula ut rhoncus.</p>
       <p>Quisque pharetra velit id velit iaculis pretium. Nullam a justo sed ligula porta semper eu quis enim.
         Pellentesque pellentesque, metus at facilisis hendrerit, lectus velit facilisis leo, quis volutpat turpis arcu
         quis enim. Nulla viverra lorem elementum interdum ultricies. Suspendisse accumsan quam nec ante mollis tempus.
         Morbi vel accumsan diam, eget convallis tellus. Suspendisse potenti.</p>
       <hr>
       <h2 id="section-4">第四部分</h2>
       <p>Suspendisse a orci facilisis, dignissim tortor vitae, ultrices mi. Vestibulum a iaculis lacus. Phasellus vitae
         convallis ligula, nec volutpat tellus. Vivamus scelerisque mollis nisl, nec vehicula elit egestas a. Sed luctus
         metus id mi gravida, faucibus convallis neque pretium. Maecenas quis sapien ut leo fringilla tempor vitae sit
         amet leo. Donec imperdiet tempus placerat. Pellentesque pulvinar ultrices nunc sed ultrices. Morbi vel mi
         pretium, fermentum lacus et, viverra tellus. Phasellus sodales libero nec dui convallis, sit amet fermentum
         sapien auctor. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Sed eu
         elementum nibh, quis varius libero.</p>
       <p>Vestibulum quis quam ut magna consequat faucibus. Pellentesque eget nisi a mi suscipit tincidunt. Ut tempus
         dictum risus. Pellentesque viverra sagittis quam at mattis. Suspendisse potenti. Aliquam sit amet gravida nibh,
         facilisis gravida odio. Phasellus auctor velit at lacus blandit, commodo iaculis justo viverra. Etiam vitae est
         arcu. Mauris vel congue dolor. Aliquam eget mi mi. Fusce quam tortor, commodo ac dui quis, bibendum viverra
         erat. Maecenas mattis lectus enim, quis tincidunt dui molestie euismod. Curabitur et diam tristique, accumsan
         nunc eu, hendrerit tellus.</p>
       <p>Phasellus fermentum, neque sit amet sodales tempor, enim ante interdum eros, eget luctus ipsum eros ut ligula.
         Nunc ornare erat quis faucibus molestie. Proin malesuada consequat commodo. Mauris iaculis, eros ut dapibus
         luctus, massa enim elementum purus, sit amet tristique purus purus nec felis. Morbi vestibulum sapien eget porta
         pulvinar. Nam at quam diam. Proin rhoncus, felis elementum accumsan dictum, felis nisi vestibulum tellus, et
         ultrices risus felis in orci. Quisque vestibulum sem nisl, vel congue leo dictum nec. Cras eget est at velit
         sagittis ullamcorper vel et lectus. In hac habitasse platea dictumst. Etiam interdum iaculis velit, vel
         sollicitudin lorem feugiat sit amet. Etiam luctus, quam sed sodales aliquam, lorem libero hendrerit urna,
         faucibus rhoncus massa nibh at felis. Curabitur ac tempus nulla, ut semper erat. Vivamus porta ullamcorper sem,
         ornare egestas mauris facilisis id.</p>
       <p>Ut ut risus nisl. Fusce porttitor eros at magna luctus, non congue nulla eleifend. Aenean porttitor feugiat
         dolor sit amet facilisis. Pellentesque venenatis magna et risus commodo, a commodo turpis gravida. Nam mollis
         massa dapibus urna aliquet, quis iaculis elit sodales. Sed eget ornare orci, eu malesuada justo. Nunc lacus
         augue, dictum quis dui id, lacinia congue quam. Nulla sem sem, aliquam nec dolor ac, tempus convallis nunc.
         Interdum et malesuada fames ac ante ipsum primis in faucibus. Nulla suscipit convallis iaculis. Quisque eget
         commodo ligula. Praesent leo dui, facilisis quis eleifend in, aliquet vitae nunc. Suspendisse fermentum odio ac
         massa ultricies pellentesque. Fusce eu suscipit massa.</p>
       <hr>
       <h2 id="section-5">第五部分</h2>
       <p>Nam eget purus nec est consectetur vehicula. Nullam ultrices nisl risus, in viverra libero egestas sit amet.
         Etiam porttitor dolor non eros pulvinar malesuada. Vestibulum sit amet est mollis nulla tempus aliquet. Praesent
         luctus hendrerit arcu non laoreet. Morbi consequat placerat magna, ac ornare odio sagittis sed. Donec vitae
         ullamcorper purus. Vivamus non metus ac justo porta volutpat.</p>
       <p>Vivamus mattis accumsan erat, vel convallis risus pretium nec. Integer nunc nulla, viverra ut sem non,
         scelerisque vehicula arcu. Fusce bibendum convallis augue sit amet lobortis. Cras porta urna turpis, sodales
         lobortis purus adipiscing id. Maecenas ullamcorper, turpis suscipit pellentesque fringilla, massa lacus pulvinar
         mi, nec dignissim velit arcu eget purus. Nam at dapibus tellus, eget euismod nisl. Ut eget venenatis sapien.
         Vivamus vulputate varius mauris, vel varius nisl facilisis ac. Nulla aliquet justo a nibh ornare, eu congue
         neque rutrum.</p>
       <p>Suspendisse a orci facilisis, dignissim tortor vitae, ultrices mi. Vestibulum a iaculis lacus. Phasellus vitae
         convallis ligula, nec volutpat tellus. Vivamus scelerisque mollis nisl, nec vehicula elit egestas a. Sed luctus
         metus id mi gravida, faucibus convallis neque pretium. Maecenas quis sapien ut leo fringilla tempor vitae sit
         amet leo. Donec imperdiet tempus placerat. Pellentesque pulvinar ultrices nunc sed ultrices. Morbi vel mi
         pretium, fermentum lacus et, viverra tellus. Phasellus sodales libero nec dui convallis, sit amet fermentum
         sapien auctor. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Sed eu
         elementum nibh, quis varius libero.</p>
       <p>Morbi sed fermentum ipsum. Morbi a orci vulputate tortor ornare blandit a quis orci. Donec aliquam sodales
         gravida. In ut ullamcorper nisi, ac pretium velit. Vestibulum vitae lectus volutpat, consequat lorem sit amet,
         pulvinar tellus. In tincidunt vel leo eget pulvinar. Curabitur a eros non lacus malesuada aliquam. Praesent et
         tempus odio. Integer a quam nunc. In hac habitasse platea dictumst. Aliquam porta nibh nulla, et mattis turpis
         placerat eget. Pellentesque dui diam, pellentesque vel gravida id, accumsan eu magna. Sed a semper arcu, ut
         dignissim leo.</p>
       <p>Sed vitae lobortis diam, id molestie magna. Aliquam consequat ipsum quis est dictum ultrices. Aenean nibh
         velit, fringilla in diam id, blandit hendrerit lacus. Donec vehicula rutrum tellus eget fermentum. Pellentesque
         ac erat et arcu ornare tincidunt. Aliquam erat volutpat. Vivamus lobortis urna quis gravida semper. In
         condimentum, est a faucibus luctus, mi dolor cursus mi, id vehicula arcu risus a nibh. Pellentesque blandit
         sapien lacus, vel vehicula nunc feugiat sit amet.</p>
     </div>
     <div class="clearfix"></div>
     <div id="comments">
       <p>Nam eget purus nec est consectetur vehicula. Nullam ultrices nisl risus, in viverra libero egestas sit amet.
         Etiam porttitor dolor non eros pulvinar malesuada. Vestibulum sit amet est mollis nulla tempus aliquet. Praesent
         luctus hendrerit arcu non laoreet. Morbi consequat placerat magna, ac ornare odio sagittis sed. Donec vitae
         ullamcorper purus. Vivamus non metus ac justo porta volutpat.</p>
       <p>Vivamus mattis accumsan erat, vel convallis risus pretium nec. Integer nunc nulla, viverra ut sem non,
         scelerisque vehicula arcu. Fusce bibendum convallis augue sit amet lobortis. Cras porta urna turpis, sodales
         lobortis purus adipiscing id. Maecenas ullamcorper, turpis suscipit pellentesque fringilla, massa lacus pulvinar
         mi, nec dignissim velit arcu eget purus. Nam at dapibus tellus, eget euismod nisl. Ut eget venenatis sapien.
         Vivamus vulputate varius mauris, vel varius nisl facilisis ac. Nulla aliquet justo a nibh ornare, eu congue
         neque rutrum.</p>
       <p>Suspendisse a orci facilisis, dignissim tortor vitae, ultrices mi. Vestibulum a iaculis lacus. Phasellus vitae
         convallis ligula, nec volutpat tellus. Vivamus scelerisque mollis nisl, nec vehicula elit egestas a. Sed luctus
         metus id mi gravida, faucibus convallis neque pretium. Maecenas quis sapien ut leo fringilla tempor vitae sit
         amet leo. Donec imperdiet tempus placerat. Pellentesque pulvinar ultrices nunc sed ultrices. Morbi vel mi
         pretium, fermentum lacus et, viverra tellus. Phasellus sodales libero nec dui convallis, sit amet fermentum
         sapien auctor. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Sed eu
         elementum nibh, quis varius libero.</p>
       <p>Morbi sed fermentum ipsum. Morbi a orci vulputate tortor ornare blandit a quis orci. Donec aliquam sodales
         gravida. In ut ullamcorper nisi, ac pretium velit. Vestibulum vitae lectus volutpat, consequat lorem sit amet,
         pulvinar tellus. In tincidunt vel leo eget pulvinar. Curabitur a eros non lacus malesuada aliquam. Praesent et
         tempus odio. Integer a quam nunc. In hac habitasse platea dictumst. Aliquam porta nibh nulla, et mattis turpis
         placerat eget. Pellentesque dui diam, pellentesque vel gravida id, accumsan eu magna. Sed a semper arcu, ut
         dignissim leo.</p>
       <p>Sed vitae lobortis diam, id molestie magna. Aliquam consequat ipsum quis est dictum ultrices. Aenean nibh
         velit, fringilla in diam id, blandit hendrerit lacus. Donec vehicula rutrum tellus eget fermentum. Pellentesque
         ac erat et arcu ornare tincidunt. Aliquam erat volutpat. Vivamus lobortis urna quis gravida semper. In
         condimentum, est a faucibus luctus, mi dolor cursus mi, id vehicula arcu risus a nibh. Pellentesque blandit
         sapien lacus, vel vehicula nunc feugiat sit amet.</p>
     </div>
     <div id="footer">
       <p>Nam eget purus nec est consectetur vehicula. Nullam ultrices nisl risus, in viverra libero egestas sit amet.
         Etiam porttitor dolor non eros pulvinar malesuada. Vestibulum sit amet est mollis nulla tempus aliquet. Praesent
         luctus hendrerit arcu non laoreet. Morbi consequat placerat magna, ac ornare odio sagittis sed. Donec vitae
         ullamcorper purus. Vivamus non metus ac justo porta volutpat.</p>
       <p>Vivamus mattis accumsan erat, vel convallis risus pretium nec. Integer nunc nulla, viverra ut sem non,
         scelerisque vehicula arcu. Fusce bibendum convallis augue sit amet lobortis. Cras porta urna turpis, sodales
         lobortis purus adipiscing id. Maecenas ullamcorper, turpis suscipit pellentesque fringilla, massa lacus pulvinar
         mi, nec dignissim velit arcu eget purus. Nam at dapibus tellus, eget euismod nisl. Ut eget venenatis sapien.
         Vivamus vulputate varius mauris, vel varius nisl facilisis ac. Nulla aliquet justo a nibh ornare, eu congue
         neque rutrum.</p>
       <p>Suspendisse a orci facilisis, dignissim tortor vitae, ultrices mi. Vestibulum a iaculis lacus. Phasellus vitae
         convallis ligula, nec volutpat tellus. Vivamus scelerisque mollis nisl, nec vehicula elit egestas a. Sed luctus
         metus id mi gravida, faucibus convallis neque pretium. Maecenas quis sapien ut leo fringilla tempor vitae sit
         amet leo. Donec imperdiet tempus placerat. Pellentesque pulvinar ultrices nunc sed ultrices. Morbi vel mi
         pretium, fermentum lacus et, viverra tellus. Phasellus sodales libero nec dui convallis, sit amet fermentum
         sapien auctor. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Sed eu
         elementum nibh, quis varius libero.</p>
       <p>Morbi sed fermentum ipsum. Morbi a orci vulputate tortor ornare blandit a quis orci. Donec aliquam sodales
         gravida. In ut ullamcorper nisi, ac pretium velit. Vestibulum vitae lectus volutpat, consequat lorem sit amet,
         pulvinar tellus. In tincidunt vel leo eget pulvinar. Curabitur a eros non lacus malesuada aliquam. Praesent et
         tempus odio. Integer a quam nunc. In hac habitasse platea dictumst. Aliquam porta nibh nulla, et mattis turpis
         placerat eget. Pellentesque dui diam, pellentesque vel gravida id, accumsan eu magna. Sed a semper arcu, ut
         dignissim leo.</p>
       <p>Sed vitae lobortis diam, id molestie magna. Aliquam consequat ipsum quis est dictum ultrices. Aenean nibh
         velit, fringilla in diam id, blandit hendrerit lacus. Donec vehicula rutrum tellus eget fermentum. Pellentesque
         ac erat et arcu ornare tincidunt. Aliquam erat volutpat. Vivamus lobortis urna quis gravida semper. In
         condimentum, est a faucibus luctus, mi dolor cursus mi, id vehicula arcu risus a nibh. Pellentesque blandit
         sapien lacus, vel vehicula nunc feugiat sit amet.</p>
     </div>
   </div>
 </div>
 </body>
```
#### 2、源码分析
1、getState   
开始时滚动元素滚动距离少于自定义 offsetTop 高度返回 affix-top ；当滚动元素滚动超过自定义 offsetTop 高度时，返回 affix；当目标元素滚动距离超过了(内容高度scrollHeight - offsetBottom - 元素高度targetHeight)时返回 affix-bottom  
2、checkPosition   
根据 getState 返回的状态进行判断设置相应的类，返回 affix-top 时不做任何处理；返回 affix 时设置 position 为 absolute；返回affix-bottom 时设置 position 为 relative，并设置 top 为相应的高度(源代码存在一个 bug，需在从 affix-bottom 状态转变为 affix 状态时设置行内 top 为空，同时设置清空行内 position)