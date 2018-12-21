---
title: bootstrap 之 pager & listGroup & badge 样式
categories:
- bootstrap
---
现在开始介绍 bootstrap 的翻页(pager)、列表组(list-group)和徽章(badge) 样式。
<!--more-->
### 一、源码
#### 1、pager.less
##### 1.1、pager.less源码
```
 //
 // Pager pagination(翻页分页导航)
 // --------------------------------------------------
 .pager {
   padding-left: 0;
   margin: @line-height-computed 0;
   list-style: none;
   text-align: center;
   &:extend(.clearfix all);
   li {
     display: inline;
     > a,
     > span {
       display: inline-block;
       padding: 5px 14px;
       background-color: @pager-bg;
       border: 1px solid @pager-border;
       border-radius: @pager-border-radius;
     }
 
     > a:hover,
     > a:focus {
       text-decoration: none;
       background-color: @pager-hover-bg;
     }
   }
 
   .next {
     > a,
     > span {
       float: right;
     }
   }
 
   .previous {
     > a,
     > span {
       float: left;
     }
   }
 
   .disabled {
     > a,
     > a:hover,
     > a:focus,
     > span {
       color: @pager-disabled-color;
       background-color: @pager-bg;
       cursor: @cursor-disabled;
     }
   }
 }
```
##### 1.2、pager.less应用
```
 <!--默认的翻页-->
 <ul class="pager">
   <li><a href="#">&larr; Previous</a></li>
   <li><a href="#">Next &rarr;</a></li>
 </ul>
 <!--对齐的链接-->
 <ul class="pager">
   <!--翻页的状态-->
   <li class="previous disabled"><a href="#">&larr; Older</a></li>
   <li class="next"><a href="#">Newer &rarr;</a></li>
 </ul>
```
#### 2、list-group.less
##### 2.1、list-group.less源码
```
 //
 // List groups(列表组)
 // --------------------------------------------------
 
 
 // Base class(基础列表样式)
 //
 // Easily usable on <ul>, <ol>, or <div>.
 .list-group {
   // No need to set list-style: none; since .list-group-item is block level
   margin-bottom: 20px;
   padding-left: 0; // reset padding because ul and ol
 }
 
 
 // Individual list items(基础列表项)
 //
 // Use on `li`s or `div`s within the `.list-group` parent.
 .list-group-item {
   position: relative;
   display: block;
   padding: 10px 15px;
   // Place the border on the list items and negative margin up for better styling
   margin-bottom: -1px;
   background-color: @list-group-bg;
   border: 1px solid @list-group-border;
 
   // Round the first and last items
   &:first-child {
     .border-top-radius(@list-group-border-radius);
   }
   &:last-child {
     margin-bottom: 0;
     .border-bottom-radius(@list-group-border-radius);
   }
 }
 
 
 // Interactive list items
 //
 // Use anchor or button elements instead of `li`s or `div`s to create interactive items.
 // Includes an extra `.active` modifier class for showing selected items.
 // 带链接的列表组
 a.list-group-item,
 button.list-group-item {
   color: @list-group-link-color;
 
   .list-group-item-heading {
     color: @list-group-link-heading-color;
   }
 
   // Hover state
   &:hover,
   &:focus {
     text-decoration: none;
     color: @list-group-link-hover-color;
     background-color: @list-group-hover-bg;
   }
 }
 
 button.list-group-item {
   width: 100%;
   text-align: left;
 }
 
 // 列表项的状态设置
 .list-group-item {
   // Disabled state
   &.disabled,
   &.disabled:hover,
   &.disabled:focus {
     background-color: @list-group-disabled-bg;
     color: @list-group-disabled-color;
     cursor: @cursor-disabled;
 
     // Force color to inherit for custom content
     .list-group-item-heading {
       color: inherit;
     }
     .list-group-item-text {
       color: @list-group-disabled-text-color;
     }
   }
 
   // Active class on item itself, not parent
   &.active,
   &.active:hover,
   &.active:focus {
     z-index: 2; // Place active items above their siblings for proper border styling
     color: @list-group-active-color;
     background-color: @list-group-active-bg;
     border-color: @list-group-active-border;
 
     // Force color to inherit for custom content
     .list-group-item-heading,
     .list-group-item-heading > small,
     .list-group-item-heading > .small {
       color: inherit;
     }
     .list-group-item-text {
       color: @list-group-active-text-color;
     }
   }
 }
 
 
 // Contextual variants(多彩列表组)
 //
 // Add modifier classes to change text and background color on individual items.
 // Organizationally, this must come after the `:hover` states.
 .list-group-item-variant(success; @state-success-bg; @state-success-text);
 .list-group-item-variant(info; @state-info-bg; @state-info-text);
 .list-group-item-variant(warning; @state-warning-bg; @state-warning-text);
 .list-group-item-variant(danger; @state-danger-bg; @state-danger-text);
 
 
 // Custom content options(自定义列表组)
 //
 // Extra classes for creating well-formatted content within `.list-group-item`s.
 .list-group-item-heading {
   margin-top: 0;
   margin-bottom: 5px;
 }
 .list-group-item-text {
   margin-bottom: 0;
   line-height: 1.3;
 }
```
##### 2.2、list-group.less应用
```
 <ul class="list-group">
   <li class="list-group-item">免费域名注册</li>
   <!--列表项的状态-->
   <li class="list-group-item disabled">免费 Window 空间托管</li>
   <li class="list-group-item">图像的数量</li>
   <!--添加徽章-->
   <li class="list-group-item"><span class="badge">新</span>24*7 支持</li>
   <li class="list-group-item">每年更新成本</li>
   <!--添加链接-->
   <a href="#" class="list-group-item active">免费域名注册</a>
   <a href="#" class="list-group-item">24*7 支持</a>
   <!--添加自定义内容-->
   <a href="#" class="list-group-item">
     <h4 class="list-group-item-heading">
       免费域名注册
     </h4>
     <p class="list-group-item-text">
       您将通过网页进行免费域名注册。
     </p>
   </a>
 </ul>
```
#### 3、badges.less
##### 3.1、badges.less源码
```
 //
 // Badges(徽章)
 // --------------------------------------------------
 
 // Base class
 .badge {
   display: inline-block;
   min-width: 10px;
   padding: 3px 7px;
   font-size: @font-size-small;
   font-weight: @badge-font-weight;
   color: @badge-color;
   line-height: @badge-line-height;
   vertical-align: middle;
   white-space: nowrap;
   text-align: center;
   background-color: @badge-bg;
   border-radius: @badge-border-radius;
 
   // Empty badges collapse automatically (not available in IE8)
   &:empty {
     display: none;
   }
 
   // Quick fix for badges in buttons
   .btn & {
     position: relative;
     top: -1px;
   }
 
   .btn-xs &,
   .btn-group-xs > .btn & {
     top: 0;
     padding: 1px 5px;
   }
 
   // Hover state, but only for links
   a& {
     &:hover,
     &:focus {
       color: @badge-link-hover-color;
       text-decoration: none;
       cursor: pointer;
     }
   }
 
   // Account for badges in navs
   .list-group-item.active > &,
   .nav-pills > .active > a > & {
     color: @badge-active-color;
     background-color: @badge-active-bg;
   }
 
   .list-group-item > & {
     float: right;
   }
 
   .list-group-item > & + & {
     margin-right: 5px;
   }
 
   .nav-pills > li > a > & {
     margin-left: 3px;
   }
 }
```
##### 3.2、badges.less应用
```
 <h4>胶囊式导航中的激活状态</h4>
 <ul class="nav nav-pills">
   <li class="active">
     <a href="#">首页
       <span class="badge">42</span>
     </a>
   </li>
   <li>
     <a href="#">简介</a>
   </li>
   <li>
     <a href="#">消息
       <span class="badge">3</span>
     </a>
   </li>
 </ul>
 <br>
 <h4>列表导航中的激活状态</h4>
 <ul class="nav nav-pills nav-stacked" style="max-width: 260px;">
   <li class="active">
     <a href="#">
       <span class="badge pull-right">42</span>首页</a>
   </li>
   <li>
     <a href="#">简介</a>
   </li>
   <li>
     <a href="#">
       <!--empty时隐藏徽标，使用 :empty 选择器-->
       <span class="badge pull-right"></span>消息
     </a>
   </li>
 </ul>
```