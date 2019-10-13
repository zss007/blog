---
title: chrome 调试小技巧
categories:
- tools
---
最近又学到了一些 chrome 调试的小技巧，赶紧记录下来。
<!--more--> 
### 一、说明
- 快速切换文件
- 在源代码中搜索
- 强制改变元素状态（方便查看不同状态下元素的样式）

### 二、快速切换文件
如果查找每个文件，一般都是打开控制台，在 source 控制面板里面一个一个去找，看下面的图就应该知道，这么多文件，你都不知道在哪个目录下面，然后就只能一个一个点开看

<img src="/assets/tools/chrome/source.png">

后来才发现原来按 Ctrl+P（cmd+p on mac）,就能快速搜寻和打开你项目的文件。

<img src="/assets/tools/chrome/search-file.gif">

### 三、在源代码中搜索
大家都知道如果在要在 Elements 查看源码，只要定位到 Elements 面板，然后按 ctrl+f 就可以了

<img src="/assets/tools/chrome/search-code.png">

可是如果你希望在源代码中搜索要怎么办呢？在页面已经加载的文件中搜寻一个特定的字符串，快捷键是 Ctrl+Shift+F (Cmd+Opt+F)，这种搜寻方式还支持正则表达式哦

<img src="/assets/tools/chrome/search-sourcecode.png">

### 四、强制改变元素状态
chrome 控制台有一个可以模拟 CSS 状态的功能，例如元素的 hover 和 focus，可以很容易的改变元素样式。在 CSS 编辑器中可以利用这个功能查看不同状态下元素的样式，相信这个功能对于模仿别人界面的前端爱好者来说是非常实用的。

<img src="/assets/tools/chrome/dom-status.gif">