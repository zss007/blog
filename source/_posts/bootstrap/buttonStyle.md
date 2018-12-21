---
title: bootstrap 之 button 样式
categories:
- bootstrap
---
现在开始介绍 bootstrap 的 button 模块样式，包括 buttons.less、button-groups.less。
<!--more-->
### 一、源码
#### 1、buttons.less
##### 1.1、buttons.less源码
```
 //
 // Buttons(按钮)
 // --------------------------------------------------
 
 // Base styles(基础样式)
 // --------------------------------------------------
 .btn {
   display: inline-block;
   margin-bottom: 0; // For input.btn
   font-weight: @btn-font-weight;
   text-align: center;
   vertical-align: middle;
   touch-action: manipulation;
   cursor: pointer;
   background-image: none; // Reset unusual Firefox-on-Android default style; see https://github.com/necolas/normalize.css/issues/214
   border: 1px solid transparent;
   white-space: nowrap;
   .button-size(@padding-base-vertical; @padding-base-horizontal; @font-size-base; @line-height-base; @btn-border-radius-base);
   .user-select(none);
 
   &,
   &:active,
   &.active {
     &:focus,
     &.focus {
       .tab-focus();
     }
   }
 
   &:hover,
   &:focus,
   &.focus {
     color: @btn-default-color;
     text-decoration: none;
   }
 
   &:active,
   &.active {
     outline: 0;
     background-image: none;
     .box-shadow(inset 0 3px 5px rgba(0,0,0,.125));
   }
 
   &.disabled,
   &[disabled],
   fieldset[disabled] & {
     cursor: @cursor-disabled;
     .opacity(.65);
     .box-shadow(none);
   }
 
   a& {
     &.disabled,
     fieldset[disabled] & {
       pointer-events: none; // Future-proof disabling of clicks on `<a>` elements
     }
   }
 }
 
 // Alternate buttons(交互式按钮)
 // --------------------------------------------------
 .btn-default {
   .button-variant(@btn-default-color; @btn-default-bg; @btn-default-border);
 }
 .btn-primary {
   .button-variant(@btn-primary-color; @btn-primary-bg; @btn-primary-border);
 }
 // Success appears as green
 .btn-success {
   .button-variant(@btn-success-color; @btn-success-bg; @btn-success-border);
 }
 // Info appears as blue-green
 .btn-info {
   .button-variant(@btn-info-color; @btn-info-bg; @btn-info-border);
 }
 // Warning appears as orange
 .btn-warning {
   .button-variant(@btn-warning-color; @btn-warning-bg; @btn-warning-border);
 }
 // Danger and error appear as red
 .btn-danger {
   .button-variant(@btn-danger-color; @btn-danger-bg; @btn-danger-border);
 }
 
 // Link buttons(链接按钮)
 // -------------------------
 // Make a button look and behave like a link
 .btn-link {
   color: @link-color;
   font-weight: normal;
   border-radius: 0;
 
   &,
   &:active,
   &.active,
   &[disabled],
   fieldset[disabled] & {
     background-color: transparent;
     .box-shadow(none);
   }
   &,
   &:hover,
   &:focus,
   &:active {
     border-color: transparent;
   }
   &:hover,
   &:focus {
     color: @link-hover-color;
     text-decoration: @link-hover-decoration;
     background-color: transparent;
   }
   &[disabled],
   fieldset[disabled] & {
     &:hover,
     &:focus {
       color: @btn-link-disabled-color;
       text-decoration: none;
     }
   }
 }
 
 // Button Sizes(按钮大小)
 // --------------------------------------------------
 .btn-lg {
   // line-height: ensure even-numbered height of button next to large input
   .button-size(@padding-large-vertical; @padding-large-horizontal; @font-size-large; @line-height-large; @btn-border-radius-large);
 }
 .btn-sm {
   // line-height: ensure proper height of button next to small input
   .button-size(@padding-small-vertical; @padding-small-horizontal; @font-size-small; @line-height-small; @btn-border-radius-small);
 }
 .btn-xs {
   .button-size(@padding-xs-vertical; @padding-xs-horizontal; @font-size-small; @line-height-small; @btn-border-radius-small);
 }
 
 // Block button(块状按钮)
 // --------------------------------------------------
 .btn-block {
   display: block;
   width: 100%;
 }
 
 // Vertically space out multiple block buttons
 .btn-block + .btn-block {
   margin-top: 5px;
 }
 
 // Specificity overrides
 input[type="submit"],
 input[type="reset"],
 input[type="button"] {
   &.btn-block {
     width: 100%;
   }
 }
```
##### 1.2、buttons.less应用
```
 // 按钮动作 & 按钮大小 & 按钮状态
 <!-- 标准的按钮 -->
 <button type="button" class="btn btn-default btn-lg active">默认大激活按钮</button>
 <!-- 提供额外的视觉效果，标识一组按钮中的原始动作 -->
 <button type="button" class="btn btn-primary disabled">原始默认大小禁用按钮</button>
 <!-- 表示一个成功的或积极的动作 -->
 <button type="button" class="btn btn-success btn-sm">成功小按钮</button>
 <!-- 信息警告消息的上下文按钮 -->
 <button type="button" class="btn btn-info btn-xs">信息特小按钮</button>
 <!-- 表示应谨慎采取的动作 -->
 <button type="button" class="btn btn-warning btn-lg btn-block">警告块级按钮</button>
 <!-- 表示一个危险的或潜在的负面动作 -->
 <button type="button" class="btn btn-danger">危险默认大小按钮</button>
 <!-- 并不强调是一个按钮，看起来像一个链接，但同时保持按钮的行为 -->
 <button type="button" class="btn btn-link">链接默认大小按钮</button>
```
#### 2、button-groups.less
##### 2.1、button-groups.less源码
```
 //
 // Button groups(按钮组)
 // --------------------------------------------------
 
 // Make the div behave like a button(按钮组)
 .btn-group,
 .btn-group-vertical {
   position: relative;
   display: inline-block;
   vertical-align: middle; // match .btn alignment given font-size hack above
   > .btn {
     position: relative;
     float: left;
     // Bring the "active" button to the front
     &:hover,
     &:focus,
     &:active,
     &.active {
       z-index: 2;
     }
   }
 }
 
 // Prevent double borders when buttons are next to each other(去除按钮组间的边框)
 .btn-group {
   .btn + .btn,
   .btn + .btn-group,
   .btn-group + .btn,
   .btn-group + .btn-group {
     margin-left: -1px;
   }
 }
 
 // Optional: Group multiple button groups together for a toolbar(工具栏，按钮分组的父容器)
 .btn-toolbar {
   margin-left: -5px; // Offset the first child's margin
   &:extend(.clearfix all);
 
   .btn,
   .btn-group,
   .input-group {
     float: left;
   }
   > .btn,
   > .btn-group,
   > .input-group {
     margin-left: 5px;
   }
 }
 
 // 去除圆角，生成按钮组圆角效果
 .btn-group > .btn:not(:first-child):not(:last-child):not(.dropdown-toggle) {
   border-radius: 0;
 }
 // Set corners individual because sometimes a single button can be in a .btn-group and we need :first-child and :last-child to both match
 .btn-group > .btn:first-child {
   margin-left: 0;
   &:not(:last-child):not(.dropdown-toggle) {
     .border-right-radius(0);
   }
 }
 // Need .dropdown-toggle since :last-child doesn't apply, given that a .dropdown-menu is used immediately after it
 .btn-group > .btn:last-child:not(:first-child),
 .btn-group > .dropdown-toggle:not(:first-child) {
   .border-left-radius(0);
 }
 
 // Custom edits for including btn-groups within btn-groups (useful for including dropdown buttons within a btn-group)
 .btn-group > .btn-group {
   float: left;
 }
 .btn-group > .btn-group:not(:first-child):not(:last-child) > .btn {
   border-radius: 0;
 }
 .btn-group > .btn-group:first-child:not(:last-child) {
   > .btn:last-child,
   > .dropdown-toggle {
     .border-right-radius(0);
   }
 }
 .btn-group > .btn-group:last-child:not(:first-child) > .btn:first-child {
   .border-left-radius(0);
 }
 
 // On active and open, don't show outline
 .btn-group .dropdown-toggle:active,
 .btn-group.open .dropdown-toggle {
   outline: 0;
 }
 
 
 // Sizing(按钮组大小设置)
 //
 // Remix the default button sizing classes into new ones for easier manipulation.
 .btn-group-xs > .btn { &:extend(.btn-xs); }
 .btn-group-sm > .btn { &:extend(.btn-sm); }
 .btn-group-lg > .btn { &:extend(.btn-lg); }
 
 
 // Split button dropdowns(嵌套分组)
 // ----------------------
 // Give the line between buttons some depth
 .btn-group > .btn + .dropdown-toggle {
   padding-left: 8px;
   padding-right: 8px;
 }
 .btn-group > .btn-lg + .dropdown-toggle {
   padding-left: 12px;
   padding-right: 12px;
 }
 
 // The clickable button for toggling the menu
 // Remove the gradient and set the same inset shadow as the :active state
 .btn-group.open .dropdown-toggle {
   .box-shadow(inset 0 3px 5px rgba(0,0,0,.125));
 
   // Show no shadow for `.btn-link` since it has no other button styles.
   &.btn-link {
     .box-shadow(none);
   }
 }
 
 
 // Reposition the caret
 .btn .caret {
   margin-left: 0;
 }
 // Carets in other button sizes
 .btn-lg .caret {
   border-width: @caret-width-large @caret-width-large 0;
   border-bottom-width: 0;
 }
 // Upside down carets for .dropup
 .dropup .btn-lg .caret {
   border-width: 0 @caret-width-large @caret-width-large;
 }
 
 
 // Vertical button groups(垂直按钮组)
 // ----------------------
 .btn-group-vertical {
   > .btn,
   > .btn-group,
   > .btn-group > .btn {
     display: block;
     float: none;
     width: 100%;
     max-width: 100%;
   }
 
   // Clear floats so dropdown menus can be properly placed
   > .btn-group {
     &:extend(.clearfix all);
     > .btn {
       float: none;
     }
   }
 
   > .btn + .btn,
   > .btn + .btn-group,
   > .btn-group + .btn,
   > .btn-group + .btn-group {
     margin-top: -1px;
     margin-left: 0;
   }
 }
 .btn-group-vertical > .btn {
   &:not(:first-child):not(:last-child) {
     border-radius: 0;
   }
   &:first-child:not(:last-child) {
     .border-top-radius(@btn-border-radius-base);
     .border-bottom-radius(0);
   }
   &:last-child:not(:first-child) {
     .border-top-radius(0);
     .border-bottom-radius(@btn-border-radius-base);
   }
 }
 .btn-group-vertical > .btn-group:not(:first-child):not(:last-child) > .btn {
   border-radius: 0;
 }
 .btn-group-vertical > .btn-group:first-child:not(:last-child) {
   > .btn:last-child,
   > .dropdown-toggle {
     .border-bottom-radius(0);
   }
 }
 .btn-group-vertical > .btn-group:last-child:not(:first-child) > .btn:first-child {
   .border-top-radius(0);
 }
 
 
 // Justified button groups(等分按钮)
 // ----------------------
 .btn-group-justified {
   display: table;
   width: 100%;
   table-layout: fixed;
   border-collapse: separate;
   > .btn,
   > .btn-group {
     float: none;
     display: table-cell;
     width: 1%;
   }
   > .btn-group .btn {
     width: 100%;
   }
 
   > .btn-group .dropdown-menu {
     left: auto;
   }
 }
 
 
 // Checkbox and radio options
 //
 // In order to support the browser's form validation feedback, powered by the
 // `required` attribute, we have to "hide" the inputs via `clip`. We cannot use
 // `display: none;` or `visibility: hidden;` as that also hides the popover.
 // Simply visually hiding the inputs via `opacity` would leave them clickable in
 // certain cases which is prevented by using `clip` and `pointer-events`.
 // This way, we ensure a DOM element is visible to position the popover from.
 //
 // See https://github.com/twbs/bootstrap/pull/12794 and
 // https://github.com/twbs/bootstrap/pull/14559 for more information.
 [data-toggle="buttons"] {
   > .btn,
   > .btn-group > .btn {
     input[type="radio"],
     input[type="checkbox"] {
       position: absolute;
       clip: rect(0,0,0,0);
       pointer-events: none;
     }
   }
 }
```
##### 2.2、button-groups.less应用
```
 <!--btn-toolbar: 按钮工具栏-->
 <div class="btn-toolbar" role="toolbar">
   <!--大按钮组-->
   <div class="btn-group btn-group-lg">
     <button type="button" class="btn btn-default">按钮 1</button>
     <button type="button" class="btn btn-default">按钮 2</button>
   </div>
   <!--小按钮组-->
   <div class="btn-group btn-group-sm">
     <button type="button" class="btn btn-default">按钮 3</button>
     <button type="button" class="btn btn-default">按钮 4</button>
   </div>
   <!--特小按钮组-->
   <div class="btn-group btn-group-xs">
     <button type="button" class="btn btn-default">按钮 5</button>
     <button type="button" class="btn btn-default">按钮 6</button>
   </div>
   <!--垂直按钮组-->
   <div class="btn-group-vertical">
     <button type="button" class="btn btn-default">按钮 7</button>
     <button type="button" class="btn btn-default">按钮 8</button>
   </div>
   <!--嵌套按钮组-->
   <div class="btn-group">
     <button type="button" class="btn btn-default">按钮 9</button>
     <button type="button" class="btn btn-default">按钮 10</button>
     <div class="btn-group">
       <button type="button" class="btn btn-default dropdown-toggle" data-toggle="dropdown">
         下列
         <span class="caret"></span>
       </button>
       <ul class="dropdown-menu">
         <li><a href="#">下拉链接 1</a></li>
         <li><a href="#">下拉链接 2</a></li>
       </ul>
     </div>
   </div>
 </div>
```