---
title: clipboard 之介绍
categories:
- essay
---
最近有用到粘贴复制功能，调用了[clipboard.js](https://github.com/zenorocha/clipboard.js)库，对其原理很感兴趣。研读源码后开始总结，这篇主要是功能介绍。
<!--more-->
### 一、为什么使用它
复制文字到剪切板不应该很难去实现。它不需要配置几十个步骤或者加载几百 KB 的文件。最重要的是，它不应该依赖 Flash 或其他臃肿的框架。这是 clipboard.js 诞生的原因。

### 二、开始
```
npm install clipboard --save
```
可以通过 npm 来安装它，如果你不使用包管理，仅需要下载一个 [ZIP](https://github.com/zenorocha/clipboard.js/archive/master.zip) 文件。
首先，引入位于 dist 目录下的脚本文件，或者引入一个第三方[CDN](https://github.com/zenorocha/clipboard.js/wiki/CDN-Providers)。
```
<script src="dist/clipboard.min.js"></script>
```
然后，你需要通过传入一个[DOM 选择器](https://github.com/zenorocha/clipboard.js/blob/master/demo/constructor-selector.html#L18), [HTML 元素](https://github.com/zenorocha/clipboard.js/blob/master/demo/constructor-node.html#L16-L17), 或者 [HTML 元素数组](https://github.com/zenorocha/clipboard.js/blob/master/demo/constructor-nodelist.html#L18-L19)作为参数，来实例化对象。
```
new Clipboard('.btn');
```
本质上，我们需要获取所有选择器匹配到的元素，并为每一个选择器设置监听事件。但仔细想想，如果有成百上千个匹配到的元素，这样做会耗费大量内存。因此，我们使用[事件代理](http://stackoverflow.com/questions/1687296/what-is-dom-event-delegation)，通过一个事件监听器来取代多个事件监听。毕竟，[性能是问题](https://twitter.com/hashtag/perfmatters)。

### 三、使用
我们正在经历一场声明式的复兴，这就是为什么我们决定利用 HTML5 data 属性 来提高可用性的原因。
#### 3.1、从另一个元素复制文本
一个很常见的用例是从另一个元素复制内容。你可以给目标元素添加一个`data-clipboard-target`属性来实现这个功能。这个属性的值就是能匹配到另一个元素的选择器。
```
<!-- Target -->
<input id="foo" value="https://github.com/zenorocha/clipboard.js.git">

<!-- Trigger -->
<button class="btn" data-clipboard-target="#foo">
    <img src="assets/clippy.svg" alt="Copy to clipboard">
</button>
```
#### 3.2、从另一个元素剪切文本
此外，你可以定义一个`data-clipboard-action`属性来指明你想要复制（copy）还是剪切（cut）内容。如果你省略这个属性，则默认为复制（copy）。正如你所预料的，剪切（cut）动作只在`<input>`或`<textarea>`元素起作用。
```
<!-- Target -->
<textarea id="bar">Mussum ipsum cacilds...</textarea>

<!-- Trigger -->
<button class="btn" data-clipboard-action="cut" data-clipboard-target="#bar">
    Cut to clipboard
</button>
```
#### 3.3、从属性复制文本
事实上，你甚至不需要从另一个元素来复制内容。你仅需要给目标元素设置一个`data-clipboard-text`属性就可以了。
```
<!-- Trigger -->
<button class="btn" data-clipboard-text="Just because you can doesn't mean you should — clipboard.js">
    Copy to clipboard
</button>
```
### 四、事件
如果你想要展示一些用户反馈，或者在用户复制/剪切后获取已经选择的文字，这里有个示例供你参考。我们通过触发自定义事件，例如`success`和`error`，让你可以设置监听并实现自定义逻辑。
```
var clipboard = new Clipboard('.btn');

clipboard.on('success', function(e) {
    console.info('Action:', e.action);
    console.info('Text:', e.text);
    console.info('Trigger:', e.trigger);

    e.clearSelection();
});

clipboard.on('error', function(e) {
    console.error('Action:', e.action);
    console.error('Trigger:', e.trigger);
});
```
### 五、高级选项
如果你不想修改 HTML，我们提供了一个非常方面的命令式的 API 给你使用。你需要做的就是声明一个函数，做一些处理，并返回一个值。例如，如果你希望动态设置 target，你需要返回一个节点（Node）.
```
new Clipboard('.btn', {
    target: function(trigger) {
        return trigger.nextElementSibling;
    }
});
```
如果你希望动态设置 text，你需要返回一个字符串。
```
new Clipboard('.btn', {
    text: function(trigger) {
        return trigger.getAttribute('aria-label');
    }
});
```
如果在 Bootstrap 模态框（Modals）中使用，或是在其他修改焦点的类库中使用，你会希望将获得焦点的元素设置为 container 属性的值。
```
new Clipboard('.btn', {
    container: document.getElementById('modal')
});
```
同样地，如果你使用单页应用，你可能想要更加精确地管理 DOM 的生命周期。你可以清理事件以及创建的对象。
```
var clipboard = new Clipboard('.btn');
clipboard.destroy();
```
### 六、浏览器支持
这个库依赖于 [Selection](https://developer.mozilla.org/en-US/docs/Web/API/Selection) 和 [execCommand](https://developer.mozilla.org/en-US/docs/Web/API/Document/execCommand) 的 API。前者 [兼容所有的浏览器](http://caniuse.com/#search=selection)，后者兼容以下浏览器。
<img src="/assets/essay/clipboard.png" alt="通用的占位符缩略图">
好消息是，如果你需要支持旧浏览器，clipboard.js 可以优雅降级。你所要做的就是在`success`事件触发时提示用户“已复制！”，`error`事件触发时提示用户“按 Ctrl+C 复制文字”（此时用户要复制的文字已经选择了）。你也可以通过运行`Clipboard.isSupported()`来检查浏览器是否支持 clipboard.js，如果不支持，你可以隐藏复制/剪切按钮。