---
title: bootstrap 之 navs & pagination 样式
categories:
- bootstrap
---
现在开始介绍 bootstrap 的 navs(导航) 和 pagination(分页)，包括 navs.less、pagination.less。
<!--more-->
### 一、源码
#### 1、navs.less
##### 1.1、navs.less源码
```
 //
 // Navs(导航)
 // --------------------------------------------------
 
 
 // Base class(基础样式)
 // --------------------------------------------------
 .nav {
   margin-bottom: 0;
   padding-left: 0; // Override default ul/ol
   list-style: none;
   &:extend(.clearfix all);
 
   > li {
     position: relative;
     display: block;
 
     > a {
       position: relative;
       display: block;
       padding: @nav-link-padding;
       &:hover,
       &:focus {
         text-decoration: none;
         background-color: @nav-link-hover-bg;
       }
     }
 
     // Disabled state sets text to gray and nukes hover/tab effects
     &.disabled > a {
       color: @nav-disabled-link-color;
 
       &:hover,
       &:focus {
         color: @nav-disabled-link-hover-color;
         text-decoration: none;
         background-color: transparent;
         cursor: @cursor-disabled;
       }
     }
   }
 
   // Open dropdowns
   .open > a {
     &,
     &:hover,
     &:focus {
       background-color: @nav-link-hover-bg;
       border-color: @link-color;
     }
   }
 
   // Nav dividers (deprecated with v3.0.1)
   //
   // This should have been removed in v3 with the dropping of `.nav-list`, but
   // we missed it. We don't currently support this anywhere, but in the interest
   // of maintaining backward compatibility in case you use it, it's deprecated.
   .nav-divider {
     .nav-divider();
   }
 
   // Prevent IE8 from misplacing imgs
   //
   // See https://github.com/h5bp/html5-boilerplate/issues/984#issuecomment-3985989
   > li > a > img {
     max-width: none;
   }
 }
 
 
 // Tabs(标签形tab导航)
 // -------------------------
 // Give the tabs something to sit on
 .nav-tabs {
   border-bottom: 1px solid @nav-tabs-border-color;
   > li {
     float: left;
     // Make the list-items overlay the bottom border
     margin-bottom: -1px;
 
     // Actual tabs (as links)
     > a {
       margin-right: 2px;
       line-height: @line-height-base;
       border: 1px solid transparent;
       border-radius: @border-radius-base @border-radius-base 0 0;
       &:hover {
         border-color: @nav-tabs-link-hover-border-color @nav-tabs-link-hover-border-color @nav-tabs-border-color;
       }
     }
 
     // Active state, and its :hover to override normal :hover
     &.active > a {
       &,
       &:hover,
       &:focus {
         color: @nav-tabs-active-link-hover-color;
         background-color: @nav-tabs-active-link-hover-bg;
         border: 1px solid @nav-tabs-active-link-hover-border-color;
         border-bottom-color: transparent;
         cursor: default;
       }
     }
   }
   // pulling this in mainly for less shorthand
   &.nav-justified {
     .nav-justified();
     .nav-tabs-justified();
   }
 }
 
 
 // Pills(胶囊形导航)
 // -------------------------
 .nav-pills {
   > li {
     float: left;
 
     // Links rendered as pills
     > a {
       border-radius: @nav-pills-border-radius;
     }
     + li {
       margin-left: 2px;
     }
 
     // Active state
     &.active > a {
       &,
       &:hover,
       &:focus {
         color: @nav-pills-active-link-hover-color;
         background-color: @nav-pills-active-link-hover-bg;
       }
     }
   }
 }
 
 
 // Stacked pills(垂直堆叠的导航)
 .nav-stacked {
   > li {
     float: none;
     + li {
       margin-top: 2px;
       margin-left: 0; // no need for this gap between nav items
     }
   }
 }
 
 
 // Nav variations
 // --------------------------------------------------
 
 // Justified nav links(自适应导航)
 // -------------------------
 .nav-justified {
   width: 100%;
 
   > li {
     float: none;
     > a {
       text-align: center;
       margin-bottom: 5px;
     }
   }
 
   > .dropdown .dropdown-menu {
     top: auto;
     left: auto;
   }
 
   @media (min-width: @screen-sm-min) {
     > li {
       display: table-cell;
       width: 1%;
       > a {
         margin-bottom: 0;
       }
     }
   }
 }
 
 // Move borders to anchors instead of bottom of list
 //
 // Mixin for adding on top the shared `.nav-justified` styles for our tabs
 .nav-tabs-justified {
   border-bottom: 0;
 
   > li > a {
     // Override margin from .nav-tabs
     margin-right: 0;
     border-radius: @border-radius-base;
   }
 
   > .active > a,
   > .active > a:hover,
   > .active > a:focus {
     border: 1px solid @nav-tabs-justified-link-border-color;
   }
 
   @media (min-width: @screen-sm-min) {
     > li > a {
       border-bottom: 1px solid @nav-tabs-justified-link-border-color;
       border-radius: @border-radius-base @border-radius-base 0 0;
     }
     > .active > a,
     > .active > a:hover,
     > .active > a:focus {
       border-bottom-color: @nav-tabs-justified-active-link-border-color;
     }
   }
 }
 
 
 // Tabbable tabs
 // -------------------------
 
 // Hide tabbable panes to start, show them when `.active`
 .tab-content {
   > .tab-pane {
     display: none;
   }
   > .active {
     display: block;
   }
 }
 
 
 // Dropdowns(二级导航)
 // -------------------------
 // Specific dropdowns
 .nav-tabs .dropdown-menu {
   // make dropdown border overlap tab border
   margin-top: -1px;
   // Remove the top rounded corners here since there is a hard edge above the menu
   .border-top-radius(0);
 }
```
##### 1.2、navs.less应用
```
 <p>标签式的导航菜单(float布局)</p>
 <ul class="nav nav-tabs">
   <li class="active"><a href="#">Home</a></li>
   <li><a href="#">SVN</a></li>
   <li><a href="#">iOS</a></li>
   <li class="disabled"><a href="#">VB.Net</a></li>
   <li><a href="#">Java</a></li>
   <li><a href="#">PHP</a></li>
 </ul>
 <p>基本的胶囊式导航菜单</p>
 <ul class="nav nav-pills">
   <li class="active"><a href="#">Home</a></li>
   <li><a href="#">SVN</a></li>
   <li><a href="#">iOS</a></li>
   <li class="disabled"><a href="#">VB.Net</a></li>
   <li><a href="#">Java</a></li>
   <li><a href="#">PHP</a></li>
 </ul>
 <p>垂直的胶囊式导航菜单</p>
 <ul class="nav nav-pills nav-stacked">
   <li class="active"><a href="#">Home</a></li>
   <li><a href="#">SVN</a></li>
   <li><a href="#">iOS</a></li>
   <li class="disabled"><a href="#">VB.Net</a></li>
   <li><a href="#">Java</a></li>
   <li><a href="#">PHP</a></li>
 </ul>
 <p>两端对齐的导航元素(小屏垂直布局，宽屏table-cell)</p>
 <ul class="nav nav-pills nav-justified">
   <li class="active"><a href="#">Home</a></li>
   <li><a href="#">SVN</a></li>
   <li><a href="#">iOS</a></li>
   <li><a href="#">VB.Net</a></li>
   <li><a href="#">Java</a></li>
   <li><a href="#">PHP</a></li>
 </ul><br>
 <ul class="nav nav-tabs nav-justified">
   <li class="active"><a href="#">Home</a></li>
   <li><a href="#">SVN</a></li>
   <li><a href="#">iOS</a></li>
   <li><a href="#">VB.Net</a></li>
   <li><a href="#">Java</a></li>
   <li><a href="#">PHP</a></li>
 </ul>
 <p>带有下拉菜单的标签</p>
 <ul class="nav nav-tabs">
   <li class="active"><a href="#">Home</a></li>
   <li><a href="#">SVN</a></li>
   <li><a href="#">iOS</a></li>
   <li><a href="#">VB.Net</a></li>
   <li class="dropdown">
     <a class="dropdown-toggle" data-toggle="dropdown" href="#">
       Java <span class="caret"></span>
     </a>
     <ul class="dropdown-menu">
       <li><a href="#">Swing</a></li>
       <li><a href="#">jMeter</a></li>
       <li><a href="#">EJB</a></li>
       <li class="divider"></li>
       <li><a href="#">分离的链接</a></li>
     </ul>
   </li>
   <li><a href="#">PHP</a></li>
 </ul>
```
#### 2、pagination.less
##### 2.1、pagination.less源码
```
 //
 // Pagination(分页导航)
 // --------------------------------------------------
 .pagination {
   display: inline-block;
   padding-left: 0;
   margin: @line-height-computed 0;
   border-radius: @border-radius-base;
 
   > li {
     display: inline; // Remove list-style and block-level defaults
     > a,
     > span {
       position: relative;
       float: left; // Collapse white-space
       padding: @padding-base-vertical @padding-base-horizontal;
       line-height: @line-height-base;
       text-decoration: none;
       color: @pagination-color;
       background-color: @pagination-bg;
       border: 1px solid @pagination-border;
       margin-left: -1px;
     }
     &:first-child {
       > a,
       > span {
         margin-left: 0;
         .border-left-radius(@border-radius-base);
       }
     }
     &:last-child {
       > a,
       > span {
         .border-right-radius(@border-radius-base);
       }
     }
   }
 
   > li > a,
   > li > span {
     &:hover,
     &:focus {
       z-index: 2;
       color: @pagination-hover-color;
       background-color: @pagination-hover-bg;
       border-color: @pagination-hover-border;
     }
   }
 
   > .active > a,
   > .active > span {
     &,
     &:hover,
     &:focus {
       z-index: 3;
       color: @pagination-active-color;
       background-color: @pagination-active-bg;
       border-color: @pagination-active-border;
       cursor: default;
     }
   }
 
   > .disabled {
     > span,
     > span:hover,
     > span:focus,
     > a,
     > a:hover,
     > a:focus {
       color: @pagination-disabled-color;
       background-color: @pagination-disabled-bg;
       border-color: @pagination-disabled-border;
       cursor: @cursor-disabled;
     }
   }
 }
 
 // Sizing
 // --------------------------------------------------
 
 // Large
 .pagination-lg {
   .pagination-size(@padding-large-vertical; @padding-large-horizontal; @font-size-large; @line-height-large; @border-radius-large);
 }
 
 // Small
 .pagination-sm {
   .pagination-size(@padding-small-vertical; @padding-small-horizontal; @font-size-small; @line-height-small; @border-radius-small);
 }
```
##### 2.2、pagination.less应用
```
 <!--pagination-lg: 分页大小-->
 <ul class="pagination pagination-lg">
   <!--display:inline 同行显示-->
   <li><a href="#">&laquo;</a></li>
   <!--active/disabled: 分页的状态-->
   <li class="active"><a href="#">1</a></li>
   <li class="disabled"><a href="#">2</a></li>
   <li><a href="#">3</a></li>
   <li><a href="#">4</a></li>
   <li><a href="#">5</a></li>
   <li><a href="#">&raquo;</a></li>
 </ul>
```