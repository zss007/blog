---
title: css 之 Sass 入门一
categories:
- css
---
Sass 是一门高于 CSS 的元语言，它能用来清晰地、结构化地描述文件样式，有着比普通 CSS 更加强大的功能。Sass 能够提供更简洁、更优雅的语法，同时提供多种功能来创建可维护和管理的样式表。采用 Ruby 语言编写的一款 CSS 预处理语言，诞生于 2007 年，是最大的成熟的 CSS 预处理语言。Sass 有两种：一种是早期版本，文件后缀名为 sass，不使用大括号和分号，以严格的缩进式语法规则来书写；另外一种与我们的 CSS 语法非常类似。
<!--more-->
### 一、安装及运行
```
//安装 sass
sudo gem install sass

//查看 sass版本号(检查 sass 是否安装成功)
sass -v

//运行(进入到相应目录文件下再跑命令，--watch 起到监听作用，一旦 style.scss 发生改变自动编译到 style.css 中)
sass --watch style.scss:style.css
```
### 二、变量
sass 的变量必须是 $ 开头，后面紧跟变量名，而变量值和变量名之间就需要使用冒号(:)分隔开（就像 CSS 属性设置一样），如果值后面加上 !default 则表示默认值。
#### 1、普通变量
定义之后可以在全局范围内使用
```
//sass style
$fontSize: 12px;
body{
    font-size:$fontSize;
}

//css style
body{
    font-size:12px;
}
```
#### 2、默认变量
sass 的默认变量仅需要在值后面加上`!default`即可，覆盖的方式也很简单，只需要在默认变量之前重新声明下变量即可
```
//sass style
$baseLineHeight: 2;
$baseLineHeight: 1.5 !default;
body{
    line-height: $baseLineHeight; 
}

//css style
body{
    line-height:2;
}
```
#### 3、特殊变量
一般我们定义的变量都为属性值，可直接使用，但是如果变量作为属性或在某些特殊情况下等则必须要以 #{$variables} 形式使用
```
//sass style
$borderDirection:       top !default; 
$baseFontSize:          12px !default;
$baseLineHeight:        1.5 !default;

//应用于class和属性
.border-#{$borderDirection}{
  border-#{$borderDirection}:1px solid #ccc;
}
//应用于复杂的属性值
body{
    font:#{$baseFontSize}/#{$baseLineHeight};
}

//css style
.border-top{
  border-top:1px solid #ccc;
}
body {
  font: 12px/1.5;
}
```
#### 4、多值变量
多值变量分为 list 类型和 map 类型，简单来说 list 类型有点像 js 中的数组，而 map 类型有点像 js 中的对象
list 数据可通过空格，逗号或小括号分隔多个值，可用 nth(`$var`,`$index`) 取值
```
$px: 5px 10px 20px 30px;

//二维数据，相当于js中的二维数组
$px: 5px 10px, 20px 30px;
$px: (5px 10px) (20px 30px);

//使用 sass style
$linkColor: #08c #333 !default;//第一个值为默认值，第二个鼠标滑过值
a{
  color:nth($linkColor,1);

  &:hover{
    color:nth($linkColor,2);
  }
}

//css style
a{
  color:#08c;
}
a:hover{
  color:#333;
}
```
map 数据以 key 和 value 成对出现，其中 value 又可以是 list，可通过 map-get(`$map`,`$key`) 取值
```
//sass style
$headings: (h1: 2em, h2: 1.5em, h3: 1.2em);
@each $header, $size in $headings {
  #{$header} {
    font-size: $size;
  }
}

//css style
h1 {
  font-size: 2em; 
}
h2 {
  font-size: 1.5em; 
}
h3 {
  font-size: 1.2em; 
}
```
#### 5、全局变量
在变量值后面加上 !global 即为全局变量，sass3.4 后的版本中正式应用
```
//sass style
$fontSize: 12px;
body{
  $fontSize: 14px !global;
  font-size:$fontSize;
}
p{
  font-size:$fontSize;
}

//css style
body{
  font-size: 14px;
}
p{
  font-size: 14px;
}
```
### 三、嵌套
#### 1、选择器嵌套
所谓选择器嵌套指的是在一个选择器中嵌套另一个选择器来实现继承，从而增强了 sass 文件的结构性和可读性。在选择器嵌套中，可以使用 & 表示父元素选择器
```
//sass style
nav {
  a {
    color: red;

    header & {
      color:green;
    }
  }
}

//css style
nav a {
  color: red;
}
header nav a {
  color: green;
}
```
#### 2、属性嵌套
所谓属性嵌套指的是有些属性拥有同一个开始单词，如 border-width，border-color 都是以 border 开头
```
//sass style
.box {
  border: {
   top: 1px solid red;
   bottom: 1px solid green;
  }
}

//css style
.box {
    border-top: 1px solid red;
    border-bottom: 1px solid green;
}
```
#### 3、伪元素嵌套
其实伪元素嵌套和属性嵌套非常类似，只不过他需要借助`&`符号一起配合使用
```
//sass style
.clearfix {
  &:before,
  &:after {
    content: "";
    display: table;
  }
  &:after {
    clear: both;
    overflow: hidden;
  }
}

//css style
.clearfix:before, .clearfix:after {
  content: "";
  display: table;
}
.clearfix:after {
  clear: both;
  overflow: hidden;
}
```
### 四、混合宏 & 继承 & 占位符
#### 1、混合宏
使用 @mixin 声明混合，可以传递参数，参数名以 $ 符号开始，多个参数以逗号分开，也可以给参数设置默认值。声明的 @mixin 通过 @include 来调用
无参宏
```
//sass style(无参宏)
@mixin center-block {
    margin-left:auto;
    margin-right:auto;
}
.demo{
    @include center-block;
}

//css style
.demo{
    margin-left:auto;
    margin-right:auto;
}
```
#### 2、有参宏
```
//sass style
@mixin opacity($opacity:50) {
  opacity: $opacity / 100;
  filter: alpha(opacity=$opacity);
}

.opacity{
  @include opacity; //参数使用默认值
}
.opacity-80{
  @include opacity(80); //传递参数
}
```
#### 3、多个参宏
```
@mixin horizontal-line($border:1px dashed #ccc, $padding:10px){
    border-bottom:$border;
    padding-top:$padding;
    padding-bottom:$padding;  
}
.imgtext-h li{
    @include horizontal-line(1px solid #ccc);
}
.imgtext-h--product li{
    @include horizontal-line($padding:15px);
}
```
#### 4、多组参宏(如果一个参数可以有多组值，如 box-shadow、transition 等，那么参数则需要在变量后加三个点表示，如 $variables...)
```
//box-shadow可以有多组值，所以在变量参数后面添加...
@mixin box-shadow($shadow...) {
  -webkit-box-shadow:$shadow;
  box-shadow:$shadow;
}
.box{
  border:1px solid #ccc;
  @include box-shadow(0 2px 2px rgba(0,0,0,.3),0 3px 3px rgba(0,0,0,.3),0 4px 4px rgba(0,0,0,.3));
}
```
#### 5、继承
sass 中，选择器继承可以让选择器继承另一个选择器的所有样式，并联合声明。使用选择器的继承，要使用关键词 @extend，后面紧跟需要继承的选择器。
```
//sass style
h1{
  border: 4px solid #ff9aa9;
}
.speaker{
  @extend h1;
  border-width: 2px;
}

//css style
h1,.speaker{
  border: 4px solid #ff9aa9;
}
.speaker{
  border-width: 2px;
}
```
#### 6、占位符
从 sass 3.2.0 以后就可以定义占位选择器 %。这种选择器的优势在于：如果不调用则不会有任何多余的 css 文件，避免了以前在一些基础的文件中预定义了很多基础的样式，然后实际应用中不管是否使用了`@extend`去继承相应的样式，都会解析出来所有的样式。占位选择器以`%`标识定义，通过`@extend`调用。
```
//sass style
%mt5 {
  margin-top: 5px;
}

.btn {
  @extend %mt5;
}

.block {
  @extend %mt5;
}

//css style
.btn, .block {
  margin-top: 5px;
}
```
#### 7、比较
<img src="/assets/css/sass.png">

