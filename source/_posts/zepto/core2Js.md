---
title: zpeto 之核心代码二
categories:
- zepto
---
这篇研究 zepto 核心代码的实例方法。
<!--more-->
### 一、源码
```
//定义zepto对象实例能够调用的方法
$.fn = {
  constructor: zepto.Z,
  length: 0,
  //由于zepto对象是类数组对象，所以拷贝一些有用的数组方法可直接使用
  forEach: emptyArray.forEach,
  reduce: emptyArray.reduce,
  push: emptyArray.push,
  sort: emptyArray.sort,
  splice: emptyArray.splice,
  indexOf: emptyArray.indexOf,
  //如果参数是zepto对象则转成数组，再将this转成数组合并
  concat: function () {
    var i, value, args = [];
    for (i = 0; i < arguments.length; i++) {
      value = arguments[i];
      args[i] = zepto.isZ(value) ? value.toArray() : value
    }
    return concat.apply(zepto.isZ(this) ? this.toArray() : this, args)
  },
  //调用内部工具函数$.map，并将返回结果包装成zepto对象
  map: function (fn) {
    return $($.map(this, function (el, i) {
      return fn.call(el, i, el)
    }))
  },
  //返回的数组再次包装成zepto对象
  slice: function () {
    return $(slice.apply(this, arguments))
  },
  //判断上下文是否已经存在，如果存在则立即执行，否则监听DOMContentLoaded事件
  ready: function (callback) {
    //需要检查document.body是否存在，因为IE浏览器会在它还没创建body元素时就触发document的ready事件
    if (readyRE.test(document.readyState) && document.body) callback($);
    else document.addEventListener('DOMContentLoaded', function () {
      callback($)
    }, false);
    return this
  },
  //不传参返回集合中所有元素并转成数组格式，传参大于0返回相应索引元素，小于0返回倒序返回
  get: function (idx) {
    return idx === undefined ? slice.call(this) : this[idx >= 0 ? idx : idx + this.length]
  },
  //将zepto类数组对象转成数组
  toArray: function () {
    return this.get()
  },
  //返回当前zepto对象的length属性，也即集合中元素的个数
  size: function () {
    return this.length
  },
  //从其父节点中删除当前集合中的元素，有效的从dom中移除(这里的重点是parentNode.removeChild(this); 如果是没有父节点的这里不会执行)
  remove: function () {
    return this.each(function () {
      if (this.parentNode != null)
        this.parentNode.removeChild(this)
    })
  },
  //返回原先的zepto对象，如果返回false可终止遍历，返回this使得可链式调用
  each: function (callback) {
    emptyArray.every.call(this, function (el, idx) {
      return callback.call(el, idx, el) !== false
    });
    return this
  },
  //过滤，如果参数是函数，返回执行为true的子元素；如果是字符串，那么调用zepto.matches来判断当前元素是否匹配相应选择器
  filter: function (selector) {
    if (isFunction(selector)) return this.not(this.not(selector));
    return $(filter.call(this, function (element) {
      return zepto.matches(element, selector)
    }))
  },
  //调用$(selector, context)来获取符合条件的集合元素，然后合并成数组去重并包装成zepto对象
  add: function (selector, context) {
    return $(uniq(this.concat($(selector, context))))
  },
  //判断当前元素集合中的第一个元素是否符css选择器
  is: function (selector) {
    return this.length > 0 && zepto.matches(this[0], selector)
  },
  //排除集合里满足条件的记录，接收参数为: css选择器, function, dom, nodeList
  not: function (selector) {
    var nodes = [];
    //判断selector是否为函数，遍历执行函数返回false，则push到数组中(safari下的typeof nodeList也是function，所以需要判断selector.call !== undefined)
    if (isFunction(selector) && selector.call !== undefined)
      this.each(function (idx) {
        if (!selector.call(this, idx)) nodes.push(this)
      });
    else {
      //当selector为字符串的时候，对集合进行筛选，也就是筛选出集合中满足selector的记录
      var excludes = typeof selector == 'string' ? this.filter(selector) :
        //如果为元素集合，那么调用slice.call得到数组，否则调用zepto构造函数
        (likeArray(selector) && isFunction(selector.item)) ? slice.call(selector) : $(selector);
      this.forEach(function (el) {
        if (excludes.indexOf(el) < 0) nodes.push(el)
      })
    }
    return $(nodes)
  },
  //判断当前对象集合的子元素是否有符合选择器的元素，或者是否包含指定的DOM节点，如果有，则返回新的对象集合
  has: function (selector) {
    return this.filter(function () {
      return isObject(selector) ?
        $.contains(this, selector) :
        $(this).find(selector).size()
    })
  },
  //从当前对象集合中获取给定索引值(以0为基数)，只取一个值(slice(-1)取倒数第一个)
  eq: function (idx) {
    return idx === -1 ? this.slice(idx) : this.slice(idx, +idx + 1)
  },
  //获取当前对象集合中的第一个元素
  first: function () {
    var el = this[0];
    return el && !isObject(el) ? el : $(el)
  },
  //获取对象集合中最后一个元素
  last: function () {
    var el = this[this.length - 1];
    return el && !isObject(el) ? el : $(el)
  },
  //在当前上下文中查找符合selector的对象
  find: function (selector) {
    var result, $this = this;
    //如果没有选择器，那么返回空zepto对象
    if (!selector) result = $();
    //如果selector为dom元素或者zepto对象时，那么返回在上下文dom元素中selector的dom元素数组
    else if (typeof selector == 'object')
      result = $(selector).filter(function () {
        var node = this;
        return emptyArray.some.call($this, function (parent) {
          return $.contains(parent, node)
        })
      });
    //如果是长度为1的zepto对象，那么直接调用zepto.qsa查询相应dom元素下符合selector条件的dom元素，再转成zepto对象
    else if (this.length == 1) result = $(zepto.qsa(this[0], selector));
    //如果是长度大于1的zepto对象，那么分别查询不同dom元素下符合selector条件的dom元素
    else result = this.map(function () {
        return zepto.qsa(this, selector)
      });
    return result;
  },
  //从元素本身开始，逐级向上级元素匹配，并返回最先匹配selector的元素。如果给定context节点参数，那么只匹配该节点的后代元素
  closest: function (selector, context) {
    var nodes = [], collection = typeof selector == 'object' && $(selector);
    this.each(function (_, node) {
      //如果selector是对象，那么封装成zepto对象并判断node及其祖先节点是否存在内部；否则调用zepto.matches判断node及其祖先节点是否匹配选择器
      while (node && !(collection ? collection.indexOf(node) >= 0 : zepto.matches(node, selector)))
        node = node !== context && !isDocument(node) && node.parentNode;
      //如果该节点和其所有祖先节点都不匹配，那么node==false，不会被push进去(nodes.indexOf(node)<0用来移除重复节点)
      if (node && nodes.indexOf(node) < 0) nodes.push(node)
    });
    return $(nodes)
  },
  //获取对象集合每个元素所有的祖先元素。如果css选择器参数给出，过滤出符合条件的元素
  parents: function (selector) {
    var ancestors = [], nodes = this;
    //现将所有祖先节点放到ancestors数组中
    while (nodes.length > 0)
      nodes = $.map(nodes, function (node) {
        if ((node = node.parentNode) && !isDocument(node) && ancestors.indexOf(node) < 0) {
          ancestors.push(node);
          return node
        }
      });
    //然后再过滤掉不符合选择器的节点
    return filtered(ancestors, selector)
  },
  //获取对象集合中每个元素的直接父元素。如果css选择器参数给出。过滤出符合条件的元素
  parent: function (selector) {
    //获取对象的所有dom元素父节点，然后去重，再过滤掉不匹配选择器的元素
    return filtered(uniq(this.pluck('parentNode')), selector)
  },
  //获得每个匹配元素集合元素的直接子元素，如果给定selector，那么返回的结果中只包含符合css选择器的元素
  children: function (selector) {
    return filtered(this.map(function () {
      return children(this)
    }), selector)
  },
  //获得每个匹配元素集合元素的子元素，包括文字和注释节点
  contents: function () {
    return this.map(function () {
      //contentDocument: 取得子窗口的 document 对象
      return this.contentDocument || slice.call(this.childNodes)
    })
  },
  //获取对象集合中所有元素的兄弟节点。如果给定CSS选择器参数，过滤出符合选择器的元素
  siblings: function (selector) {
    //先获取dom节点父元素，那么获取该父元素所有的子元素，过滤掉当前元素，再进行选择器匹配
    return filtered(this.map(function (i, el) {
      return filter.call(children(el.parentNode), function (child) {
        return child !== el
      })
    }), selector)
  },
  //清空对象集合中每个元素的DOM内容
  empty: function () {
    return this.each(function () {
      this.innerHTML = ''
    })
  },
  //获取对象集合中每一个元素的属性值
  pluck: function (property) {
    return $.map(this, function (el) {
      return el[property]
    })
  },
  //恢复对象集合中每个元素默认的“display”值。如果你用hide将元素隐藏，用该属性可以将其显示。相当于去掉了display：none
  show: function () {
    return this.each(function () {
      //清除内联样式display="none"
      this.style.display == "none" && (this.style.display = '');
      //Window.getComputedStyle()获取最终样式，可查找伪元素样式；如果最终样式display为none，那么设置将display设置为默认
      if (getComputedStyle(this, '').getPropertyValue("display") == "none")
        this.style.display = defaultDisplay(this.nodeName)
    })
  },
  //用给定的内容替换所有匹配的元素
  replaceWith: function (newContent) {
    //现在元素之前插入新的匹配元素，然后移除this下的dom元素
    return this.before(newContent).remove();
  },
  //在每个匹配的元素外层包上一个html元素(structure参数可以是一个单独的元素或者一些嵌套的元素。也可以是一个html字符串片段或者dom节点。还可以是一个生成用来包元素的回调函数，这个函数返回前两种类型的包裹片段)
  wrap: function (structure) {
    var func = isFunction(structure);
    if (this[0] && !func)
      var dom = $(structure).get(0),
        clone = dom.parentNode || this.length > 1;

    return this.each(function (index) {
      $(this).wrapAll(
        func ? structure.call(this, index) :
          clone ? dom.cloneNode(true) : dom
      )
    })
  },
  //在所有匹配元素外面包一个单独的结构，结构可以是单个元素或几个嵌套的元素
  wrapAll: function (structure) {
    if (this[0]) {
      //包裹内容插入到第一个元素前
      $(this[0]).before(structure = $(structure));
      var children;
      //取包裹内容里的第一个子元素的最里层
      while ((children = structure.children()).length) structure = children.first();
      //将当期zepto对象对应的dom元素全部插入到最里层元素
      $(structure).append(this)
    }
    return this
  },
  //将每个元素中的内容包裹在一个单独的结构中
  wrapInner: function (structure) {
    var func = isFunction(structure);
    return this.each(function (index) {
      var self = $(this), contents = self.contents(),
        dom = func ? structure.call(this, index) : structure;
      //如果没有内容文本，那么直接将dom元素放到zepto对应dom元素中；如果有，则将内容文本节点放到dom中，并将dom插入到文本节点前面(wrapAll实现)
      contents.length ? contents.wrapAll(dom) : self.append(dom)
    })
  },
  //移除集合中每个元素的直接父节点，并把他们的子元素保留在原来的位置
  unwrap: function () {
    //获取直接父节点并遍历，用父节点的直接子元素替换掉父节点达到移除每个元素的直接父节点目的
    this.parent().each(function () {
      $(this).replaceWith($(this).children())
    });
    return this
  },
  //通过深度克隆来复制集合中的所有元素
  clone: function () {
    return this.map(function () {
      //cloneNode(true):如果为true，那么这个节点的子节点同时被拷贝；否则只能拷贝当前特定节点
      return this.cloneNode(true)
    })
  },
  //通过设置css的属性display为none来将对象集合中的元素隐藏
  hide: function () {
    return this.css("display", "none")
  },
  //显示或隐藏匹配元素
  toggle: function (setting) {
    return this.each(function () {
      var el = $(this);
      //如果没有给定setting，那么获取display属性作为setting，根据setting判断显示还是隐藏
      (setting === undefined ? el.css("display") == "none" : setting) ? el.show() : el.hide()
    })
  },
  //获取对象集合中每一个元素的前一个兄弟节点，通过选择器来进行过滤
  prev: function (selector) {
    //根据元素的previousElementSibling属性来获取前一个兄弟节点
    return $(this.pluck('previousElementSibling')).filter(selector || '*')
  },
  //获取对象集合中每一个元素的下一个兄弟节点(可以选择性的带上过滤选择器)
  next: function (selector) {
    //根据元素的nextElementSibling属性来获取下一个兄弟节点；如果为null，会在zepto.matches被排除
    return $(this.pluck('nextElementSibling')).filter(selector || '*')
  },
  //获取或设置对象集合中元素的HTML内容(通过innerHTML读内容,append()写内容)
  html: function (html) {
    return 0 in arguments ?
      //当给定content参数时，用其替换对象集合中每个元素的内容
      this.each(function (idx) {
        var originHtml = this.innerHTML;
        $(this).empty().append(funcArg(this, html, idx, originHtml))
      }) :
      //当没有给定content参数时，返回对象集合中第一个元素的innerHtml
      (0 in this ? this[0].innerHTML : null)
  },
  //获取或者设置所有对象集合中元素的文本内容
  text: function (text) {
    return 0 in arguments ?
      this.each(function (idx) {
        var newText = funcArg(this, text, idx, this.textContent);
        this.textContent = newText == null ? '' : '' + newText
      }) :
      (0 in this ? this.pluck('textContent').join("") : null)
  },
  //读取第一个dom元素属性或设置所有dom元素属性(attr(name)、attr(name, value)、attr(name, function(index, oldValue){ ... })、attr({ name: value, name2: value2, ... }))
  attr: function (name, value) {
    var result;
    return (typeof name == 'string' && !(1 in arguments)) ?
      (0 in this && this[0].nodeType == 1 && (result = this[0].getAttribute(name)) != null ? result : undefined) :
      this.each(function (idx) {
        if (this.nodeType !== 1) return;
        if (isObject(name)) for (key in name) setAttribute(this, key, name[key])
        else setAttribute(this, name, funcArg(this, value, idx, this.getAttribute(name)))
      })
  },
  //移除当前对象集合中所有元素的指定属性
  removeAttr: function (name) {
    return this.each(function () {
      this.nodeType === 1 && name.split(' ').forEach(function (attribute) {
        //这里的this是dom节点，因为forEach绑定了this值(这里没有指定attribute的值，会将这个属性移除掉)
        setAttribute(this, attribute)
      }, this)
    })
  },
  //读取或设置dom元素的属性值(中文api: 它在读取属性值的情况下优先于attr，因为这些属性值会因为用户的交互发生改变，如checked和selected)
  prop: function (name, value) {
    //优先读取修正属性，DOM的两字母属性都是驼峰格式
    name = propMap[name] || name;
    return (1 in arguments) ?
      this.each(function (idx) {
        this[name] = funcArg(this, value, idx, this[name])
      }) :
      //这里是直接获得dom上的属性，而不是调用getAttribute方法
      (this[0] && this[0][name])
  },
  //从集合的每个DOM节点中删除一个属性(值得注意的是如果尝试删除DOM的一些内置属性，如className或maxLength，将不会有任何效果，因为浏览器禁止删除这些属性)
  removeProp: function (name) {
    name = propMap[name] || name;
    //这是用JavaScript的delete操作符，删除dom元素的属性
    return this.each(function () {
      delete this[name]
    })
  },
  //读取或写入dom的 data-* 自定义属性
  data: function (name, value) {
    //改成连字符格式(aTcF->a-tc-f)
    var attrName = 'data-' + name.replace(capitalRE, '-$1').toLowerCase();
    var data = (1 in arguments) ?
      this.attr(attrName, value) :
      this.attr(attrName);
    //如果是设置自定义属性，那么返回this，即当前zepto对象，deserializeValue(data)返回的还是this
    return data !== null ? deserializeValue(data) : undefined
  },
  //获取或设置匹配元素的值
  val: function (value) {
    //当给定value参数，那么将设置所有元素的value
    if (0 in arguments) {
      if (value == null) value = "";
      return this.each(function (idx) {
        this.value = funcArg(this, value, idx, this.value)
      })
    } else {
      //当没有给定value参数，返回第一个元素的值。如果是<select multiple>标签，则返回一个包含被选中的option值的数组(this[0].multiple:多选select)
      return this[0] && (this[0].multiple ?
          $(this[0]).find('option').filter(function () {
            return this.selected
          }).pluck('value') :
          this[0].value)
    }
  },
  //读/写坐标  距离文档document的偏移值
  offset: function (coordinates) {
    //写入坐标
    if (coordinates) return this.each(function (index) {
      var $this = $(this),
        //如果coordinates是函数，执行函数
        coords = funcArg(this, coordinates, index, $this.offset()),
        //取父元素坐标
        parentOffset = $this.offsetParent().offset(),
        //计算出合理的坐标(相对于父元素的偏移量-父元素的偏移量=相对于document的偏移量)
        props = {
          top: coords.top - parentOffset.top,
          left: coords.left - parentOffset.left
        };
      //修正postin  static->relative
      if ($this.css('position') == 'static') props['position'] = 'relative';
      //写入样式
      $this.css(props)
    });
    //读取坐标
    if (!this.length) return null;
    /**
     * document.documentElement !== this[0]：解决$('html').offset()返回width为undefined的bug；
     * 调用getBoundingClientRect方法之前判断this[0]是否在元素上，存在该方法
     * */
    if (document.documentElement !== this[0] && !$.contains(document.documentElement, this[0]))
      return {top: 0, left: 0};
    //读取到元素相对于页面视窗的位置
    var obj = this[0].getBoundingClientRect();
    //window.pageYOffset就是类似Math.max(document.documentElement.scrollTop||document.body.scrollTop)
    return {
      left: obj.left + window.pageXOffset,
      top: obj.top + window.pageYOffset,
      width: Math.round(obj.width),
      height: Math.round(obj.height)
    }
  },
  //读取或设置DOM元素的css属性(当value参数不存在的时候，返回对象集合中第一个元素的css属性。当value参数存在时，设置对象集合中每一个元素的对应css属性)
  css: function (property, value) {
    //只有一个传参，读
    if (arguments.length < 2) {
      var element = this[0];
      if (typeof property == 'string') {
        if (!element) return;
        //如果设置了行内样式，直接读取，否则读取计算样式(没有考虑计算样式!important情况)
        return element.style[camelize(property)] || getComputedStyle(element, '').getPropertyValue(property)
      } else if (isArray(property)) {
        //如果property是数组，则返回一个属性对象
        if (!element) return;
        var props = {};
        var computedStyle = getComputedStyle(element, '');
        $.each(property, function (_, prop) {
          props[prop] = (element.style[camelize(prop)] || computedStyle.getPropertyValue(prop))
        });
        return props
      }
    }
    //有多个传参，写
    var css = '';
    if (type(property) == 'string') {
      //null、undefined、false时，删掉样式
      if (!value && value !== 0)
        this.each(function () {
          this.style.removeProperty(dasherize(property))
        });
      else
      //dasherize是将字符串转换成css属性(background-color格式)，并根据value类型可能加上'px'
        css = dasherize(property) + ":" + maybeAddPx(property, value)
    } else {
      //如果property是对象的话，遍历处理
      for (key in property)
        if (!property[key] && property[key] !== 0)
          this.each(function () {
            this.style.removeProperty(dasherize(key))
          });
        else
          css += dasherize(key) + ':' + maybeAddPx(key, property[key]) + ';'
    }
    return this.each(function () {
      //这里并没有去重，但是并没有影响
      this.style.cssText += ';' + css;
    })
  },
  //获取一个元素的索引值
  index: function (element) {
    //如果没有给出element参数，那么获取this的父元素，然后获取所有子元素，并判断第一个元素出现的位置；如果给出了element，返回它在当前对象集合中的位置
    return element ? this.indexOf($(element)[0]) : this.parent().children().indexOf(this[0])
  },
  //检查对象集合中是否有元素含有指定的class
  hasClass: function (name) {
    //如果没有传参数，则直接返回false
    if (!name) return false;
    return emptyArray.some.call(this, function (el) {
      //这里this为classRE(name)，name被转成正则表达式，这里使用正则因为className(el)可能是空格隔开的多个class组成
      return this.test(className(el))
    }, classRE(name))
  },
  //为每个匹配的元素添加指定的class类名。多个class类名使用空格分隔
  addClass: function (name) {
    if (!name) return this;
    return this.each(function (idx) {
      //处理非dom元素，继续下一个元素(没有return false)
      if (!('className' in this)) return;
      classList = [];
      //className(this)：获取当前dom元素的className；处理name为函数情况
      var cls = className(this), newName = funcArg(this, name, idx, cls);
      //根据newName的空格分成数组，分别判断当前元素是否存在；如果不存在则添加到数组中
      newName.split(/\s+/g).forEach(function (klass) {
        if (!$(this).hasClass(klass)) classList.push(klass)
      }, this);
      //设置dom元素的className
      classList.length && className(this, cls + (cls ? " " : "") + classList.join(" "))
    })
  },
  //移除当前对象集合中所有元素的指定class。如果没有指定name参数，将移出所有的class。多个class参数名称可以利用空格分隔
  removeClass: function (name) {
    return this.each(function (idx) {
      if (!('className' in this)) return;
      if (name === undefined) return className(this, '');
      classList = className(this);
      //使用正则表达式移除相应klass
      funcArg(this, name, idx, classList).split(/\s+/g).forEach(function (klass) {
        classList = classList.replace(classRE(klass), " ")
      });
      className(this, classList.trim())
    })
  },
  //在匹配的元素集合中的每个元素上添加或删除一个或多个样式类。如果class的名称存在则删除它，如果不存在，就添加它
  toggleClass: function (name, when) {
    if (!name) return this;
    return this.each(function (idx) {
      var $this = $(this), names = funcArg(this, name, idx, className(this));
      names.split(/\s+/g).forEach(function (klass) {
        (when === undefined ? !$this.hasClass(klass) : when) ?
          $this.addClass(klass) : $this.removeClass(klass)
      })
    })
  },
  //读写元素 滚动条的垂直偏移
  scrollTop: function (value) {
    if (!this.length) return;
    var hasScrollTop = 'scrollTop' in this[0];
    if (value === undefined) return hasScrollTop ? this[0].scrollTop : this[0].pageYOffset;
    return this.each(hasScrollTop ?
      function () {
        this.scrollTop = value
      } :
      function () {
        this.scrollTo(this.scrollX, value)
      })
  },
  //读写元素 滚动条的水平偏移
  scrollLeft: function (value) {
    if (!this.length) return;
    var hasScrollLeft = 'scrollLeft' in this[0];
    if (value === undefined) return hasScrollLeft ? this[0].scrollLeft : this[0].pageXOffset;
    return this.each(hasScrollLeft ?
      function () {
        this.scrollLeft = value
      } :
      function () {
        this.scrollTo(value, this.scrollY)
      })
  },
  //获取对象集合中第一个元素的位置(获取相对父元素的坐标)
  position: function () {
    if (!this.length) return;
    //获取第一个元素
    var elem = this[0],
      //获取第一个定位过的祖先元素
      offsetParent = this.offsetParent(),
      //对象集合中第一个元素的offset
      offset = this.offset(),
      parentOffset = rootNodeRE.test(offsetParent[0].nodeName) ? {top: 0, left: 0} : offsetParent.offset();
    // Subtract element margins(offset减去margin影响的值，再减去父元素的偏移量和边框宽度就等于相对于父元素的坐标)
    // note: when an element has margin: auto the offsetLeft and marginLeft are the same in Safari causing offset.left to incorrectly be 0
    offset.top -= parseFloat($(elem).css('margin-top')) || 0;
    offset.left -= parseFloat($(elem).css('margin-left')) || 0;
    //加上父元素的边框
    parentOffset.top += parseFloat($(offsetParent[0]).css('border-top-width')) || 0;
    parentOffset.left += parseFloat($(offsetParent[0]).css('border-left-width')) || 0;
    // Subtract the two offsets
    return {
      top: offset.top - parentOffset.top,
      left: offset.left - parentOffset.left
    }
  },
  //找到第一个定位过的祖先元素(读取父元素中第一个其position设为relative或absolute的可见元素)
  offsetParent: function () {
    return this.map(function () {
      var parent = this.offsetParent || document.body;
      while (parent && !rootNodeRE.test(parent.nodeName) && $(parent).css("position") == "static")
        parent = parent.offsetParent;
      return parent
    })
  }
};

// for now
$.fn.detach = $.fn.remove;
//width:获取对象集合中第一个元素的宽或者设置对象集合中所有元素的宽；height:获取对象集合中第一个元素的高度或者设置对象集合中所有元素的高度
['width', 'height'].forEach(function (dimension) {
  //width、height=>Width、Height
  var dimensionProperty =
    dimension.replace(/./, function (m) {
      return m[0].toUpperCase()
    });
  //$.fn.width、$.fn.height
  $.fn[dimension] = function (value) {
    //获取对象集合中第一个元素
    var offset, el = this[0];
    //window用innerWidth、innerHeight获取
    if (value === undefined) return isWindow(el) ? el['inner' + dimensionProperty] :
      //document，用scrollWidth、scrollHeight获取
      isDocument(el) ? el.documentElement['scroll' + dimensionProperty] :
        //否则用offsetWidth、offsetHeight
        (offset = this.offset()) && offset[dimension];
    else return this.each(function (idx) {
      el = $(this);
      //如果value是函数，那么执行'arg.call(context, idx, payload)'，el[dimension]()为原先值；否则直接返回value
      el.css(dimension, funcArg(this, value, idx, el[dimension]()))
    })
  }
});
//递归遍历 node 的子节点，将节点交由回调函数 fun 处理
function traverseNode(node, fun) {
  fun(node);
  for (var i = 0, len = node.childNodes.length; i < len; i++)
    traverseNode(node.childNodes[i], fun)
}

//生成after、prepend、before、append、insertAfter、insertBefore、appendTo、prependTo方法
adjacencyOperators.forEach(function (operator, operatorIndex) {
  var inside = operatorIndex % 2; //=> prepend, append
  /**
   * after:在每个匹配的元素后插入内容(外部插入)
   * prepend:将参数内容插入到每个匹配元素的前面(内部插入)
   * before:在匹配每个元素的前面插入内容(外部插入)
   * append:在每个匹配的元素末尾插入内容(内部插入)
   * */
  $.fn[operator] = function () {
    //参数可以是节点数组、zepto对象或者html片段
    var argType, nodes = $.map(arguments, function (arg) {
        var arr = [];
        argType = type(arg);
        //如果是dom节点数组、zepto对象数组、html片段数组或者其混合状态
        if (argType == "array") {
          arg.forEach(function (el) {
            if (el.nodeType !== undefined) return arr.push(el);
            else if ($.zepto.isZ(el)) return arr = arr.concat(el.get());
            arr = arr.concat(zepto.fragment(el))
          });
          return arr
        }
        //如果是对象或者null直接返回；否则生成dom元素并返回
        return argType == "object" || arg == null ?
          arg : zepto.fragment(arg)
      }),
      //如果zepto对象内dom元素大于1,需要克隆里面的元素，拿克隆的元素来进行操作，否则会出现多次操作同一元素，前面的操作会被覆盖无效
      parent, copyByClone = this.length > 1;
    //为0，不需要操作，直接返回
    if (nodes.length < 1) return this;
    //遍历zepto对象下的dom元素
    return this.each(function (_, target) {
      //inside 1(内部插入)：prepend, append取自身；inside 0(外部插入)：after, before取父元素
      parent = inside ? target : target.parentNode;
      //0:after、1:prepend、2:before、3:append
      target = operatorIndex == 0 ? target.nextSibling :
        operatorIndex == 1 ? target.firstChild :
          operatorIndex == 2 ? target :
            null;
      //父元素是否在document中
      var parentInDocument = $.contains(document.documentElement, parent);
      //遍历待插入的元素(一个疑问：nodes可能会二维数组，没有对这种情况进行判断处理)
      nodes.forEach(function (node) {
        //拷贝需要添加的节点，避免多次操作同一元素，前面的操作会被覆盖无效
        if (copyByClone) node = node.cloneNode(true);
        //定位元素不存在，没法执行插入操作，直接删除，返回
        else if (!parent) return $(node).remove();
        //insertBefore(newnode,existingnode):newnode 需要插入的节点对象；existingnode 在其之前插入新节点的子节点，如果未规定会在结尾插入
        parent.insertBefore(node, target);
        //如果父元素在document里，则调用traverseNode来处理node节点及node节点的所有子节点
        if (parentInDocument) traverseNode(node, function (el) {
          //脚本通过insertBefore的方法插入到dom中时，是不会执行脚本的，所以需要使用eval来进行处理。
          if (el.nodeName != null && el.nodeName.toUpperCase() === 'SCRIPT' &&
            (!el.type || el.type === 'text/javascript') && !el.src) {
            /**
             * ownerDocument:返回的是元素的根节点，也即document对象；defaultView返回的是document对象所关联的window对象；
             * 这里主要是处理iframe里的script，因为在iframe中有独立的window对象；如果不存在该属性，则默认使用当前的window对象
             * */
            var target = el.ownerDocument ? el.ownerDocument.defaultView : window;
            //最后调用window的eval方法，执行script中的脚本，脚本用el.innerHTML取得
            target['eval'].call(target, el.innerHTML)
          }
        })
      })
    })
  };
  //生成after=>insertAfter、prepend=>prependTo、before=>insertBefore、append=>appendTo方法(简单地反向调用对应的方法)
  $.fn[inside ? operator + 'To' : 'insert' + (operatorIndex ? 'Before' : 'After')] = function (html) {
    $(html)[operator](this);
    return this
  }
});
```
### 二、源码分析
#### 1、prop & attr
prop 是直接读取获取设置 dom 元素上的属性值，删除使用 delete；
attr 是调用 setAttribute、getAttribute 和 removeAttribute 方法。
#### 2、css
读取 dom.style 上的属性值(行内样式)或者调用 getComputedStyle(element, '').getPropertyValue(property) 方法(计算样式)；设置 css 值时设置的是 dom.style.cssText，不用去重，就算重复了也没关系，后面的属性值会覆盖前面的。
#### 3、after、prepend、before、append、insertAfter、insertBefore、appendTo、prependTo
after、prepend、before、append 巧妙的用了 parent.insertBefore(node, target) 方法来统一处理；insertAfter、insertBefore、appendTo、prependTo 巧妙的反向调用对应的方法
#### 4、cloneNode(true)
拷贝节点副本，为了避免同时添加一个元素多次，后面的操作会覆盖前面的(设置 true 为深拷贝，同时拷贝其子节点)
#### 5、wrap、wrapAll、wrapInner
```
原文
<ul>
  <li title='苹果'>苹果</li>
  <li title='橘子'>橘子</li>
  <li title='菠萝'>菠萝</li>
</ul>
 
1、$("li").wrap("<div></div>");
每一个选择器都添加
<ul>
  <div><li title="苹果">苹果</li></div>
  <div><li title="橘子">橘子</li></div>
  <div><li title="菠萝">菠萝</li></div>
</ul>
 
2、$("li").wrapAll("<div></div>");
在所有选中的选择器最外面添加
<ul>
  <div>
    <li title="苹果">苹果</li>
    <li title="橘子">橘子</li>
    <li title="菠萝">菠萝</li>
  </div>
</ul>
 
3、$("li").wrapInner("<div></div>");
为选择器的内容添加
<ul>
  <li title='苹果'><div>苹果</div></li>
  <li title='橘子'><div>橘子</div></li>
  <li title='菠萝'><div>菠萝</div></li>
</ul>
```
这几个方法有点绕，看上面的实例就知道了。