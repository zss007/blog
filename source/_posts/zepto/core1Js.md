---
title: zpeto 之核心代码一
categories:
- zepto
---
这篇主要研究 zepto 核心模块的结构和一些工具方法。
<!--more-->
### 一、源码
```
(function (global, factory) {
  if (typeof define === 'function' && define.amd)
    define(function () {
      return factory(global)
    });
  else
    factory(global)
}(this, function (window) {
  var Zepto = (function () {
    //定义要使用的变量
    var undefined, key, $, classList, emptyArray = [], concat = emptyArray.concat, filter = emptyArray.filter, slice = emptyArray.slice,
      document = window.document,
      elementDisplay = {}, classCache = {},
      //不需要添加'px'后缀的css属性
      cssNumber = {
        'column-count': 1,
        'columns': 1,
        'font-weight': 1,
        'line-height': 1,
        'opacity': 1,
        'z-index': 1,
        'zoom': 1
      },
      //匹配HTML代码
      fragmentRE = /^\s*<(\w+|!)[^>]*>/,
      //匹配单个HTML标签
      singleTagRE = /^<(\w+)\s*\/?>(?:<\/\1>|)$/,
      //匹配自闭合标签
      tagExpanderRE = /<(?!area|br|col|embed|hr|img|input|link|meta|param)(([\w:]+)[^>]*)\/>/ig,
      //匹配根节点
      rootNodeRE = /^(?:body|html)$/i,
      //匹配A-Z
      capitalRE = /([A-Z])/g,
      //这些属性已经存在相应方法($.fn.x)
      methodAttributes = ['val', 'css', 'html', 'text', 'data', 'width', 'height', 'offset'],
      //操作方法集合，方便高效生成方法
      adjacencyOperators = ['after', 'prepend', 'before', 'append'],
      //容器对象，用于生成fragment虚拟的节点元素
      table = document.createElement('table'),
      tableRow = document.createElement('tr'),
      containers = {
        'tr': document.createElement('tbody'),
        'tbody': table, 'thead': table, 'tfoot': table,
        'td': tableRow, 'th': tableRow,
        '*': document.createElement('div')
      },
      //document.readyState的状态
      readyRE = /complete|loaded|interactive/,
      //匹配 a-z、A-Z、0-9、下划线、连词符组合起来的单词，这其实就是单个id和class的命名规则
      simpleSelectorRE = /^[\w-]*$/,
      //存储对象的类型
      class2type = {},
      //存储对象的toString字符串方法
      toString = class2type.toString,
      zepto = {},
      camelize, uniq,
      tempParent = document.createElement('div'),
      //修正dom元素属性
      propMap = {
        'tabindex': 'tabIndex',
        'readonly': 'readOnly',
        'for': 'htmlFor',
        'class': 'className',
        'maxlength': 'maxLength',
        'cellspacing': 'cellSpacing',
        'cellpadding': 'cellPadding',
        'rowspan': 'rowSpan',
        'colspan': 'colSpan',
        'usemap': 'useMap',
        'frameborder': 'frameBorder',
        'contenteditable': 'contentEditable'
      },
      /**
       * 判断是否为数组类型(如果不支持isArray，那么调用instanceof来判断是否是数组)
       * 这里存在一个问题，如果存在iframe，那么iframe中的数组对象 instanceof 页面的Array为false
       * 有个更好的解决方案：Object.prototype.toString.call(object) === '[object Array]'，即$.type为array
       * */
      isArray = Array.isArray || function (object) {
          return object instanceof Array
        };
    //元素是否匹配选择器
    zepto.matches = function (element, selector) {
      //没参数，非元素，直接返回
      if (!selector || !element || element.nodeType !== 1) return false;
      //如果浏览器支持MatchesSelector直接调用
      var matchesSelector = element.matches || element.webkitMatchesSelector ||
        element.mozMatchesSelector || element.oMatchesSelector ||
        element.matchesSelector;
      if (matchesSelector) return matchesSelector.call(element, selector);
      //浏览器不支持MatchesSelector的话
      var match, parent = element.parentNode, temp = !parent;
      //如果元素没有父元素，存入到临时的div中
      if (temp) (parent = tempParent).appendChild(element);
      //在父元素中查找匹配选择器的子元素，如果子元素中出现了当前元素，那么说明该子元素匹配选择器(注意~取反位运算符，作用是将值取负数再减1，如-1变成0、0变成-1)
      match = ~zepto.qsa(parent, selector).indexOf(element);
      //移除临时div子元素
      temp && tempParent.removeChild(element);
      return match
    };
    //判断类型模块(基础类型会被转成引用类型，通过{}.toString.call(obj))
    function type(obj) {
      return obj == null ? String(obj) :
        class2type[toString.call(obj)] || "object"
    }

    //判断是否是函数
    function isFunction(value) {
      return type(value) == "function"
    }

    //判断是否为window对象(通过判断对象的obj的window属性指向自身)
    function isWindow(obj) {
      return obj != null && obj == obj.window
    }

    //判断是否为document对象(通过判断对象的节点属性nodeType等于属性DOCUMENT_NODE)
    function isDocument(obj) {
      return obj != null && obj.nodeType == obj.DOCUMENT_NODE
    }

    //判断是否为对象
    function isObject(obj) {
      return type(obj) == "object"
    }

    //测试对象是否是“纯粹”的对象，这个对象是通过对象常量("{}")或者new Object创建的，如果是，则返回true(通过判断obj._proto_==Object.prototype)
    function isPlainObject(obj) {
      return isObject(obj) && !isWindow(obj) && Object.getPrototypeOf(obj) == Object.prototype
    }

    //类数组(不是function或window对象，存在length，且length==0或者length为number而且length-1属性也存在的引用类型或者真正的数组)
    //比如：likeArrayData = {'0': 0,'1': 1,"2": 2,length: 3}
    function likeArray(obj) {
      var length = !!obj && 'length' in obj && obj.length,
        type = $.type(obj);

      return 'function' != type && !isWindow(obj) && (
          'array' == type || length === 0 ||
          (typeof length == 'number' && length > 0 && (length - 1) in obj)
        )
    }

    //删除数组的null和undefined(undefined类型转换和null相同)
    function compact(array) {
      return filter.call(array, function (item) {
        return item != null
      })
    }

    //数组扁平化，最多只能展开一层嵌套(巧用apply方法)
    function flatten(array) {
      return array.length > 0 ? $.fn.concat.apply([], array) : array
    }

    //字符串驼峰化(将word-word的形式的字符串转换成wordWord的形式，'-'可以为一个或多个)
    camelize = function (str) {
      return str.replace(/-+(.)?/g, function (match, chr) {
        return chr ? chr.toUpperCase() : ''
      })
    };
    //将驼峰式的写法转换成连字符'-'的写法
    function dasherize(str) {
      return str.replace(/::/g, '/')
        .replace(/([A-Z]+)([A-Z][a-z])/g, '$1_$2')
        .replace(/([a-z\d])([A-Z])/g, '$1_$2')
        .replace(/_/g, '-')
        .toLowerCase()
    }

    //数组去重(使用原生数组过滤方法filter)
    uniq = function (array) {
      return filter.call(array, function (item, idx) {
        return array.indexOf(item) == idx
      })
    };
    //将参数变为正则表达式
    function classRE(name) {
      return name in classCache ?
        classCache[name] : (classCache[name] = new RegExp('(^|\\s)' + name + '(\\s|$)'))
    }

    //将一些css样式，为number类型
    function maybeAddPx(name, value) {
      return (typeof value == "number" && !cssNumber[dasherize(name)]) ? value + "px" : value
    }

    //获取节点名对应dom元素默认display属性
    function defaultDisplay(nodeName) {
      var element, display;
      if (!elementDisplay[nodeName]) {
        //根据节点名新建dom元素，再添加到document中
        element = document.createElement(nodeName);
        document.body.appendChild(element);
        //获取最终样式display作为默认样式，并移除创建的元素
        display = getComputedStyle(element, '').getPropertyValue("display");
        element.parentNode.removeChild(element);
        //如果默认的最终样式display为none，强制设置为block，并缓存到对象中，下次可直接使用，避免dom操作
        display == "none" && (display = "block");
        elementDisplay[nodeName] = display
      }
      return elementDisplay[nodeName]
    }

    //返回元素的直接子元素
    function children(element) {
      //如果存在children属性，那么将element.children改成数组
      return 'children' in element ?
        slice.call(element.children) :
        //否则调用element.childNodes，并返回元素节点
        $.map(element.childNodes, function (node) {
          if (node.nodeType == 1) return node
        })
    }

    //实际构造函数(将dom数组转化为类数组对象{0:dom0,1:dom1,length:2,selector:xxx}，并设置对应的length属性和selector属性)
    function Z(dom, selector) {
      var i, len = dom ? dom.length : 0;
      for (i = 0; i < len; i++) this[i] = dom[i]
      this.length = len;
      this.selector = selector || ''
    }

    //从给定的html片段生成dom数组
    zepto.fragment = function (html, name, properties) {
      var dom, nodes, container;
      //如果是单个标签，则用该标签名创建dom对象(如'<div></div>'，不能含有)
      if (singleTagRE.test(html)) dom = $(document.createElement(RegExp.$1));
      if (!dom) {
        //对html进行修复，如<p class="test" />修复成<p class="test" /></p>
        if (html.replace) html = html.replace(tagExpanderRE, "<$1></$2>");
        //如果没有指定标签名，则获取标签名。如传入<div>test</div>，获取到的name为div
        if (name === undefined) name = fragmentRE.test(html) && RegExp.$1;
        //如果传进来的标签名或者查找到的标签名不在containers中，那么设置标签名为*，即div
        if (!(name in containers)) name = '*';
        //将containers标签名对应dom对象赋值给container
        container = containers[name];
        //用容器将html字符串片段包起来
        container.innerHTML = '' + html;
        //获取子节点(这里使用slice.call是很有必要的)
        dom = $.each(slice.call(container.childNodes), function () {
          container.removeChild(this)
        })
      }
      //如果属性值(即$函数的context参数)为纯对象，则给dom元素设置属性
      if (isPlainObject(properties)) {
        nodes = $(dom);
        $.each(properties, function (key, value) {
          //zepto已经定义了相应的方法，则调用zepto对应的方法($.fn.x)
          if (methodAttributes.indexOf(key) > -1) nodes[key](value);
          //否则统一调用zepto的attr方法设置属性($.fn.attr)
          else nodes.attr(key, value)
        })
      }

      return dom
    };
    //返回Z函数的实例
    zepto.Z = function (dom, selector) {
      return new Z(dom, selector)
    };
    //判断是否是zepto对象，因为做了'zepto.Z.prototype = Z.prototype = $.fn'操作，同时所有的zepto的实例都继承了$.fn对象的方法
    zepto.isZ = function (object) {
      return object instanceof zepto.Z
    };
    //初始化zepto对象
    zepto.init = function (selector, context) {
      var dom;
      //如果没有给出选择器，那么返回空的zepto对象{"length":0,"selector":""}
      if (!selector) return zepto.Z();
      //处理字符串选择器，用法$(selector, [context])、$(htmlString)、$(htmlString, attributes)
      else if (typeof selector == 'string') {
        selector = selector.trim();
        //selector的第一个字符为<，并且为html片段，根据html片段创建节点(Chrome 21和Firefox 15会报DOM error 12错误，如果标签不是以<开始的话)
        if (selector[0] == '<' && fragmentRE.test(selector))
          dom = zepto.fragment(selector, RegExp.$1, context), selector = null;
        //如果存在context，那么在指定上下文选择节点
        else if (context !== undefined) return $(context).find(selector);
        //如果没有指定上下文，那么用CSS选择器来选择节点
        else dom = zepto.qsa(document, selector)
      }
      //如果选择器是函数，那么在dom加载完成后执行，参数为$(如果dom已经加载则直接执行；否则设置监听等待dom加载完成再执行，用法Zepto(function($){ ... }))
      else if (isFunction(selector)) return $(document).ready(selector);
      //如果选择器就是zepto对象，那么直接返回这个对象，用法$(<Zepto collection>)
      else if (zepto.isZ(selector)) return selector;
      else {
        //如果选择器为数组，那么删除数组中的null和undefined，用法$(<DOM nodes>)
        if (isArray(selector)) dom = compact(selector);
        //如果选择器是对象，则将对象包装成数组，用法$(<DOM nodes>)
        else if (isObject(selector))
          dom = [selector], selector = null;
        //如果是html片段，那么根据html片段创建相应节点
        else if (fragmentRE.test(selector))
          dom = zepto.fragment(selector.trim(), RegExp.$1, context), selector = null;
        //如果存在context，那么在指定上下文选择节点
        else if (context !== undefined) return $(context).find(selector);
        //如果没有指定上下文，那么用CSS选择器来选择节点
        else dom = zepto.qsa(document, selector)
      }
      //根据节点dom来创建一个新的zepto对象
      return zepto.Z(dom, selector)
    };
    //当调用方法$时即调用zepto.init(该方法实现选择节点的细节和创造可调用插件方法的zepto对象)
    $ = function (selector, context) {
      return zepto.init(selector, context)
    };
    //扩展目标对象的属性
    function extend(target, source, deep) {
      for (key in source)
        //如果为深度复制，并且源对象的属性值为纯粹对象或者数组
        if (deep && (isPlainObject(source[key]) || isArray(source[key]))) {
          //如果源对象的属性值为纯粹对象，并且目标对象对应的属性值不为纯粹对象，则将目标对象对应的属性值置为空对象
          if (isPlainObject(source[key]) && !isPlainObject(target[key]))
            target[key] = {};
          //如果源对象的属性值为数组，并且目标对象对应的属性值不为数组，则将目标对象对应的属性值置为空数组
          if (isArray(source[key]) && !isArray(target[key]))
            target[key] = [];
          //递归调用extend函数
          extend(target[key], source[key], deep)
        }
        else if (source[key] !== undefined) target[key] = source[key]
    }

    /**
     * $.extend 方法可以用来扩展目标对象的属性。目标对象的同名属性会被源对象的非空属性(undefined)覆盖。
     * 调用方法：$.extend(target, [source, [source2, ...]]) | $.extend(true, target, [source, ...])
     * */
    $.extend = function (target) {
      //args用来接收第一个参数外的所有参数
      var deep, args = slice.call(arguments, 1);
      //如果第一个参数target为boolean类型，那么是$.extend(true, target, [source, ...])调用，并将target重新赋值
      if (typeof target == 'boolean') {
        deep = target;
        target = args.shift()
      }
      args.forEach(function (arg) {
        extend(target, arg, deep)
      });
      return target
    };
    //zepto的选择器实现方法，不直接调用querySelectorAll方法是因为这个方法相较于getElementById、getElementsByClassName、getElementsByTagName而言更耗性能
    zepto.qsa = function (element, selector) {
      var found, //找到的元素
        maybeID = selector[0] == '#',
        maybeClass = !maybeID && selector[0] == '.',
        //确保一个字符的标签也能够被检测到(去掉id和class选择器第一个字符)
        nameOnly = maybeID || maybeClass ? selector.slice(1) : selector,
        //单个id、class、标签的命名规则
        isSimple = simpleSelectorRE.test(nameOnly);
      return (element.getElementById && isSimple && maybeID) ? //Safari的DocumentFragment元素没有getElementById方法
        ((found = element.getElementById(nameOnly)) ? [found] : []) :
        (element.nodeType !== 1 && element.nodeType !== 9 && element.nodeType !== 11) ? [] : //排除不合法的element(合法节点:1 Element、9 Document、11 DocumentFragment)
          //slice.call 处理所获取到的集合，这样获取到的DOM集合就成为了数组，可以直接使用数组的方法了
          slice.call(
            isSimple && !maybeID && element.getElementsByClassName ? //DocumentFragment元素没有getElementsByClassName/TagName方法
              maybeClass ? element.getElementsByClassName(nameOnly) : //简单的单个选择器，是类选择器，则使用getElementsByClassName查询类名符合的元素
                element.getElementsByTagName(selector) : //如果也不是类选择器，那么调用getElementsByTagName查询标签名符合的元素
              element.querySelectorAll(selector) //如果不是简单的单个选择器，或者没有getElementsByClassName方法，那么调用querySelectorAll查询所有
          )
    };
    //过滤掉nodes参数中不符合selector的节点；如果selector为null/undefined那么直接返回节点对象
    function filtered(nodes, selector) {
      return selector == null ? $(nodes) : $(nodes).filter(selector)
    }

    //检查给定的父节点中是否包含有给定的子节点(document.documentElement.contains检测浏览器是否支持contains方法)
    $.contains = document.documentElement.contains ?
      function (parent, node) {
        return parent !== node && parent.contains(node)
      } :
      function (parent, node) {
        while (node && (node = node.parentNode))
          if (node === parent) return true;
        return false
      };
    //判断arg是否是函数，如果是则执行并返回执行结果；否则直接返回args
    function funcArg(context, arg, idx, payload) {
      return isFunction(arg) ? arg.call(context, idx, payload) : arg
    }

    //设置属性
    function setAttribute(node, name, value) {
      //value为null/undefined,处理成删除，否则设值
      value == null ? node.removeAttribute(name) : node.setAttribute(name, value)
    }

    //设置或返回节点的className
    function className(node, value) {
      var klass = node.className || '',
        //为了兼容SVGAnimatedString
        svg = klass && klass.baseVal !== undefined;

      if (value === undefined) return svg ? klass.baseVal : klass;
      svg ? (klass.baseVal = value) : (node.className = value)
    }

    /**
     * 反系列化值("true"=>true, "false"=>false, "null"=>null, "42"=>42, "42.5"=>42.5, "08"=>"08", JSON=>parse if valid, String=>self)
     * +value将字符串转成number类型，注意'08'会被转成数字8，最后当成字符串处理；如果是非数字格式字符串的话，被转化为NaN，继续后面的判断操作
     * */
    function deserializeValue(value) {
      try {
        return value ?
          value == "true" ||
          ( value == "false" ? false :
            value == "null" ? null :
              +value + "" == value ? +value :
                /^[\[\{]/.test(value) ? $.parseJSON(value) :
                  value )
          : value
      } catch (e) {
        return value
      }
    }

    $.type = type;
    $.isFunction = isFunction;
    $.isWindow = isWindow;
    $.isArray = isArray;
    $.isPlainObject = isPlainObject;
    //判断是否为空对象
    $.isEmptyObject = function (obj) {
      var name;
      for (name in obj) return false
      return true
    };
    //是否为数值(不为null或undefined、不为布尔值、不为NaN、为有限数值、不为字符串或者字符串但是字符串长度大于0且转换成数字后不为NaN)
    $.isNumeric = function (val) {
      var num = Number(val), type = typeof val;
      return val != null && type != 'boolean' &&
        (type != 'string' || val.length) && !isNaN(num) && isFinite(num) || false
    };
    //返回指定元素在数组中的索引值(第三个参数fromIndex为可选参数，表示从哪个索引值开始向后查找)
    $.inArray = function (elem, array, i) {
      return emptyArray.indexOf.call(array, elem, i)
    };
    //字符串转换成驼峰式的字符串
    $.camelCase = camelize;
    //删除字符串头尾的空格(如果参数为null或undefined，则直接返回空字符串；否则调用字符串原生的trim方法去除头尾的空格)
    $.trim = function (str) {
      return str == null ? "" : String.prototype.trim.call(str)
    };

    //为了拓展插件
    $.uuid = 0;
    $.support = {};
    $.expr = {};
    $.noop = function () {
    };
    //遍历类数组或对象中的元素，将回调函数返回值组成一个新的数组，并将该数组扁平化后返回，会将null及undefined排除
    $.map = function (elements, callback) {
      var value, values = [], i, key;
      if (likeArray(elements))
        for (i = 0; i < elements.length; i++) {
          value = callback(elements[i], i);
          if (value != null) values.push(value)
        }
      else
        for (key in elements) {
          value = callback(elements[key], key);
          if (value != null) values.push(value)
        }
      return flatten(values)
    };
    //用来遍历数组或者类数组对象(回调函数this指向的是item，回调函数参数是(index, item))
    $.each = function (elements, callback) {
      var i, key;
      if (likeArray(elements)) {  //类数组
        for (i = 0; i < elements.length; i++)
          if (callback.call(elements[i], i, elements[i]) === false) return elements
      } else {    //对象
        for (key in elements)
          if (callback.call(elements[key], key, elements[key]) === false) return elements
      }
      return elements
    };
    //其实就是数组的filter函数
    $.grep = function (elements, callback) {
      return filter.call(elements, callback)
    };
    //将原生JSON解析方法赋值给$.parseJSON
    if (window.JSON) $.parseJSON = JSON.parse;
    //生成class2type映射
    $.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function (i, name) {
      class2type["[object " + name + "]"] = name.toLowerCase()
    });
    //将$.fn分别赋值给Z和zepto.Z的prototype
    zepto.Z.prototype = Z.prototype = $.fn;

    //将内部的操作函数放到$.zepto命名空间下，以便外部能够使用
    zepto.uniq = uniq;
    zepto.deserializeValue = deserializeValue;
    $.zepto = zepto;

    return $
  })();

  //赋值给全局对象使Zepto成为全局函数
  window.Zepto = Zepto;
  //如果$已经被占用了，那么使用Zepto；否则使用$代替Zepto
  window.$ === undefined && (window.$ = Zepto);
  return Zepto
}));
```
### 二、源码分析
#### 1、amd
zepto 方法支持 amd 模块加载，整体是个匿名闭包自运行函数。
#### 2、结构
function Z 是真正的构造函数，这里 zepto 实例结构为类数组是因为方法调用好用的数组方法，不直接用数组格式是因为继承`$.fn`，使得所有实例可直接调用`$.fn`中的方法。
这里把构造函数、初始化、选择器查找、生成 fragment 等方法统一封装到 zepto 对象中去了。
这里有个地方需要注意，为了使 zepto 实例可调 $.fn 方法，把`$.fn`给了 Z.prototype，同时为了使得 zepto.isZ 里面 object instanceof zepto.Z 可以正常运行，让 zepto.Z.prototype = Z.prototype(instanceof 判断`object._proto_`是否在 zepto.Z.prototype 的原型链上)；将 zepto 对象赋值给 $ 的 zepto 属性，以便外部能够调用；而`$`是一个函数，内调用 zepto.init 完成初始化。
直接在`$`下的属性方法是工具方法，在`$.fn`对象中的是zepto 实例的方法。
将`$`返回给 Zepto 并赋值给全局对象，判断`$`是否被占用了，如果占用了那么使用 Zepto；否则可使用`$`代替 Zepto。
#### 3、$.type
这里很巧妙的先将特定的 js 类型数据遍历赋值到 class2type 对象属性上，后面直接从 class2type 对象获取相应的数据类型。
#### 4、$.isArray
判断是否为数组类型(如果不支持 isArray，那么调用 instanceof 来判断是否是数组)。这里存在一个问题，如果存在 iframe，那么 iframe 中的数组对象 instanceof 页面的 Array 为false。有个更好的解决方案：Object.prototype.toString.call(object) === '[object Array]'，即 $.type 为 array。
#### 5、function flatten
这里巧妙的用了 apply 方法，使得 concat.apply 能够展开一层嵌套。
#### 6、uniq
这是除了新建一个结果数组、利用对象属性外，另一个很不错的数组去重方法。调用 filter 方法过滤掉当前索引不等于首次出现在数组位置的元素。
#### 7、HTMLCollection.item
获取相应位置出现的 dom 元素，等同于 HTMLCollection[item]
#### 8、children & childNodes
childNodes: 返回指定元素的子元素集合，包括属性节点，元素节点和文本节点；
children: 返回所有元素节点的子元素集合。