### 五、其他
#### 1、注释
类似 CSS 的注释方式，使用 `/*` 开头，结束使用 `*/`，会在编译出来的 CSS 显示
类似 JavaScript 的注释方式，使用 `//`，在编译出来的 CSS 中不会显示
#### 2、数据类型
数字: 如，1、 2、 13、 10px；
字符串：有引号字符串或无引号字符串，如，"foo"、 'bar'、 baz；
颜色：如，blue、 #04a3f9、 rgba(255,0,0,0.5)；
布尔型：如，true、 false；
空值：如，null；
值列表：用空格或者逗号分开，如，1.5em 1em 0 2em 、 Helvetica, Arial, sans-serif。
#### 3、插值 #{}
```
//设置属性值
$properties: (margin, padding);
@mixin set-value($side, $value) {
    @each $prop in $properties {
        #{$prop}-#{$side}: $value;
    }
}
.login-box {
    @include set-value(top, 14px);
}

//构建选择器
@mixin generate-sizes($class, $small, $medium, $big) {
    .#{$class}-small { font-size: $small; }
    .#{$class}-medium { font-size: $medium; }
    .#{$class}-big { font-size: $big; }
}
@include generate-sizes("header-text", 12px, 20px, 40px);
```
当你想设置属性或属性值的时候你可以使用字符串插入进来。另一个有用的用法是构建一个选择器，如上。
```
$btn-success: #5cb85c;
$btn-info: #5bc0de;
$btn-warning: #f0ad4e;
$btn-danger: #d9534f;
@mixin set-bg($name) {
    background-color: $btn-#{$name};//btn有多种状态,都存在变量里;
}
.btn {
    @include set-bg(success);
}
//编译会报错：(Undefined variable: "$btn-".)

@mixin btn-status {
    margin-top: 20px;
    background: #F00;
}
$flag: "status";
.box {
    @include btn-#{$flag};
}
//也会报错：(Invalid CSS after "...nclude updated -": expected "}", was "#{$flag};")
```
限制 (mixin 中) 如上。