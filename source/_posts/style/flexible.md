---
title: style 之 H5 终端适配
categories:
- style 
---
这节主要是研究手淘团队的适配方案 amfe-flexible，及[源码](https://github.com/amfe/lib-flexible)解析。
<!--more-->
### 一、基础
#### 1.1、css 中的 1px 并不等于设备的 1px
在 css 中我们一般使用 px 作为单位，在桌面浏览器中 css 的 1 个像素往往都是对应着电脑屏幕的 1 个物理像素，这可能会造成我们的一个错觉，那就是 css 中的像素就是设备的物理像素。但实际情况却并非如此，css 中的像素只是一个抽象的单位，在不同的设备或不同的环境中，css 中的 1px 所代表的设备物理像素是不同的。在为桌面浏览器设计的网页中，我们无需对这个津津计较，但在移动设备上，必须弄明白这点。在早先的移动设备中，屏幕像素密度都比较低，如 iphone3，它的分辨率为 320x480，在 iphone3 上，一个 css 像素确实是等于一个屏幕物理像素的。后来随着技术的发展，移动设备的屏幕像素密度越来越高，从 iphone4 开始，苹果公司便推出了所谓的 Retina 屏，分辨率提高了一倍，变成 640x960，但屏幕尺寸却没变化，这就意味着同样大小的屏幕上，像素却多了一倍，这时，一个 css 像素是等于两个物理像素的。其他品牌的移动设备也是这个道理。例如安卓设备根据屏幕像素密度可分为 ldpi、mdpi、hdpi、xhdpi 等不同的等级，分辨率也是五花八门，安卓设备上的一个 css 像素相当于多少个屏幕物理像素，也因设备的不同而不同，没有一个定论。
#### 1.2、devicePixelRatio
在移动端浏览器中以及某些桌面浏览器中，window 对象有一个 devicePixelRatio 属性，它的官方的定义为：设备物理像素和设备独立像素的比例，也就是 devicePixelRatio = 物理像素 / 独立像素。css 中的 px 就可以看做是设备的独立像素，所以通过 devicePixelRatio，我们可以知道该设备上一个 css 像素代表多少个物理像素。例如，在 Retina 屏的iphone上，devicePixelRatio 的值为 2，也就是说 1 个 css 像素相当于 2 个物理像素。
### 二、源码分析
##### 2.1、源码
```
(function flexible (window, document) {
  var docEl = document.documentElement
  var dpr = window.devicePixelRatio || 1

  // adjust body font size
  function setBodyFontSize () {
    if (document.body) {
      document.body.style.fontSize = (12 * dpr) + 'px'
    }
    else {
      document.addEventListener('DOMContentLoaded', setBodyFontSize)
    }
  }
  setBodyFontSize();

  // set 1rem = viewWidth / 10
  function setRemUnit () {
    var rem = docEl.clientWidth / 10
    docEl.style.fontSize = rem + 'px'
  }

  setRemUnit()

  // reset rem unit on page resize
  window.addEventListener('resize', setRemUnit)
  window.addEventListener('pageshow', function (e) {
    if (e.persisted) {
      setRemUnit()
    }
  })

  // detect 0.5px supports
  if (dpr >= 2) {
    var fakeBody = document.createElement('body')
    var testElement = document.createElement('div')
    testElement.style.border = '.5px solid transparent'
    fakeBody.appendChild(testElement)
    docEl.appendChild(fakeBody)
    if (testElement.offsetHeight === 1) {
      docEl.classList.add('hairlines')
    }
    docEl.removeChild(fakeBody)
  }
}(window, document))
```
##### 2.2、源码分析
- setBodyFontSize 用来设置 body 的 fontSize，值为 (12 * dpr) + 'px'
- setRemUnit 用来设置 document.documentElement 即 html 标签的 fontSize，值为 clientWidth 的 1/10
- resize、pageshow 在窗口大小调整或[从缓存中载入页面](http://www.runoob.com/jsref/event-onpageshow.html)时，重新调用 setRemUnit

##### 2.3、.5px 方案
2014 年的 WWDC 大会中，Ted O’Conor 在分享 “设计响应的Web体验” 主题时提到关于 Retina Hairlines 一词，也就是 Retina 极细的线：在 Retina 屏上仅仅显示 1 物理像素的边框。
amfe-flexible 的源码中，在页面上添加 div 元素，设置边框为 0.5px，并判断 offsetHeight 是否为 1 来判断是否支持 .5px，即如果支持 .5px 的话，那么在其 html 标签上添加 hairlines 类。这样就可以在写样式时进行 ”渐进增强“，即在支持 .5px 时使用其来显示极细的线。