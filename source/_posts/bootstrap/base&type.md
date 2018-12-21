---
title: bootstrap 之 bootstrap & type 样式
categories:
- bootstrap
---
现在开始介绍 bootstrap 的基础和排版(type)样式。
<!--more-->
### 一、源码
#### 1、bootstrap.less
##### 1.1、bootstrap.less 源码
```
 /*!
  * Bootstrap v3.3.7 (http://getbootstrap.com)
  * Copyright 2011-2016 Twitter, Inc.
  * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
  */
 
 // Core variables and mixins(全局变量和宏)
 @import "variables.less";
 @import "mixins.less";
 
 // Reset and dependencies(重置全局默认样式)
 @import "normalize.less";
 // 设置打印样式，使用媒体查询(@media print)
 @import "print.less";
 // 引用字体图标样式
 @import "glyphicons.less";
 
 // Core CSS(核心CSS)
 // 设置全局基础样式
 @import "scaffolding.less";
 // 排版样式
 @import "type.less";
 // 代码
 @import "code.less";
 // 网格系统
 @import "grid.less";
 // 表格
 @import "tables.less";
 // 表单
 @import "forms.less";
 // 按钮
 @import "buttons.less";
 
 // Components(组件)
 // 组件动画
 @import "component-animations.less";
 // 下拉菜单
 @import "dropdowns.less";
 // 按钮组
 @import "button-groups.less";
 @import "input-groups.less";
 // 导航
 @import "navs.less";
 // 导航条
 @import "navbar.less";
 // 面包屑式导航
 @import "breadcrumbs.less";
 // 分页导航
 @import "pagination.less";
 // 翻页分页导航
 @import "pager.less";
 // 标签
 @import "labels.less";
 // 徽章
 @import "badges.less";
 @import "jumbotron.less";
 // 缩略图
 @import "thumbnails.less";
 // 警示框
 @import "alerts.less";
 // 进度条
 @import "progress-bars.less";
 // 媒体对象
 @import "media.less";
 // 列表组
 @import "list-group.less";
 // 面板
 @import "panels.less";
 @import "responsive-embed.less";
 @import "wells.less";
 @import "close.less";
 
 // Components w/ JavaScript
 // 静态弹出框
 @import "modals.less";
 // 提示框
 @import "tooltip.less";
 // 弹出框
 @import "popovers.less";
 // 图片轮播
 @import "carousel.less";
 
 // Utility classes(工具样式)
 @import "utilities.less";
 @import "responsive-utilities.less";
```
#### 2、type.less
##### 2.1、type.less 源码
```
 //
 // Typography(排版)
 // --------------------------------------------------
 
 
 // Headings(标题)
 // -------------------------
 
 // 常常会碰到在一个标题后面紧跟着一行小的副标题(small)
 h1, h2, h3, h4, h5, h6,
 .h1, .h2, .h3, .h4, .h5, .h6 {
   font-family: @headings-font-family;
   font-weight: @headings-font-weight;
   line-height: @headings-line-height;
   color: @headings-color;
 
   small,
   .small {
     font-weight: normal;
     line-height: 1;
     color: @headings-small-color;
   }
 }
 
 h1, .h1,
 h2, .h2,
 h3, .h3 {
   margin-top: @line-height-computed;
   margin-bottom: (@line-height-computed / 2);
 
   small,
   .small {
     font-size: 65%;
   }
 }
 h4, .h4,
 h5, .h5,
 h6, .h6 {
   margin-top: (@line-height-computed / 2);
   margin-bottom: (@line-height-computed / 2);
 
   small,
   .small {
     font-size: 75%;
   }
 }
 
 h1, .h1 { font-size: @font-size-h1; }
 h2, .h2 { font-size: @font-size-h2; }
 h3, .h3 { font-size: @font-size-h3; }
 h4, .h4 { font-size: @font-size-h4; }
 h5, .h5 { font-size: @font-size-h5; }
 h6, .h6 { font-size: @font-size-h6; }
 
 
 // Body text(正文文本)
 // -------------------------
 
 p {
   margin: 0 0 (@line-height-computed / 2);
 }
 
 // 处理一个段落的文字，让其显示效果显著，强调内容
 .lead {
   margin-bottom: @line-height-computed;
   font-size: floor((@font-size-base * 1.15));
   font-weight: 300;
   line-height: 1.4;
 
   //大中型浏览器字体稍大
   @media (min-width: @screen-sm-min) {
     font-size: (@font-size-base * 1.5);
   }
 }
 
 
 // Emphasis & misc 强调突出
 // -------------------------
 
 // Ex: (12px small font / 14px base font) * 100% = about 85%
 small,
 .small {
   font-size: floor((100% * @font-size-small / @font-size-base));
 }
 
 mark,
 .mark {
   background-color: @state-warning-bg;
   padding: .2em;
 }
 
 // Alignment(对齐)
 .text-left           { text-align: left; }
 .text-right          { text-align: right; }
 .text-center         { text-align: center; }
 // 适用于长文本排版，不过目前两端对齐在各浏览器下解析各有不同，特别是应用于中文文本的时候，所以项目中慎用
 .text-justify        { text-align: justify; }
 .text-nowrap         { white-space: nowrap; }
 
 // Transformation(文本大小写转换)
 .text-lowercase      { text-transform: lowercase; }
 .text-uppercase      { text-transform: uppercase; }
 .text-capitalize     { text-transform: capitalize; }
 
 // Contextual colors(文本颜色)
 .text-muted {  //提示
   color: @text-muted;
 }
 .text-primary {  //主要
   .text-emphasis-variant(@brand-primary);
 }
 .text-success { //成功
   .text-emphasis-variant(@state-success-text);
 }
 .text-info { //通知信息
   .text-emphasis-variant(@state-info-text);
 }
 .text-warning { //警告
   .text-emphasis-variant(@state-warning-text);
 }
 .text-danger {  //危险
   .text-emphasis-variant(@state-danger-text);
 }
 
 // Contextual backgrounds(文本背景)
 // For now(目前，暂时) we'll leave these alongside the text classes until v4 when we can
 // safely shift things around (per SemVer rules).
 .bg-primary {
   // Given the contrast(对比) here, this is the only class to have its color inverted
   // automatically.
   color: #fff;
   .bg-variant(@brand-primary);
 }
 .bg-success {
   .bg-variant(@state-success-bg);
 }
 .bg-info {
   .bg-variant(@state-info-bg);
 }
 .bg-warning {
   .bg-variant(@state-warning-bg);
 }
 .bg-danger {
   .bg-variant(@state-danger-bg);
 }
 
 
 // Page header(页面标题)
 // -------------------------
 
 .page-header {
   padding-bottom: ((@line-height-computed / 2) - 1);
   margin: (@line-height-computed * 2) 0 @line-height-computed;
   border-bottom: 1px solid @page-header-border-color;
 }
 
 
 // Lists(列表)
 // -------------------------
 
 // Unordered and Ordered lists
 ul,
 ol {
   margin-top: 0;
   margin-bottom: (@line-height-computed / 2);
   ul,
   ol {
     margin-bottom: 0;
   }
 }
 
 // List options
 
 // Unstyled keeps list items block level, just removes default browser padding and list-style(去点列表)
 .list-unstyled {
   padding-left: 0;
   list-style: none;
 }
 
 // Inline turns list items into inline-block() (内联列表，为制作水平导航而生)
 .list-inline {
   .list-unstyled();
   margin-left: -5px;
 
   > li {
     display: inline-block;
     padding-left: 5px;
     padding-right: 5px;
   }
 }
 
 // Description Lists (定义列表)
 dl {
   margin-top: 0; // Remove browser default
   margin-bottom: @line-height-computed;
 }
 dt,
 dd {
   line-height: @line-height-base;
 }
 dt {
   font-weight: bold;
 }
 dd {
   margin-left: 0; // Undo browser default
 }
 
 // Horizontal description lists (水平定义列表)
 //
 // Defaults to being stacked without any of the below styles applied, until the
 // grid breakpoint is reached (default of ~768px).
 
 .dl-horizontal {
   dd {
     &:extend(.clearfix all); // Clear the floated `dt` if an empty `dd` is present
   }
 
   @media (min-width: @dl-horizontal-breakpoint) {
     dt {
       float: left;
       width: (@dl-horizontal-offset - 20);
       clear: left;
       text-align: right;
       .text-overflow();
     }
     dd {
       margin-left: @dl-horizontal-offset;
     }
   }
 }
 
 
 // Misc
 // -------------------------
 
 // Abbreviations and acronyms(缩略词)
 abbr[title],
 // Add data-* attribute to help out our tooltip plugin, per https://github.com/twbs/bootstrap/issues/5257
 abbr[data-original-title] {
   cursor: help;
   border-bottom: 1px dotted @abbr-border-color;
 }
 // 词首字母缩略词
 .initialism {
   font-size: 90%;
   .text-uppercase();
 }
 
 // Blockquotes(引用)
 blockquote {
   padding: (@line-height-computed / 2) @line-height-computed;
   margin: 0 0 @line-height-computed;
   font-size: @blockquote-font-size;
   border-left: 5px solid @blockquote-border-color;
 
   p,
   ul,
   ol {
     &:last-child {
       margin-bottom: 0;
     }
   }
 
   // Note: Deprecated small and .small as of v3.1.0
   // Context: https://github.com/twbs/bootstrap/issues/11660
   footer,
   small,
   .small {
     display: block;
     font-size: 80%; // back to default font-size
     line-height: @line-height-base;
     color: @blockquote-small-color;
 
     &:before {
       content: '\2014 \00A0'; // em dash, nbsp
     }
   }
 }
 
 // Opposite alignment of blockquote
 //
 // Heads up: `blockquote.pull-right` has been deprecated as of v3.1.0.
 .blockquote-reverse,
 blockquote.pull-right {
   padding-right: 15px;
   padding-left: 0;
   border-right: 5px solid @blockquote-border-color;
   border-left: 0;
   text-align: right;
 
   // Account for citation
   footer,
   small,
   .small {
     &:before { content: ''; }
     &:after {
       content: '\00A0 \2014'; // nbsp, em dash
     }
   }
 }
 
 // Addresses
 address {
   margin-bottom: @line-height-computed;
   font-style: normal;
   line-height: @line-height-base;
 }
```
##### 2.2、type.less 应用
```
 <!--标题&内联子标题-->
 <h1>我是标题1 h1. <small>我是副标题1 h1</small></h1>
 <h2>我是标题2 h2. <small>我是副标题2 h2</small></h2>
 <h3>我是标题3 h3. <small>我是副标题3 h3</small></h3>
 <h4>我是标题4 h4. <small>我是副标题4 h4</small></h4>
 <h5>我是标题5 h5. <small>我是副标题5 h5</small></h5>
 <h6>我是标题6 h6. <small>我是副标题6 h6</small></h6>
 <!--引导主体副本-->
 <h2>引导主体副本</h2>
 <p class="lead">这是一个演示引导主体副本用法的实例。这是一个演示引导主体副本用法的实例。。</p>
 <!--强调-->
 <small>本行内容是在标签内</small><br>
 <strong>本行内容是在标签内</strong><br>
 <em>本行内容是在标签内，并呈现为斜体</em><br>
 <p class="text-left">向左对齐文本</p>
 <p class="text-center">居中对齐文本</p>
 <p class="text-right">向右对齐文本</p>
 <!--文本颜色，同文本背景-->
 <p class="text-muted bg-success">本行内容是减弱的</p>
 <p class="text-primary">本行内容带有一个 primary class</p>
 <p class="text-success">本行内容带有一个 success class</p>
 <p class="text-info">本行内容带有一个 info class</p>
 <p class="text-warning">本行内容带有一个 warning class</p>
 <p class="text-danger">本行内容带有一个 danger class</p>
 <!--缩写-->
 <abbr title="World Wide Web">WWW</abbr><br>
 <abbr title="Real Simple Syndication" class="initialism">RSS</abbr>
 <!--地址-->
 <address>
   <strong>Some Company, Inc.</strong><br>
   007 street<br>
   Some City, State XXXXX<br>
   <abbr title="Phone">P:</abbr> (123) 456-7890
 </address>
 <address>
   <strong>Full Name</strong><br>
   <a href="mailto:#">mailto@somedomain.com</a>
 </address>
 <!--引用-->
 <blockquote>
   <p>
     这是一个默认的引用实例。这是一个默认的引用实例。这是一个默认的引用实例。这是一个默认的引用实例。
   </p>
 </blockquote>
 <blockquote>
   这是一个带有源标题的引用。
   <small>Someone famous in <cite title="Source Title">Source Title</cite></small>
 </blockquote>
 <blockquote class="pull-right">
   这是一个向右对齐的引用。
   <small>Someone famous in <cite title="Source Title">Source Title</cite></small>
 </blockquote>
 <!--列表-->
 <h4>有序列表</h4>
 <ol>
   <li>Item 1</li>
   <li>Item 2</li>
   <li>Item 3</li>
   <li>Item 4</li>
 </ol>
 <h4>无序列表</h4>
 <ul>
   <li>Item 1</li>
   <li>Item 2</li>
   <li>Item 3</li>
   <li>Item 4</li>
 </ul>
 <h4>未定义样式列表</h4>
 <ul class="list-unstyled">
   <li>Item 1</li>
   <li>Item 2</li>
   <li>Item 3</li>
   <li>Item 4</li>
 </ul>
 <h4>内联列表</h4>
 <ul class="list-inline">
   <li>Item 1</li>
   <li>Item 2</li>
   <li>Item 3</li>
   <li>Item 4</li>
 </ul>
 <h4>定义列表</h4>
 <dl>
   <dt>Description 1</dt>
   <dd>Item 1</dd>
   <dt>Description 2</dt>
   <dd>Item 2</dd>
 </dl>
 <h4>水平的定义列表</h4>
 <dl class="dl-horizontal">
   <dt>Description 1</dt>
   <dd>Item 1</dd>
   <dt>Description 2</dt>
   <dd>Item 2</dd>
 </dl>
```