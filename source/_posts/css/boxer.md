---
title: css 之盒模型及 BFC
categories:
- css
---
CSS 盒模型是 CSS 的基石，非常重要的一块内容。
<!--more-->
### 一、盒模型基本概念
- margin + border + padding + content
- 标准模型： 计算内容宽度时 width = content width，高度计算相同
- IE 模型： 计算内容宽度时 width = content + padding + border
- box-sizing: border-box（IE 模型）、content-box（标准模型）

### 二、JS 获取盒模型的宽高
- dom.style.width/height: 只能取到内联样式
- dom.currentStyle.width/height: 拿到计算后的宽高，但只有 IE 支持
- window.getComputedStyle(dom).width/height: 拿到计算后的宽高，兼容所有浏览器
- dom.getBoundingClientRect().width/height: 拿到四条边相对左边上边的距离，然后获取宽高

### 三、边距重叠
父子边距重叠、兄弟边距重叠、空元素边距重叠(自身上下边距重叠)，取最大值
### 四、BFC
##### 4.1、块级格式化上下文，其原理
- 在同一个 BFC 元素内部垂直方向发生边距重叠
- BFC 区域不会与浮动元素 box 重叠
- 独立容器，内外互不影响
- 计算 BFC 高度，浮动元素参与计算

##### 4.2、创建 BFC
- position的值不为 static 或者 relative
- float 不为 none
- display 为 table 相关
- overflow 不为 visible

##### 4.3、BFC 实例
```
<section id="sec">
    <style>
        #sec {
            overflow: hidden;
            background: #f00;
        }

        .child {
            height: 100px;
            margin-top: 10px;
            background: yellow;
        }
    </style>
    <article class="child"></article>
</section>
```
##### 4.4、BFC 应用
```
<!-- BFC 垂直方向边距重叠 -->
<section id="margin">
    <style>
        #margin {
            background: pink;
            overflow: hidden;
        }

        #margin>p {
            margin: 5px auto 25px;
            background: red;
        }
    </style>
    <p>垂直 1</p>
    <!-- 创建一个 BFC 来消除边距重合 -->
    <div style="overflow: hidden;">
        <p>垂直 2</p>
    </div>
    <p>垂直 3</p>
</section>

<!-- BFC 不与 float 重叠 -->
<section id="layout">
    <style>
        #layout .left {
            float: left;
            width: 100px;
            height: 100px;
            background: pink;
        }

        #layout .right {
            height: 110px;
            background: #ccc;
            overflow: hidden;
        }
    </style>
    <div class="left">float left</div>
    <div class="right">float right</div>
</section>

<!-- BFC 子元素即使是 float 也会参与高度计算 -->
<section id="float">
    <style>
        #float {
            background: red;
            overflow: auto;
        }

        #float .float {
            float: left;
            font-size: 25px;
        }
    </style>
    <div class="float">我是浮动元素</div>
</section>
```