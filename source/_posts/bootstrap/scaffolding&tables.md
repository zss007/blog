---
title: bootstrap 之 scaffolding & tables 样式
categories:
- bootstrap
---
现在开始介绍 bootstrap 的 scaffolding(脚手架) 和 tables(表格)，包括 scaffolding.less、tables.less。
<!--more-->
### 一、源码
#### 1、scaffolding.less
##### 1.1、scaffolding.less源码
```
 //
 // Scaffolding(脚手架)
 // --------------------------------------------------
 
 
 // Reset the box-sizing(content-box:标准盒模型；border-box:IE低版本/怪异模式)
 //
 // Heads up! This reset may cause conflicts with some third-party widgets.
 // For recommendations on resolving such conflicts, see
 // http://getbootstrap.com/getting-started/#third-box-sizing
 * {
   .box-sizing(border-box);
 }
 *:before,
 *:after {
   .box-sizing(border-box);
 }
 
 
 // Body reset
 
 html {
   font-size: 10px;
   -webkit-tap-highlight-color: rgba(0,0,0,0);
 }
 
 body {
   font-family: @font-family-base;
   font-size: @font-size-base;
   line-height: @line-height-base;
   color: @text-color;
   background-color: @body-bg;
 }
 
 // Reset fonts for relevant elements
 input,
 button,
 select,
 textarea {
   font-family: inherit;
   font-size: inherit;
   line-height: inherit;
 }
 
 
 // Links
 
 a {
   color: @link-color;
   text-decoration: none;
 
   &:hover,
   &:focus {
     color: @link-hover-color;
     text-decoration: @link-hover-decoration;
   }
 
   &:focus {
     .tab-focus();
   }
 }
 
 
 // Figures
 //
 // We reset this here because previously Normalize had no `figure` margins. This
 // ensures we don't break anyone's use of the element.
 
 figure {
   margin: 0;
 }
 
 
 // Images(图像)
 
 img {
   vertical-align: middle;
 }
 
 // Responsive images (ensure images don't scale beyond their parents，响应式图片展示防止图片拉伸超出父元素)
 .img-responsive {
   .img-responsive();
 }
 
 // Rounded corners(圆角)
 .img-rounded {
   border-radius: @border-radius-large;
 }
 
 // Image thumbnails(缩略图)
 //
 // Heads up! This is mixin-ed into thumbnails.less for `.thumbnail`.
 .img-thumbnail{
   padding: @thumbnail-padding;
   line-height: @line-height-base;
   background-color: @thumbnail-bg;
   border: 1px solid @thumbnail-border;
   border-radius: @thumbnail-border-radius;
   .transition(all .2s ease-in-out);
 
   // Keep them at most 100% wide
   .img-responsive(inline-block);
 }
 
 // Perfect circle(圆图片，设置border-radius为50%)
 .img-circle {
   border-radius: 50%; // set radius in percents
 }
 
 
 // Horizontal rules
 
 hr {
   margin-top:    @line-height-computed;
   margin-bottom: @line-height-computed;
   border: 0;
   border-top: 1px solid @hr-border;
 }
 
 
 // Only display content to screen readers
 //
 // 只在屏幕阅读器上使用
 // See: http://a11yproject.com/posts/how-to-hide-content
 
 .sr-only {
   position: absolute;
   width: 1px;
   height: 1px;
   margin: -1px;
   padding: 0;
   overflow: hidden;
   clip: rect(0,0,0,0);
   border: 0;
 }
 
 // Use in conjunction(联合，连接) with .sr-only to only display content when it's focused.
 // Useful for "Skip to main content" links; see http://www.w3.org/TR/2013/NOTE-WCAG20-TECHS-20130905/G1
 // Credit: HTML5 Boilerplate
 
 .sr-only-focusable {
   &:active,
   &:focus {
     position: static;
     width: auto;
     height: auto;
     margin: 0;
     overflow: visible;
     clip: auto;
   }
 }
 
 
 // iOS "clickable elements" fix for role="button"
 //
 // 修复iOS的addEventListener监听click事件在不是交互元素上不触发的bug
 // 方案:1、设置cursor: pointer；2、添加onclick属性；3、使用交互性的元素如a；4、不使用click，使用事件委托到父元素
 // Fixes "clickability" issue (and more generally(更普遍地，更概括地), the firing of events such as focus as well)
 // for traditionally non-focusable elements with role="button"
 // see https://developer.mozilla.org/en-US/docs/Web/Events/click#Safari_Mobile
 
 [role="button"] {
   cursor: pointer;
 }
```
##### 1.2、scaffolding.less应用
```
 <!--这里需要介绍图像 img 的应用-->
 <img src="http://www.runoob.com/wp-content/uploads/2014/06/download.png" class="img-rounded">
 <img src="http://www.runoob.com/wp-content/uploads/2014/06/download.png" class="img-circle">
 <img src="http://www.runoob.com/wp-content/uploads/2014/06/download.png" class="img-thumbnail">
 <img src="http://www.runoob.com/wp-content/uploads/2014/06/download.png" class="img-responsive">
```
#### 2、tables.less
##### 2.1、tables.less源码
```
 //
 // Tables 表格
 // --------------------------------------------------
 
 
 table {
   background-color: @table-bg;
 }
 caption {
   padding-top: @table-cell-padding;
   padding-bottom: @table-cell-padding;
   color: @text-muted;
   text-align: left;
 }
 th {
   text-align: left;
 }
 
 
 // Baseline styles(基础表格)
 
 .table {
   width: 100%;
   max-width: 100%;
   margin-bottom: @line-height-computed;
   // Cells
   > thead,
   > tbody,
   > tfoot {
     > tr {
       > th,
       > td {
         padding: @table-cell-padding;
         line-height: @line-height-base;
         vertical-align: top;
         border-top: 1px solid @table-border-color;
       }
     }
   }
   // Bottom align for column headings
   > thead > tr > th {
     vertical-align: bottom;
     border-bottom: 2px solid @table-border-color;
   }
   // Remove top border from thead by default
   > caption + thead,
   > colgroup + thead,
   > thead:first-child {
     > tr:first-child {
       > th,
       > td {
         border-top: 0;
       }
     }
   }
   // Account for multiple tbody instances(+: 相邻选择符)
   > tbody + tbody {
     border-top: 2px solid @table-border-color;
   }
 
   // Nesting
   .table {
     background-color: @body-bg;
   }
 }
 
 
 // Condensed table w/ half padding(紧凑型表格)
 
 .table-condensed {
   > thead,
   > tbody,
   > tfoot {
     > tr {
       > th,
       > td {
         padding: @table-condensed-cell-padding;
       }
     }
   }
 }
 
 
 // Bordered version(带边框表格)
 //
 // Add borders all around the table and between all the columns.
 
 .table-bordered {
   border: 1px solid @table-border-color;
   > thead,
   > tbody,
   > tfoot {
     > tr {
       > th,
       > td {
         border: 1px solid @table-border-color;
       }
     }
   }
   > thead > tr {
     > th,
     > td {
       border-bottom-width: 2px;
     }
   }
 }
 
 
 // Zebra-striping(斑马线表格)
 //
 // Default zebra-stripe styles (alternating gray and transparent backgrounds)
 
 .table-striped {
   > tbody > tr:nth-of-type(odd) {
     background-color: @table-bg-accent;
   }
 }
 
 
 // Hover effect(鼠标悬浮高亮的表格)
 //
 // Placed here since it has to come after the potential zebra striping
 
 .table-hover {
   > tbody > tr:hover {
     background-color: @table-bg-hover;
   }
 }
 
 
 // Table cell sizing
 //
 // Reset default table behavior('class*=': 选择所有类名中含有'col-'的元素; ^=: 表示开头; $=: 表示结尾)
 
 table col[class*="col-"] {
   position: static; // Prevent border hiding in Firefox and IE9-11 (see https://github.com/twbs/bootstrap/issues/11623)
   float: none;
   display: table-column;
 }
 table {
   td,
   th {
     &[class*="col-"] {
       position: static; // Prevent border hiding in Firefox and IE9-11 (see https://github.com/twbs/bootstrap/issues/11623)
       float: none;
       display: table-cell;
     }
   }
 }
 
 
 // Table backgrounds
 //
 // Exact selectors below required to override `.table-striped` and prevent
 // inheritance to nested tables.
 
 // Generate the contextual variants
 // 当前活动
 .table-row-variant(active; @table-bg-active);
 // 成功或者正确的行为
 .table-row-variant(success; @state-success-bg);
 // 中立的信息或行为
 .table-row-variant(info; @state-info-bg);
 // 表示警告，需要特别注意
 .table-row-variant(warning; @state-warning-bg);
 // 表示危险或者可能是错误的行为
 .table-row-variant(danger; @state-danger-bg);
 
 
 // Responsive tables(响应式表格)
 //
 // Wrap your tables in `.table-responsive` and we'll make them mobile friendly
 // by enabling horizontal scrolling. Only applies <768px. Everything above that
 // will display normally.
 
 .table-responsive {
   overflow-x: auto;
   min-height: 0.01%; // Workaround for IE9 bug (see https://github.com/twbs/bootstrap/issues/14837)
 
   @media screen and (max-width: @screen-xs-max) {
     width: 100%;
     margin-bottom: (@line-height-computed * 0.75);
     overflow-y: hidden;
     -ms-overflow-style: -ms-autohiding-scrollbar;
     border: 1px solid @table-border-color;
 
     // Tighten up spacing
     > .table {
       margin-bottom: 0;
 
       // Ensure the content doesn't wrap
       > thead,
       > tbody,
       > tfoot {
         > tr {
           > th,
           > td {
             white-space: nowrap;
           }
         }
       }
     }
 
     // Special overrides for the bordered tables
     > .table-bordered {
       border: 0;
 
       // Nuke the appropriate borders so that the parent can handle them
       > thead,
       > tbody,
       > tfoot {
         > tr {
           > th:first-child,
           > td:first-child {
             border-left: 0;
           }
           > th:last-child,
           > td:last-child {
             border-right: 0;
           }
         }
       }
 
       // Only nuke the last row's bottom-border in `tbody` and `tfoot` since
       // chances are there will be only one `tr` in a `thead` and that would
       // remove the border altogether.
       > tbody,
       > tfoot {
         > tr:last-child {
           > th,
           > td {
             border-bottom: 0;
           }
         }
       }
 
     }
   }
 }
```
##### 2.2、tables.less应用
```
 <!--table-responsive:响应式表格；上下文类：active等；table-bordered：边框表格；table-condensed：精简表格；table-hover：悬停表格；table-striped：条纹表格-->
 <table class="table table-responsive table-bordered table-condensed table-hover">
   <caption>表格布局</caption>
   <thead>
   <tr class="active">
     <th>名称</th>
     <th>城市</th>
     <th>邮编</th>
   </tr>
   </thead>
   <tbody>
   <tr class="success">
     <td>Tanmay</td>
     <td>Bangalore</td>
     <td>560001</td>
   </tr>
   <tr class="danger">
     <td>Sachin</td>
     <td>Mumbai</td>
     <td>400003</td>
   </tr>
   <tr class="warning">
     <td>Uma</td>
     <td>Pune</td>
     <td>411027</td>
   </tr>
   </tbody>
 </table>
```