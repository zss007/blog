---
title: zepto 之 data.js
categories:
- source
---
一个全面的 data() 方法，能够在内存中存储任意对象。
<!--more-->
### 一、源码
```
    (function($){
        //+new Date() 转化为Number类型，会调用Date.prototype上的valueOf方法，等同于Date.prototype.getTime()
        var data = {}, dataAttr = $.fn.data, camelize = $.camelCase,
            exp = $.expando = 'Zepto' + (+new Date()), emptyArray = [];

        // Get value from node:
        // 1. first try key as given,
        // 2. then try camelized key,
        // 3. fall back to reading "data-*" attribute.
        function getData(node, name) {
            var id = node[exp], store = id && data[id];
            //如果没有指定属性值，则返回整个属性存储对象
            if (name === undefined) return store || setData(node);
            else {
                if (store) {
                    if (name in store) return store[name];
                    var camelName = camelize(name);
                    if (camelName in store) return store[camelName]
                }
                //调用之前核心库里面的$.fn.data方法获取元素的属性值
                return dataAttr.call($(node), name)
            }
        }

        // Store value under camelized key on node
        function setData(node, name, value) {
            //第一次为设置值，并将所有dom元素上的自定义属性放到缓存中；后面则从缓存中提取数据
            var id = node[exp] || (node[exp] = ++$.uuid),
                store = data[id] || (data[id] = attributeData(node));
            if (name !== undefined) store[camelize(name)] = value;
            return store
        }

        // Read all "data-*" attributes from a node
        //将node中所有自定义属性组成对象返回
        function attributeData(node) {
            var store = {};
            $.each(node.attributes || emptyArray, function(i, attr){
                if (attr.name.indexOf('data-') == 0)
                    store[camelize(attr.name.replace('data-', ''))] =
                        $.zepto.deserializeValue(attr.value)
            });
            return store
        }

        //根据参数设置或提取数据
        $.fn.data = function(name, value) {
            return value === undefined ?
                // set multiple values via object
                $.isPlainObject(name) ?
                    this.each(function(i, node){
                        $.each(name, function(key, value){ setData(node, key, value) })
                    }) :
                    // get value from first element
                    (0 in this ? getData(this[0], name) : undefined) :
                // set value on all elements
                this.each(function(){ setData(this, name, value) })
        };

        //调用$.fn.data方法
        $.data = function(elem, name, value) {
            return $(elem).data(name, value)
        };

        //判断dom元素是否存储了缓存对象
        $.hasData = function(elem) {
            var id = elem[exp], store = id && data[id];
            return store ? !$.isEmptyObject(store) : false
        };

        //移除dom元素上的缓存对象
        $.fn.removeData = function(names) {
            if (typeof names == 'string') names = names.split(/\s+/);
            return this.each(function(){
                var id = this[exp], store = id && data[id];
                if (store) $.each(names || store, function(key){
                    delete store[names ? camelize(this) : key]
                })
            })
        }

        // Generate extended `remove` and `empty` functions
        //调用remove和empty进行dom操作的同时移除dom元素上的缓存对象
        ;['remove', 'empty'].forEach(function(methodName){
            var origFn = $.fn[methodName];
            $.fn[methodName] = function() {
                var elements = this.find('*');
                if (methodName === 'remove') elements = elements.add(this);
                elements.removeData();
                return origFn.call(this)
            }
        })
    })(Zepto);
```
### 二、源码分析
1、0 in this 判断是否为数组
2、当 data.js 文件加载完后会生成一个 exp 变量，这个 exp 变量就是节点存储标识的自定义属性名，这个属性名对应的是一个个 id，通过这个 id 可以到 data 对象中找到该节点对应的数据缓存对象。