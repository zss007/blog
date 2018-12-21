---
title: css 之页面布局一
categories:
- css
---
页面布局主要考察 HTML 以及 CSS 的功底，对页面布局的把控能力。
<!--more-->
### 一、场景
三栏布局：要求：高度已知，左右固定，中间自适应。先设置初始化样式：
```
* {
    padding: 0;
    margin: 0;
}

section {
    margin-top: 15px;
}

.container div {
    min-height: 100px;
}
```
### 二、float 布局
```
<!-- 浮动解决方案 -->
<section class="container float">
    <style>
        .float {
            clear: both;
        }

        .float .left {
            float: left;
            width: 100px;
            background: aquamarine;
        }

        .float .right {
            float: right;
            width: 100px;
            background: azure;
        }

        .float .center {
            overflow: hidden;
            background: beige;
        }
    </style>
    <div class="left"></div>
    <div class="right"></div>
    <div class="center">
        <h1>浮动解决方案</h1>
        <p>兼容性好，但是要处理浮动关系</p>
    </div>
</section>
```
### 三、绝对定位布局
```
<!-- 绝对定位解决方案 -->
<section class="container absolute">
    <style>
        .absolute {
            position: relative;
            height: 100px;
        }

        .absolute div {
            position: absolute;
        }

        .absolute .left {
            width: 100px;
            left: 0;
            background: aquamarine;
        }

        .absolute .right {
            width: 100px;
            right: 0;
            background: azure;
        }

        .absolute .center {
            right: 100px;
            left: 100px;
            background: beige;
        }
    </style>      
    <div class="left"></div>
    <div class="right"></div>
    <div class="center">
        <h1>绝对定位解决方案</h1>
        <p>简单，快捷，可扩展性差</p>
    </div>
</section>
```
### 四、flex 布局
```
<!-- flex 解决方案 -->
<section class="container flex">
    <style>
        .flex {
            display: flex;
        }

        .flex .left {
            width: 100px;
            background: aquamarine;
        }

        .flex .right {
            width: 100px;
            background: azure;
        }

        .flex .center {
            flex: 1;
            background: beige;
        }
    </style>
    <div class="left"></div>
    <div class="center">
        <h1>flex 解决方案</h1>
        <p>简单，快捷，兼容性差</p>
    </div>
    <div class="right"></div>
</section>
```
### 五、table 布局
```
<!-- table 解决方案 -->
<section class="container table">
    <style>
        .table {
            display: table;
            width: 100%;
            height: 100px;
        }

        .table div {
            display: table-cell;
        }

        .table .left {
            width: 100px;
            background: aquamarine;
        }

        .table .right {
            width: 100px;
            background: azure;
        }

        .table .center {
            background: beige;
        }
    </style>
    <div class="left"></div>
    <div class="center">
        <h1>table 解决方案</h1>
        <p>兼容性非常好，但是三栏等高，可能不合适</p>
    </div>
    <div class="right"></div>
</section>
```
### 六、grid 布局
```
<!-- grid 解决方案 -->
<section class="container grid">
    <style>
        .grid {
            display: grid;
            height: 100%;
            grid-template-rows: 100px;
            grid-template-columns: 100px auto 100px;
        }

        .grid .left {
            background: aquamarine;
        }

        .grid .right {
            background: azure;
        }

        .grid .center {
            background: beige;
        }
    </style>
    <div class="left"></div>
    <div class="center">
        <h1>grid 解决方案</h1>
        <p>功能强大，兼容性不好</p>
    </div>
    <div class="right"></div>
</section>
```