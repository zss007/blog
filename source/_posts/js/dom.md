---
title: js 之 DOM 事件
categories:
- js
---
HTML DOM 允许 JavaScript 对 HTML 事件作出反应。
<!--more-->
### 一、DOM 事件级别
- DOM0 级：DOM0 级事件就是将一个函数赋值给一个事件处理属性，element.onclick=function(){}
- DOM1 级：没有与事件相关的东西
- DOM2 级：DOM2 级事件定义了 addEventListener 和 removeEventListener 两个方法，分别用来绑定和解绑事件，element.addEventListener('click',function(){},false)
- DOM3 级：DOM3 级事件在 DOM2 级事件的基础上添加了更多的事件类型，element.addEventListener('keyup',function(){},false)

### 二、DOM 事件模型 & 事件流
- 冒泡：从当前元素到最外层元素
- 捕获：从最外层元素到目标元素
- 事件流：捕获 -> 目标阶段 -> 冒泡

### 三、描述 Dom 事件流捕获的具体流程
- window - document - html - body - ... - 目标元素
- 获取 html 标签：document.documentElement

### 四、Event 对象常见应用
- e.preventDefault()
- e.stopPropagation()
- e.target: 获取触发事件的元素
- e.currentTarget: 指的是绑定事件的元素
- e.stopImmediatePropagation: 事件响应优先级

### 五、自定义事件
```
const eve = new Event('custom');
ev.addEventListener('custom', () => {
    ...
});
ev.dispathEvent(eve);

// Event 不足，只能指定事件名，如果要传参数，可以使用 CustomEvent，如：
var obj = document
// 添加一个适当的事件监听器
obj.addEventListener("cat", function (e) { console.log(e.detail) })
// 创建并分发事件
var event = new CustomEvent("cat", { "detail": { "hazcheeseburger": true } })
obj.dispatchEvent(event)
```
