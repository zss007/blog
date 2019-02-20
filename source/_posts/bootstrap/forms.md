---
title: bootstrap 之 form 样式
categories:
- bootstrap
---
现在开始介绍 bootstrap 的表单(forms)样式。
<!--more-->
### 一、源码
#### 1、forms.less
##### 1.1、forms.less 源码
```
 //
 // Forms(表单)
 // --------------------------------------------------
 
 // Normalize non-controls(定制非表单控件)
 //
 // Restyle and baseline non-control form elements.
 fieldset {
   padding: 0;
   margin: 0;
   border: 0;
   // Chrome and Firefox set a `min-width: min-content;` on fieldsets,
   // so we reset that to ensure it behaves more like a standard block element.
   // See https://github.com/twbs/bootstrap/issues/12359.
   min-width: 0;
 }
 
 legend {
   display: block;
   width: 100%;
   padding: 0;
   margin-bottom: @line-height-computed;
   font-size: (@font-size-base * 1.5);
   line-height: inherit;
   color: @legend-color;
   border: 0;
   border-bottom: 1px solid @legend-border-color;
 }
 
 label {
   display: inline-block;
   max-width: 100%; // Force IE8 to wrap long content (see https://github.com/twbs/bootstrap/issues/13141)
   margin-bottom: 5px;
   font-weight: bold;
 }
 
 // Normalize form controls(定制表单控件)
 //
 // While most of our form styles require extra classes, some basic normalization
 // is required to ensure optimum display with or without those classes to better
 // address browser inconsistencies.
 
 // Override content-box in Normalize (* isn't specific enough)
 input[type="search"] {
   .box-sizing(border-box);
 }
 
 // Position radios and checkboxes better
 input[type="radio"],
 input[type="checkbox"] {
   margin: 4px 0 0;
   margin-top: 1px \9; // IE8-9
   line-height: normal;
 }
 input[type="file"] {
   display: block;
 }
 
 // Make range inputs behave like textual form controls
 input[type="range"] {
   display: block;
   width: 100%;
 }
 
 // Make multiple select elements height not fixed
 select[multiple],
 select[size] {
   height: auto;
 }
 
 // Focus for file, radio, and checkbox
 input[type="file"]:focus,
 input[type="radio"]:focus,
 input[type="checkbox"]:focus {
   .tab-focus();
 }
 
 // Adjust output element
 output {
   display: block;
   padding-top: (@padding-base-vertical + 1);
   font-size: @font-size-base;
   line-height: @line-height-base;
   color: @input-color;
 }
 
 // Common form controls(普通的表单控件)
 //
 // Shared size and type resets for form controls. Apply `.form-control` to any
 // of the following form controls:
 //
 // select
 // textarea
 // input[type="text"]
 // input[type="password"]
 // input[type="datetime"]
 // input[type="datetime-local"]
 // input[type="date"]
 // input[type="month"]
 // input[type="time"]
 // input[type="week"]
 // input[type="number"]
 // input[type="email"]
 // input[type="url"]
 // input[type="search"]
 // input[type="tel"]
 // input[type="color"]
 .form-control {
   display: block;
   width: 100%;
   height: @input-height-base; // Make inputs at least the height of their button counterpart (base line-height + padding + border)
   padding: @padding-base-vertical @padding-base-horizontal;
   font-size: @font-size-base;
   line-height: @line-height-base;
   color: @input-color;
   background-color: @input-bg;
   background-image: none; // Reset unusual Firefox-on-Android default style; see https://github.com/necolas/normalize.css/issues/214
   border: 1px solid @input-border;
   border-radius: @input-border-radius; // Note: This has no effect on <select>s in some browsers, due to the limited stylability of <select>s in CSS.
   .box-shadow(inset 0 1px 1px rgba(0,0,0,.075));
   .transition(~"border-color ease-in-out .15s, box-shadow ease-in-out .15s");
 
   // Customize the `:focus` state to imitate native WebKit styles.
   .form-control-focus();
 
   // Placeholder
   .placeholder();
 
   // Unstyle the caret on `<select>`s in IE10+.
   &::-ms-expand {
     border: 0;
     background-color: transparent;
   }
 
   // Disabled and read-only inputs
   //
   // HTML5 says that controls under a fieldset > legend:first-child won't be
   // disabled if the fieldset is disabled. Due to implementation difficulty, we
   // don't honor that edge case; we style them as disabled anyway.
   &[disabled],
   &[readonly],
   fieldset[disabled] & {
     background-color: @input-bg-disabled;
     opacity: 1; // iOS fix for unreadable disabled content; see https://github.com/twbs/bootstrap/issues/11655
   }
 
   &[disabled],
   fieldset[disabled] & {
     cursor: @cursor-disabled;
   }
 
   // Reset height for `textarea`s
   textarea& {
     height: auto;
   }
 }
 
 // Search inputs in iOS
 //
 // This overrides the extra rounded corners on search inputs in iOS so that our
 // `.form-control` class can properly style them. Note that this cannot simply
 // be added to `.form-control` as it's not specific enough. For details, see
 // https://github.com/twbs/bootstrap/issues/11586.
 input[type="search"] {
   -webkit-appearance: none;
 }
 
 // Special styles for iOS temporal inputs
 //
 // In Mobile Safari, setting `display: block` on temporal inputs causes the
 // text within the input to become vertically misaligned. As a workaround, we
 // set a pixel line-height that matches the given height of the input, but only
 // for Safari. See https://bugs.webkit.org/show_bug.cgi?id=139848
 //
 // Note that as of 9.3, iOS doesn't support `week`.
 @media screen and (-webkit-min-device-pixel-ratio: 0) {
   input[type="date"],
   input[type="time"],
   input[type="datetime-local"],
   input[type="month"] {
     &.form-control {
       line-height: @input-height-base;
     }
 
     &.input-sm,
     .input-group-sm & {
       line-height: @input-height-small;
     }
 
     &.input-lg,
     .input-group-lg & {
       line-height: @input-height-large;
     }
   }
 }
 
 // Form groups(表单组)
 //
 // Designed to help with the organization and spacing of vertical forms. For
 // horizontal forms, use the predefined grid classes.
 .form-group {
   margin-bottom: @form-group-margin-bottom;
 }
 
 
 // Checkboxes and radios(表单控件:复选框checkbox和单选框radio)
 //
 // Indent the labels to position radios/checkboxes as hanging controls.
 .radio,
 .checkbox {
   position: relative;
   display: block;
   margin-top: 10px;
   margin-bottom: 10px;
 
   label {
     min-height: @line-height-computed; // Ensure the input doesn't jump when there is no text
     padding-left: 20px;
     margin-bottom: 0;
     font-weight: normal;
     cursor: pointer;
   }
 }
 .radio input[type="radio"],
 .radio-inline input[type="radio"],
 .checkbox input[type="checkbox"],
 .checkbox-inline input[type="checkbox"] {
   position: absolute;
   margin-left: -20px;
   margin-top: 4px \9;
 }
 .radio + .radio,
 .checkbox + .checkbox {
   margin-top: -5px; // Move up sibling radios or checkboxes for tighter spacing
 }
 
 // Radios and checkboxes on same line(复选框和单选框水平排列)
 .radio-inline,
 .checkbox-inline {
   position: relative;
   display: inline-block;
   padding-left: 20px;
   margin-bottom: 0;
   vertical-align: middle;
   font-weight: normal;
   cursor: pointer;
 }
 .radio-inline + .radio-inline,
 .checkbox-inline + .checkbox-inline {
   margin-top: 0;
   margin-left: 10px; // space out consecutive inline controls
 }
 
 // Apply same disabled cursor tweak as for inputs(添加disable样式)
 // Some special care is needed because <label>s don't inherit their parent's `cursor`.
 //
 // Note: Neither radios nor checkboxes can be readonly.
 input[type="radio"],
 input[type="checkbox"] {
   &[disabled],
   &.disabled,
   fieldset[disabled] & {
     cursor: @cursor-disabled;
   }
 }
 // These classes are used directly on <label>s
 .radio-inline,
 .checkbox-inline {
   &.disabled,
   fieldset[disabled] & {
     cursor: @cursor-disabled;
   }
 }
 // These classes are used on elements with <label> descendants
 .radio,
 .checkbox {
   &.disabled,
   fieldset[disabled] & {
     label {
       cursor: @cursor-disabled;
     }
   }
 }
 
 
 // Static form control text(静态表单控制文本)
 //
 // Apply class to a `p` element to make any string of text align with labels in
 // a horizontal form layout.
 .form-control-static {
   // Size it appropriately next to real form controls
   padding-top: (@padding-base-vertical + 1);
   padding-bottom: (@padding-base-vertical + 1);
   // Remove default margin from `p`
   margin-bottom: 0;
   min-height: (@line-height-computed + @font-size-base);
 
   &.input-lg,
   &.input-sm {
     padding-left: 0;
     padding-right: 0;
   }
 }
 
 // Form control sizing(表单控件大小)
 //
 // Build on `.form-control` with modifier classes to decrease or increase the
 // height and font-size of form controls.
 //
 // The `.form-group-* form-control` variations are sadly duplicated to avoid the
 // issue documented in https://github.com/twbs/bootstrap/issues/15074.
 .input-sm {
   .input-size(@input-height-small; @padding-small-vertical; @padding-small-horizontal; @font-size-small; @line-height-small; @input-border-radius-small);
 }
 .form-group-sm {
   .form-control {
     height: @input-height-small;
     padding: @padding-small-vertical @padding-small-horizontal;
     font-size: @font-size-small;
     line-height: @line-height-small;
     border-radius: @input-border-radius-small;
   }
   select.form-control {
     height: @input-height-small;
     line-height: @input-height-small;
   }
   textarea.form-control,
   select[multiple].form-control {
     height: auto;
   }
   .form-control-static {
     height: @input-height-small;
     min-height: (@line-height-computed + @font-size-small);
     padding: (@padding-small-vertical + 1) @padding-small-horizontal;
     font-size: @font-size-small;
     line-height: @line-height-small;
   }
 }
 
 .input-lg {
   .input-size(@input-height-large; @padding-large-vertical; @padding-large-horizontal; @font-size-large; @line-height-large; @input-border-radius-large);
 }
 .form-group-lg {
   .form-control {
     height: @input-height-large;
     padding: @padding-large-vertical @padding-large-horizontal;
     font-size: @font-size-large;
     line-height: @line-height-large;
     border-radius: @input-border-radius-large;
   }
   select.form-control {
     height: @input-height-large;
     line-height: @input-height-large;
   }
   textarea.form-control,
   select[multiple].form-control {
     height: auto;
   }
   .form-control-static {
     height: @input-height-large;
     min-height: (@line-height-computed + @font-size-large);
     padding: (@padding-large-vertical + 1) @padding-large-horizontal;
     font-size: @font-size-large;
     line-height: @line-height-large;
   }
 }
 
 
 // Form control feedback states(验证状态)
 //
 // Apply contextual and semantic states to individual form controls.
 .has-feedback {
   // Enable absolute positioning
   position: relative;
 
   // Ensure icons don't overlap text
   .form-control {
     padding-right: (@input-height-base * 1.25);
   }
 }
 // Feedback icon (requires .glyphicon classes)
 .form-control-feedback {
   position: absolute;
   top: 0;
   right: 0;
   z-index: 2; // Ensure icon is above input groups
   display: block;
   width: @input-height-base;
   height: @input-height-base;
   line-height: @input-height-base;
   text-align: center;
   pointer-events: none;
 }
 .input-lg + .form-control-feedback,
 .input-group-lg + .form-control-feedback,
 .form-group-lg .form-control + .form-control-feedback {
   width: @input-height-large;
   height: @input-height-large;
   line-height: @input-height-large;
 }
 .input-sm + .form-control-feedback,
 .input-group-sm + .form-control-feedback,
 .form-group-sm .form-control + .form-control-feedback {
   width: @input-height-small;
   height: @input-height-small;
   line-height: @input-height-small;
 }
 
 // Feedback states
 .has-success {
   .form-control-validation(@state-success-text; @state-success-text; @state-success-bg);
 }
 .has-warning {
   .form-control-validation(@state-warning-text; @state-warning-text; @state-warning-bg);
 }
 .has-error {
   .form-control-validation(@state-danger-text; @state-danger-text; @state-danger-bg);
 }
 
 // Reposition feedback icon if input has visible label above
 .has-feedback label {
 
   & ~ .form-control-feedback {
     top: (@line-height-computed + 5); // Height of the `label` and its margin
   }
   &.sr-only ~ .form-control-feedback {
     top: 0;
   }
 }
 
 // Help text(提示信息)
 //
 // Apply to any element you wish to create light text for placement immediately
 // below a form control. Use for general help, formatting, or instructional text.
 .help-block {
   display: block; // account for any element using help-block
   margin-top: 5px;
   margin-bottom: 10px;
   color: lighten(@text-color, 25%); // lighten the text some for contrast
 }
 
 
 // Inline forms(内联表单)
 //
 // Make forms appear inline(-block) by adding the `.form-inline` class. Inline
 // forms begin stacked on extra small (mobile) devices and then go inline when
 // viewports reach <768px.
 //
 // Requires wrapping inputs and labels with `.form-group` for proper display of
 // default HTML form controls and our custom form controls (e.g., input groups).
 //
 // Heads up! This is mixin-ed into `.navbar-form` in navbars.less.
 .form-inline {
 
   // Kick in the inline
   @media (min-width: @screen-sm-min) {
     // Inline-block all the things for "inline"
     .form-group {
       display: inline-block;
       margin-bottom: 0;
       vertical-align: middle;
     }
 
     // In navbar-form, allow folks to *not* use `.form-group`
     .form-control {
       display: inline-block;
       width: auto; // Prevent labels from stacking above inputs in `.form-group`
       vertical-align: middle;
     }
 
     // Make static controls behave like regular ones
     .form-control-static {
       display: inline-block;
     }
 
     .input-group {
       display: inline-table;
       vertical-align: middle;
 
       .input-group-addon,
       .input-group-btn,
       .form-control {
         width: auto;
       }
     }
 
     // Input groups need that 100% width though
     .input-group > .form-control {
       width: 100%;
     }
 
     .control-label {
       margin-bottom: 0;
       vertical-align: middle;
     }
 
     // Remove default margin on radios/checkboxes that were used for stacking, and
     // then undo the floating of radios and checkboxes to match.
     .radio,
     .checkbox {
       display: inline-block;
       margin-top: 0;
       margin-bottom: 0;
       vertical-align: middle;
 
       label {
         padding-left: 0;
       }
     }
     .radio input[type="radio"],
     .checkbox input[type="checkbox"] {
       position: relative;
       margin-left: 0;
     }
 
     // Re-override the feedback icon.
     .has-feedback .form-control-feedback {
       top: 0;
     }
   }
 }
 
 
 // Horizontal forms(水平表单)
 //
 // Horizontal forms are built on grid classes and allow you to create forms with
 // labels on the left and inputs on the right.
 .form-horizontal {
 
   // Consistent vertical alignment of radios and checkboxes
   //
   // Labels also get some reset styles, but that is scoped to a media query below.
   .radio,
   .checkbox,
   .radio-inline,
   .checkbox-inline {
     margin-top: 0;
     margin-bottom: 0;
     padding-top: (@padding-base-vertical + 1); // Default padding plus a border
   }
   // Account for padding we're adding to ensure the alignment and of help text
   // and other content below items
   .radio,
   .checkbox {
     min-height: (@line-height-computed + (@padding-base-vertical + 1));
   }
 
   // Make form groups behave like rows
   .form-group {
     .make-row();
   }
 
   // Reset spacing and right align labels, but scope to media queries so that
   // labels on narrow viewports stack the same as a default form example.
   @media (min-width: @screen-sm-min) {
     .control-label {
       text-align: right;
       margin-bottom: 0;
       padding-top: (@padding-base-vertical + 1); // Default padding plus a border
     }
   }
 
   // Validation states
   //
   // Reposition the icon because it's now within a grid column and columns have
   // `position: relative;` on them. Also accounts for the grid gutter padding.
   .has-feedback .form-control-feedback {
     right: floor((@grid-gutter-width / 2));
   }
 
   // Form group sizes
   //
   // Quick utility class for applying `.input-lg` and `.input-sm` styles to the
   // inputs and labels within a `.form-group`.
   .form-group-lg {
     @media (min-width: @screen-sm-min) {
       .control-label {
         padding-top: (@padding-large-vertical + 1);
         font-size: @font-size-large;
       }
     }
   }
   .form-group-sm {
     @media (min-width: @screen-sm-min) {
       .control-label {
         padding-top: (@padding-small-vertical + 1);
         font-size: @font-size-small;
       }
     }
   }
 }
```
##### 1.2、forms.less应用
```
 <!--默认垂直表单-->
 <form role="form">
   <div class="form-group">
     <label for="name1">名称</label>
     <input type="text" class="form-control" id="name1" placeholder="请输入名称">
   </div>
   <div class="form-group">
     <label for="inputfile1">文件输入</label>
     <input type="file" id="inputfile1">
     <p class="help-block">这里是块级帮助文本的实例。</p>
   </div>
   <div class="checkbox">
     <label>
       <input type="checkbox">请打勾
     </label>
   </div>
   <button type="submit" class="btn btn-default">提交</button>
 </form>
 <!--内联表单-->
 <form class="form-inline" role="form">
   <div class="form-group">
     <label class="sr-only" for="name2">名称</label>
     <input type="text" class="form-control" id="name2" placeholder="请输入名称">
   </div>
   <div class="form-group">
     <label class="sr-only" for="inputfile2">文件输入</label>
     <input type="file" id="inputfile2">
   </div>
   <div class="checkbox">
     <label>
       <input type="checkbox">请打勾
     </label>
   </div>
   <button type="submit" class="btn btn-default">提交</button>
 </form>
 <!--水平表单-->
 <form class="form-horizontal" role="form">
   <!--表单帮助文本-->
   <label>帮助文本实例</label>
   <input class="form-control" type="text" placeholder="">
   <span class="help-block">一个较长的帮助文本块，超过一行，
   需要扩展到下一行。本实例中的帮助文本总共有两行。</span>
   <!--在一个水平表单内的表单标签后放置纯文本-->
   <div class="form-group">
     <label class="col-sm-2 control-label">Email</label>
     <div class="col-sm-10">
       <p class="form-control-static">email@example.com</p>
     </div>
   </div>
   <div class="form-group">
     <label for="firstname" class="col-sm-2 control-label">名字</label>
     <div class="col-sm-10">
       <input type="text" class="form-control" id="firstname" placeholder="请输入名字">
     </div>
   </div>
   <div class="form-group">
     <label for="lastname" class="col-sm-2 control-label">姓</label>
     <div class="col-sm-10">
       <input type="text" class="form-control" id="lastname" placeholder="请输入姓">
     </div>
   </div>
   <div class="form-group">
     <div class="col-sm-offset-2 col-sm-10">
       <div class="checkbox">
         <label>
           <input type="checkbox">请记住我
         </label>
       </div>
     </div>
   </div>
   <div class="form-group">
     <div class="col-sm-offset-2 col-sm-10">
       <button type="submit" class="btn btn-default">登录</button>
     </div>
   </div>
 </form>
 <!--表单控件状态-->
 <form class="form-horizontal" role="form">
   <div class="form-group">
     <label class="col-sm-2 control-label">聚焦</label>
     <div class="col-sm-10">
       <input class="form-control" id="focusedInput" type="text" value="该输入框获得焦点...">
     </div>
   </div>
   <div class="form-group">
     <label for="disabledInput" class="col-sm-2 control-label">禁用</label>
     <div class="col-sm-10">
       <!--禁用的输入框 input-->
       <input class="form-control" id="disabledInput" type="text" placeholder="该输入框禁止输入..." disabled>
     </div>
   </div>
   <!--禁用的字段集 fieldset-->
   <fieldset disabled>
     <div class="form-group">
       <label for="disabledTextInput" class="col-sm-2 control-label">禁用输入（Fieldset disabled）</label>
       <div class="col-sm-10">
         <input type="text" id="disabledTextInput" class="form-control" placeholder="禁止输入">
       </div>
     </div>
     <div class="form-group">
       <label for="disabledSelect" class="col-sm-2 control-label">禁用选择菜单（Fieldset disabled）</label>
       <div class="col-sm-10">
         <select id="disabledSelect" class="form-control">
           <option>禁止选择</option>
         </select>
       </div>
     </div>
   </fieldset>
   <!--验证状态-->
   <div class="form-group has-success">
     <label class="col-sm-2 control-label" for="inputSuccess">输入成功</label>
     <div class="col-sm-10">
       <input type="text" class="form-control" id="inputSuccess">
     </div>
   </div>
   <div class="form-group has-warning">
     <label class="col-sm-2 control-label" for="inputWarning">输入警告</label>
     <div class="col-sm-10">
       <input type="text" class="form-control" id="inputWarning">
     </div>
   </div>
   <div class="form-group has-error">
     <label class="col-sm-2 control-label" for="inputError">输入错误</label>
     <div class="col-sm-10">
       <input type="text" class="form-control" id="inputError">
     </div>
   </div>
 </form>
```