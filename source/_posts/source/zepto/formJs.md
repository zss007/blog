---
title: zepto 之 form.js
categories:
- source
---
研究总结下之前用过的 zepto 类库，先从表单模块源码开始。
<!--more-->
### 一、源码
```
    //部分代码改动，为了消除警告和格式排版
    //form表单(序列化form表单里面字段和注册submit事件以及触发表单提交事件)
    (function ($) {
        //序列表单内容为JSON数组
        $.fn.serializeArray = function () {
            var name, type, result = [],
                add = function (value) {
                    if (value.forEach) return value.forEach(add);
                    result.push({name: name, value: value})
                };
            if (this[0]) $.each(this[0].elements, function (_, field) {
                type = field.type;
                name = field.name;
                //排除fieldset，禁用元素，submit,reset,button，file和未被选中的radio,checkbox,原因是这些元素不需要传递给服务器
                if (name && field.nodeName.toLowerCase() != 'fieldset' && !field.disabled && type != 'submit' && type != 'reset' && type != 'button' && type != 'file' &&
                    ((type != 'radio' && type != 'checkbox') || field.checked))
                //{name:value}形式加入到结果数组中
                    add($(field).val())
            });
            return result
        };

        //序列表表单内容为查询字符串
        $.fn.serialize = function () {
            var result = [];
            this.serializeArray().forEach(function (elm) {
                //每个key-value，都保守URI编码
                result.push(encodeURIComponent(elm.name) + '=' + encodeURIComponent(elm.value))
            });
            return result.join('&')
        };

        //提交表单
        $.fn.submit = function (callback) {
            //0 in arguments 判断是否传了回调函数，传了回调，就当成绑定submit事件
            if (0 in arguments) this.bind('submit', callback);
            else if (this.length) { //没有传回调，当成直接提交，this.length表示有表单元素
                var event = $.Event('submit');
                //get和eq方法不同，get方法返回的dom节点，eq返回的还是Zepto对象集合
                this.eq(0).trigger(event);
                if (!event.isDefaultPrevented()) this.get(0).submit()
            }
            return this
        }
    })(Zepto);
```
### 二、源码分析
整个表单模块被一个自运行匿名函数包含，防止作用域污染。
#### serializeArray
1、循环递归函数 add 解决 value 为数组的情况;  
2、formObject.elements 为包含表单里面所有字段节点的数组;  
提示: 如果 elements 数组具有名称(input 标签的 id 或 name 属性)，那么该元素的名称就是 formObject 的一个属性，因此可以使用名称而不是数字来引用 input 对象；比如，假设 x 是一个 form 对象，其中的一个 input 对象的名称是 fname，则可以使用 x.fname 来引用该对象
#### submit
1、$.Event 是创建事件,trigger(event) 触发自定义事件,并不会提交表单;
2、document.forms[0].submit() 执行默认提交表单处理