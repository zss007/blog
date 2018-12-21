---
title: bootstrap 之 code & grid 样式
categories:
- bootstrap
---
现在开始介绍 bootstrap 的 code(代码) 和 grid(栅格)，包括 code.less、grid.less。
<!--more-->
### 一、源码
#### 1、code.less
##### 1.1、code.less 源码
```
 //
 // Code (inline and block) 代码
 // --------------------------------------------------
 
 
 // Inline and block code styles
 code,
 kbd,
 pre,
 samp {
   font-family: @font-family-monospace;
 }
 
 // Inline code 显示单行内联代码
 code {
   padding: 2px 4px;
   font-size: 90%;
   color: @code-color;
   background-color: @code-bg;
   border-radius: @border-radius-base;
 }
 
 // User input typically entered via keyboard 显示用户输入代码
 kbd {
   padding: 2px 4px;
   font-size: 90%;
   color: @kbd-color;
   background-color: @kbd-bg;
   border-radius: @border-radius-small;
   box-shadow: inset 0 -1px 0 rgba(0,0,0,.25);
 
   kbd {
     padding: 0;
     font-size: 100%;
     font-weight: bold;
     box-shadow: none;
   }
 }
 
 // Blocks of code 显示多行块代码
 pre {
   display: block;
   padding: ((@line-height-computed - 1) / 2);
   margin: 0 0 (@line-height-computed / 2);
   font-size: (@font-size-base - 1); // 14px to 13px
   line-height: @line-height-base;
   word-break: break-all;
   word-wrap: break-word;
   color: @pre-color;
   background-color: @pre-bg;
   border: 1px solid @pre-border-color;
   border-radius: @border-radius-base;
 
   // Account for some code outputs that place code tags in pre tags
   code {
     padding: 0;
     font-size: inherit;
     color: inherit;
     white-space: pre-wrap;
     background-color: transparent;
     border-radius: 0;
   }
 }
 
 // Enable scrollable blocks of code
 .pre-scrollable {
   max-height: @pre-scrollable-max-height;
   overflow-y: scroll;
 }
```
##### 1.2、code.less 应用
```
 <!--code&pre-->
 <p><code>&lt;header&gt;</code> 作为内联元素被包围。</p>
 <p>如果需要把代码显示为一个独立的块元素，请使用 &lt;pre&gt; 标签：</p>
 <pre>
 &lt;article&gt;
 	&lt;h1&gt;Article Heading&lt;/h1&gt;
 &lt;/article&gt;
 </pre>
 <!--kbd-->
 <p>使用 kbd 元素表示按键输入:</p>
 <p>使用 <kbd>ctrl + p</kbd> 来打开打印窗口。</p>
 <!--samp-->
 <p>使用 samp 元素包含电脑输出的内容:</p>
 <p><samp>This text is output from a computer program....</samp></p>
 <!--pre-scrollable-->
 <div class="container">
   <h2>代码</h2>
   <p>使用 pre 元素输出多行:</p>
   <pre>在 pre 元素中的文本
 	宽度的显示与文本的宽度一样，
 	保留了  空  格 和
 
 	换行。</pre>
   <p>如果你添加 .pre-scrollable 类,  pre 元素最大的高度 max-height 为 350px ，并生成一个 Y 轴的滚动条:</p>
   <pre class="pre-scrollable">在 pre 元素中的文本
 	宽度的显示与文本的宽度一样，
 	保留了  空  格 和
 
 	换行。</pre>
 </div>
```
#### 2、grid.less
##### 2.1、grid.less 源码
```
 //
 // Grid system(网格系统)
 // --------------------------------------------------
 
 
 // Container widths(容器宽度)
 //
 // Set the container width, and override it for fixed navbars in media queries.
 .container {
   .container-fixed();
 
   @media (min-width: @screen-sm-min) {
     width: @container-sm;
   }
   @media (min-width: @screen-md-min) {
     width: @container-md;
   }
   @media (min-width: @screen-lg-min) {
     width: @container-lg;
   }
 }
 
 
 // Fluid container
 //
 // Utilizes the mixin meant for fixed width containers, but without any defined
 // width for fluid, full width layouts.
 .container-fluid {
   .container-fixed();
 }
 
 
 // Row(行容器)
 //
 // Rows contain and clear the floats of your columns.
 .row {
   .make-row();
 }
 
 
 // Columns(网格列)
 //
 // Common styles for small and large grid columns
 .make-grid-columns();
 
 
 // Extra small grid(生成超小网格)
 //
 // Columns, offsets, pushes, and pulls for extra small devices like
 // smartphones.
 .make-grid(xs);
 
 
 // Small grid(生成小网格)
 //
 // Columns, offsets, pushes, and pulls for the small device range, from phones
 // to tablets.
 @media (min-width: @screen-sm-min) {
   .make-grid(sm);
 }
 
 
 // Medium grid(生成中等网格)
 //
 // Columns, offsets, pushes, and pulls for the desktop device range.
 @media (min-width: @screen-md-min) {
   .make-grid(md);
 }
 
 
 // Large grid(生成大网格)
 //
 // Columns, offsets, pushes, and pulls for the large desktop device range.
 @media (min-width: @screen-lg-min) {
   .make-grid(lg);
 }
```
##### 2.2、grid.less 应用
```
 <div class="container">
     <div class="row" >
         <div class="col-xs-6 col-sm-3" 
             style="background-color: #dedef8;
             box-shadow: inset 1px -1px 1px #444, inset -1px 1px 1px #444;">
             <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit.</p>
         </div>
         <div class="col-xs-6 col-sm-3" 
         style="background-color: #dedef8;box-shadow: 
         inset 1px -1px 1px #444, inset -1px 1px 1px #444;">
             <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do 
             eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut 
             enim ad minim veniam, quis nostrud exercitation ullamco laboris 
             nisi ut aliquip ex ea commodo consequat.
             </p>
             <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do 
             eiusmod tempor incididunt ut. 
             </p>
         </div>
  
         <div class="clearfix visible-xs"></div>
  
         <div class="col-xs-6 col-sm-3" 
         style="background-color: #dedef8;
         box-shadow:inset 1px -1px 1px #444, inset -1px 1px 1px #444;">
             <p>Ut enim ad minim veniam, quis nostrud exercitation ullamco 
             laboris nisi ut aliquip ex ea commodo consequat. 
             </p>
         </div>
         <div class="col-xs-6 col-sm-3" 
         style="background-color: #dedef8;box-shadow: 
         inset 1px -1px 1px #444, inset -1px 1px 1px #444;">
             <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do 
             eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut 
             enim ad minim 
             </p>
         </div>
     </div>
 </div>
```
##### 2.3、grid.less 工作原理
1、行必须放置在 .container class 内，以便获得适当的对齐（alignment）和内边距（padding）。   
2、使用行来创建列的水平组。   
3、内容应该放置在列内，且唯有列可以是行的直接子元素。   
4、预定义的网格类，比如 .row 和 .col-xs-4，可用于快速创建网格布局。LESS 混合类可用于更多语义布局。   
5、列通过内边距（padding）来创建列内容之间的间隙。该内边距是通过 .rows 上的外边距（margin）取负，表示第一列和最后一列的行偏移。   
6、网格系统是通过指定您想要横跨的十二个可用的列来创建的。例如，要创建三个相等的列，则使用三个 .col-xs-4。   