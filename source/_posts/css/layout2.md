---
title: css 之页面布局二
categories:
- css
---
上节主要讲到水平三栏布局，这节主要探讨垂直三栏布局。
<!--more-->
### 一、场景
三栏布局：要求：上下高度固定，中间自适应。先设置初始化样式：
```
 * {
    padding: 0;
    margin: 0;
}

html,
body,
section {
    height: 100%;
}

section {
    float: left;
    margin-left: 15px;
}

.container div {
    width: 200px;
}
```
### 二、绝对定位布局
```
<!-- 绝对定位解决方案 -->
<section class="container absolute">
    <style>
        .absolute {
            position: relative;
            width: 200px;
        }

        .absolute div {
            position: absolute;
        }

        .absolute .top {
            top: 0;
            height: 100px;
            background: aquamarine;
        }

        .absolute .bottom {
            bottom: 0;
            height: 100px;
            background: azure;
        }

        .absolute .middle {
            overflow: scroll;
            top: 100px;
            bottom: 100px;
            background: beige;
        }
    </style>
    <div class="top"></div>
    <div class="bottom"></div>
    <div class="middle">
        <h1>绝对定位解决方案</h1>
        <p>简单快捷，可拓展性差</p>
    </div>
</section>
```
### 三、flex 布局
```
<!-- flex 解决方案 -->
<section class="container flex">
    <style>
        .flex {
            display: flex;
            flex-direction: column;
        }

        .flex .top {
            height: 100px;
            background: aquamarine;
        }

        .flex .bottom {
            height: 100px;
            background: azure;
        }

        .flex .middle {
            flex: 1;
            overflow: scroll;
            background: beige;
        }
    </style>
    <div class="top"></div>
    <div class="middle">
        <h1>flex解决方案</h1>
        <p>简单快捷，兼容性差</p>
    </div>
    <div class="bottom"></div>
</section>
```
### 四、table 布局
```
<!-- table 解决方案 -->
<section class="container table">
    <style>
        .table {
            display: table;
            width: 200px;
        }

        .table div {
            display: table-row;
        }

        .table .top {
            height: 100px;
            background: aquamarine;
        }

        .table .middle {
            display: block;
            overflow: scroll;
            height: 100%;
            background: beige;
        }

        .table .bottom {
            height: 100px;
            background: azure;
        }
    </style>
    <div class="top">&nbsp;</div>
    <div class="middle">
        <h1>table 解决方案</h1>
        <p>兼容性非常好，但是不太推荐</p>
    </div>
    <div class="bottom">&nbsp;</div>
</section>
```
### 五、grid 布局
```
<!-- grid 解决方案 -->
<section class="container grid">
    <style>
        .grid {
            display: grid;
            grid-template-columns: 200px;
            grid-template-rows: 100px auto 100px;
        }

        .grid .top {
            background: aquamarine;
        }

        .grid .bottom {
            background: azure;
        }

        .grid .middle {
            overflow: scroll;
            background: beige;
        }
    </style>
    <div class="top"></div>
    <div class="middle">
        <h1>grid 解决方案</h1>
        <p>功能强大，兼容性不好</p>
    </div>
    <div class="bottom"></div>
</section>
```