---
title: bootstrap 之 media & close & well 样式
categories:
- bootstrap
---
现在开始介绍 bootstrap 的多媒体(media)、关闭图标(close)和凹陷容器(well)样式。
<!--more-->
### 一、源码
#### 1、media.less
##### 1.1、media.less 源码
```
 .media {
   // Proper spacing between instances of .media
   margin-top: 15px;
 
   &:first-child {
     margin-top: 0;
   }
 }
 
 .media,
 .media-body {
   zoom: 1;
   overflow: hidden;
 }
 
 .media-body {
   width: 10000px;
 }
 
 .media-object {
   display: block;
 
   // Fix collapse in webkit from max-width: 100% and display: table-cell.
   &.img-thumbnail {
     max-width: none;
   }
 }
 
 .media-right,
 .media > .pull-right {
   padding-left: 10px;
 }
 
 .media-left,
 .media > .pull-left {
   padding-right: 10px;
 }
 
 .media-left,
 .media-right,
 .media-body {
   display: table-cell;
   vertical-align: top;
 }
 
 .media-middle {
   vertical-align: middle;
 }
 
 .media-bottom {
   vertical-align: bottom;
 }
 
 // Reset margins on headings for tighter default spacing
 .media-heading {
   margin-top: 0;
   margin-bottom: 5px;
 }
 
 // Media list variation
 //
 // Undo default ul/ol styles
 .media-list {
   padding-left: 0;
   list-style: none;
 }
```
##### 1.2、media.less 应用
```
 <ul class="media-list">
   <li class="media">
     <!--使用table布局同行显示-->
     <a class="media-left" href="#">
       <img src="/assets/bootstrap/kittens.jpg" width="64px" alt="通用的占位符图像">
     </a>
     <div class="media-body">
       <h4 class="media-heading">媒体标题</h4>
       <p>这是一些示例文本。这是一些示例文本。
         这是一些示例文本。这是一些示例文本。
         这是一些示例文本。这是一些示例文本。
         这是一些示例文本。这是一些示例文本。
         这是一些示例文本。这是一些示例文本。</p>
       <!-- 嵌套的媒体对象 -->
       <div class="media">
         <a class="media-left" href="#">
           <img src="/assets/bootstrap/kittens.jpg" width="64px" alt="通用的占位符图像">
         </a>
         <div class="media-body">
           <h4 class="media-heading">嵌套的媒体标题</h4>
           这是一些示例文本。这是一些示例文本。
           这是一些示例文本。这是一些示例文本。
           这是一些示例文本。这是一些示例文本。
           这是一些示例文本。这是一些示例文本。
           这是一些示例文本。这是一些示例文本。
           <!-- 嵌套的媒体对象 -->
           <div class="media">
             <a class="media-left" href="#">
               <img src="/assets/bootstrap/kittens.jpg" width="64px" alt="通用的占位符图像">
             </a>
             <div class="media-body">
               <h4 class="media-heading">嵌套的媒体标题</h4>
               这是一些示例文本。这是一些示例文本。
               这是一些示例文本。这是一些示例文本。
               这是一些示例文本。这是一些示例文本。
               这是一些示例文本。这是一些示例文本。
               这是一些示例文本。这是一些示例文本。
             </div>
           </div>
         </div>
       </div>
       <!-- 嵌套的媒体对象 -->
       <div class="media">
         <a class="media-left" href="#">
           <img src="/assets/bootstrap/kittens.jpg" width="64px" alt="通用的占位符图像">
         </a>
         <div class="media-body">
           <h4 class="media-heading">嵌套的媒体标题</h4>
           这是一些示例文本。这是一些示例文本。
           这是一些示例文本。这是一些示例文本。
           这是一些示例文本。这是一些示例文本。
           这是一些示例文本。这是一些示例文本。
           这是一些示例文本。这是一些示例文本。
         </div>
       </div>
     </div>
   </li>
   <li class="media">
     <a class="media-right" href="#">
       <img src="/assets/bootstrap/kittens.jpg" width="64px" alt="通用的占位符图像">
     </a>
     <div class="media-body">
       <h4 class="media-heading">媒体标题</h4>
       这是一些示例文本。这是一些示例文本。
       这是一些示例文本。这是一些示例文本。
       这是一些示例文本。这是一些示例文本。
       这是一些示例文本。这是一些示例文本。
       这是一些示例文本。这是一些示例文本。
     </div>
   </li>
 </ul>
```
#### 2、close.less
##### 2.1、close.less 源码
```
 //
 // Close icons
 // --------------------------------------------------
 
 
 .close {
   float: right;
   font-size: (@font-size-base * 1.5);
   font-weight: @close-font-weight;
   line-height: 1;
   color: @close-color;
   text-shadow: @close-text-shadow;
   .opacity(.2);
 
   &:hover,
   &:focus {
     color: @close-color;
     text-decoration: none;
     cursor: pointer;
     .opacity(.5);
   }
 
   // Additional properties for button version
   // iOS requires the button element instead of an anchor tag.
   // If you want the anchor version, it requires `href="#"`.
   // See https://developer.mozilla.org/en-US/docs/Web/Events/click#Safari_Mobile
   button& {
     padding: 0;
     cursor: pointer;
     background: transparent;
     border: 0;
     -webkit-appearance: none;
   }
 }
```
##### 2.2、close.less 应用
```
 <span class="close">x</span>
```
#### 3、wells.less.less
##### 3.1、wells.less 源码
```
 //
 // Wells
 // --------------------------------------------------
 
 
 // Base class
 .well {
   min-height: 20px;
   padding: 19px;
   margin-bottom: 20px;
   background-color: @well-bg;
   border: 1px solid @well-border;
   border-radius: @border-radius-base;
   .box-shadow(inset 0 1px 1px rgba(0,0,0,.05));
   blockquote {
     border-color: #ddd;
     border-color: rgba(0,0,0,.15);
   }
 }
 
 // Sizes
 .well-lg {
   padding: 24px;
   border-radius: @border-radius-large;
 }
 .well-sm {
   padding: 9px;
   border-radius: @border-radius-small;
 }
```
##### 3.2、wells.less 应用
```
 <!--box-shadow: inset 达到凹陷的效果-->
 <div class="well">您好，我在 Well 中！</div>
 <div class="well well-lg">您好，我在大的 Well 中！</div>
 <div class="well well-sm">您好，我在小的 Well 中！</div>
```