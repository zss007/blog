---
title: bootstrap 之 jumbotron & thumbnail & progressBar 样式
categories:
- bootstrap
---
现在开始介绍 bootstrap 的超大屏幕(jumbotron)、缩略图(thumbnail)和徽章(badge)样式。
<!--more-->
### 一、源码
#### 1、jumbotron.less
##### 1.1、jumbotron.less源码
```
 //
 // Jumbotron
 // --------------------------------------------------
 
 
 .jumbotron {
   padding-top:    @jumbotron-padding;
   padding-bottom: @jumbotron-padding;
   margin-bottom: @jumbotron-padding;
   color: @jumbotron-color;
   background-color: @jumbotron-bg;
 
   h1,
   .h1 {
     color: @jumbotron-heading-color;
   }
 
   p {
     margin-bottom: (@jumbotron-padding / 2);
     font-size: @jumbotron-font-size;
     font-weight: 200;
   }
 
   > hr {
     border-top-color: darken(@jumbotron-bg, 10%);
   }
 
   .container &,
   .container-fluid & {
     border-radius: @border-radius-large; // Only round corners at higher resolutions if contained in a container
     padding-left:  (@grid-gutter-width / 2);
     padding-right: (@grid-gutter-width / 2);
   }
 
   .container {
     max-width: 100%;
   }
 
   @media screen and (min-width: @screen-sm-min) {
     padding-top:    (@jumbotron-padding * 1.6);
     padding-bottom: (@jumbotron-padding * 1.6);
 
     .container &,
     .container-fluid & {
       padding-left:  (@jumbotron-padding * 2);
       padding-right: (@jumbotron-padding * 2);
     }
 
     h1,
     .h1 {
       font-size: @jumbotron-heading-font-size;
     }
   }
 }
```
##### 1.2、jumbotron.less应用
```
 <!--超大屏幕: h1更大，font-weight被减为200-->
 <div class="container">
   <div class="jumbotron">
     <h1>欢迎登陆页面！</h1>
     <p>这是一个超大屏幕（Jumbotron）的实例。</p>
     <p><a class="btn btn-primary btn-lg" role="button">
       学习更多</a>
     </p>
   </div>
 </div>
```
#### 2、thumbnails.less
##### 2.1、thumbnails.less源码
```
 //
 // Thumbnails(更为复杂的缩略图，相对于scaffolding.less的.img-thumbnail来说，可以配合bootstrap的栅格系统显示标题、描述内容、按钮等)
 // --------------------------------------------------
 
 
 // Mixin and adjust the regular image class
 .thumbnail {
   display: block;
   padding: @thumbnail-padding;
   margin-bottom: @line-height-computed;
   line-height: @line-height-base;
   background-color: @thumbnail-bg;
   border: 1px solid @thumbnail-border;
   border-radius: @thumbnail-border-radius;
   .transition(border .2s ease-in-out);
 
   > img,
   a > img {
     &:extend(.img-responsive);
     margin-left: auto;
     margin-right: auto;
   }
 
   // Add a hover state for linked versions only
   a&:hover,
   a&:focus,
   a&.active {
     border-color: @link-color;
   }
 
   // Image captions
   .caption {
     padding: @thumbnail-caption-padding;
     color: @thumbnail-caption-color;
   }
 }
```
##### 2.2、thumbnails.less应用
```
 <div class="row">
   <div class="col-sm-6 col-md-3">
     <div class="thumbnail">
       <img src="/assets/bootstrap/kittens.jpg" alt="通用的占位符缩略图">
       <div class="caption">
         <h3>缩略图标签</h3>
         <p>一些示例文本。一些示例文本。</p>
         <p>
           <a href="#" class="btn btn-primary" role="button">
             按钮
           </a>
           <a href="#" class="btn btn-default" role="button">
             按钮
           </a>
         </p>
       </div>
     </div>
   </div>
   <div class="col-sm-6 col-md-3">
     <div class="thumbnail">
       <img src="/assets/bootstrap/kittens.jpg" alt="通用的占位符缩略图">
       <div class="caption">
         <h3>缩略图标签</h3>
         <p>一些示例文本。一些示例文本。</p>
         <p>
           <a href="#" class="btn btn-primary" role="button">
             按钮
           </a>
           <a href="#" class="btn btn-default" role="button">
             按钮
           </a>
         </p>
       </div>
     </div>
   </div>
 </div>
```
#### 3、progress-bars.less
##### 3.1、progress-bars.less源码
```
 //
 // Progress bars
 // --------------------------------------------------
 
 
 // Bar animations(进度条动画)
 // -------------------------
 // WebKit
 @-webkit-keyframes progress-bar-stripes {
   from  { background-position: 40px 0; }
   to    { background-position: 0 0; }
 }
 
 // Spec and IE10+
 @keyframes progress-bar-stripes {
   from  { background-position: 40px 0; }
   to    { background-position: 0 0; }
 }
 
 
 // Bar itself
 // -------------------------
 
 // Outer container(外容器)
 .progress {
   overflow: hidden;
   height: @line-height-computed;
   margin-bottom: @line-height-computed;
   background-color: @progress-bg;
   border-radius: @progress-border-radius;
   .box-shadow(inset 0 1px 2px rgba(0,0,0,.1));
 }
 
 // Bar of progress(进度条)
 .progress-bar {
   float: left;
   width: 0%;
   height: 100%;
   font-size: @font-size-small;
   line-height: @line-height-computed;
   color: @progress-bar-color;
   text-align: center;
   background-color: @progress-bar-bg;
   .box-shadow(inset 0 -1px 0 rgba(0,0,0,.15));
   .transition(width .6s ease);
 }
 
 // Striped bars(条纹进度条)
 //
 // `.progress-striped .progress-bar` is deprecated as of v3.2.0 in favor of the
 // `.progress-bar-striped` class, which you just add to an existing
 // `.progress-bar`.
 .progress-striped .progress-bar,
 .progress-bar-striped {
   #gradient > .striped();
   background-size: 40px 40px;
 }
 
 // Call animation for the active one(动态条纹进度条)
 //
 // `.progress.active .progress-bar` is deprecated as of v3.2.0 in favor of the
 // `.progress-bar.active` approach.
 .progress.active .progress-bar,
 .progress-bar.active {
   .animation(progress-bar-stripes 2s linear infinite);
 }
 
 
 // Variations(彩色进度条)
 // -------------------------
 .progress-bar-success {
   .progress-bar-variant(@progress-bar-success-bg);
 }
 
 .progress-bar-info {
   .progress-bar-variant(@progress-bar-info-bg);
 }
 
 .progress-bar-warning {
   .progress-bar-variant(@progress-bar-warning-bg);
 }
 
 .progress-bar-danger {
   .progress-bar-variant(@progress-bar-danger-bg);
 }
```
##### 3.2、progress-bars.less应用
```
 <!--进度条-->
 <div class="progress">
   <!--进度条状态-->
   <div class="progress-bar progress-bar-success" role="progressbar"
        aria-valuenow="60" aria-valuemin="0" aria-valuemax="100"
        style="width: 90%;">
     <span class="sr-only">90% 完成（成功）</span>
   </div>
 </div>
 <div class="progress">
   <div class="progress-bar progress-bar-info" role="progressbar"
        aria-valuenow="60" aria-valuemin="0" aria-valuemax="100"
        style="width: 30%;">
     <span class="sr-only">30% 完成（信息）</span>
   </div>
 </div>
 <!--条纹进度条-->
 <div class="progress progress-striped">
   <div class="progress-bar progress-bar-warning" role="progressbar"
        aria-valuenow="60" aria-valuemin="0" aria-valuemax="100"
        style="width: 20%;">
     <span class="sr-only">20% 完成（警告）</span>
   </div>
 </div>
 <!--动画的进度条: 使线性渐变具有动画效果-->
 <div class="progress progress-striped active">
   <div class="progress-bar progress-bar-danger" role="progressbar"
        aria-valuenow="60" aria-valuemin="0" aria-valuemax="100"
        style="width: 10%;">
     <span class="sr-only">10% 完成（危险）</span>
   </div>
 </div>
 <!--堆叠的进度条-->
 <div class="progress">
   <div class="progress-bar progress-bar-success" role="progressbar"
        aria-valuenow="60" aria-valuemin="0" aria-valuemax="100"
        style="width: 40%;">
     <span class="sr-only">40% 完成</span>
   </div>
   <div class="progress-bar progress-bar-info" role="progressbar"
        aria-valuenow="60" aria-valuemin="0" aria-valuemax="100"
        style="width: 30%;">
     <span class="sr-only">30% 完成（信息）</span>
   </div>
   <div class="progress-bar progress-bar-warning" role="progressbar"
        aria-valuenow="60" aria-valuemin="0" aria-valuemax="100"
        style="width: 20%;">
     <span class="sr-only">20% 完成（警告）</span>
   </div>
 </div>
```