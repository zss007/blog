---
title: zepto 之 ajax.js
categories:
- zepto
---
`$.ajax`是 zepto 发送请求的核心方法，`$.get`,`$.post`,`$.jsonp`都是封装了`$.ajax`方法。`$.ajax`将 jsonp 与异步请求的代码格式统一起来，内部主要是先处理url、数据和请求头部，然后新建 XMLHttpRequest 对象发送请求。
<!--more-->
### 一、源码
```
    (function ($) {
        var jsonpID = +new Date(),
            document = window.document,
            key,
            name,
            rscript = /<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi,
            scriptTypeRE = /^(?:text|application)\/javascript/i,
            xmlTypeRE = /^(?:text|application)\/xml/i,
            jsonType = 'application/json',
            htmlType = 'text/html',
            blankRE = /^\s*$/,
            originAnchor = document.createElement('a');

        originAnchor.href = window.location.href;

        //触发自定义事件并且如果阻止默认事件触发返回false
        function triggerAndReturn(context, eventName, data) {
            var event = $.Event(eventName);
            $(context).trigger(event, data);
            return !event.isDefaultPrevented()
        }

        //触发ajax的全局事件
        function triggerGlobal(settings, context, eventName, data) {
            if (settings.global) return triggerAndReturn(context || document, eventName, data)
        }

        //正在ajax请求数量
        $.active = 0;

        //第一次ajax触发时触发,绑定在document上
        function ajaxStart(settings) {
            if (settings.global && $.active++ === 0) triggerGlobal(settings, null, 'ajaxStart')
        }

        //所有ajax都已经停止触发,绑定在document上
        function ajaxStop(settings) {
            if (settings.global && !(--$.active)) triggerGlobal(settings, null, 'ajaxStop')
        }

        //触发beforeSend事件或全局ajaxBeforeSend事件，如果有一个返回false,则取消此次请求；否则触发ajaxSend全局事件
        function ajaxBeforeSend(xhr, settings) {
            var context = settings.context;
            if (settings.beforeSend.call(context, xhr, settings) === false ||
                triggerGlobal(settings, context, 'ajaxBeforeSend', [xhr, settings]) === false)
                return false;

            triggerGlobal(settings, context, 'ajaxSend', [xhr, settings])
        }

        //请求成功调用函数
        function ajaxSuccess(data, xhr, settings, deferred) {
            var context = settings.context, status = 'success';
            settings.success.call(context, data, status, xhr);
            if (deferred) deferred.resolveWith(context, [data, status, xhr]);
            triggerGlobal(settings, context, 'ajaxSuccess', [xhr, settings, data]);
            ajaxComplete(status, xhr, settings)
        }

        //请求失败调用函数 type: "timeout", "error", "abort", "parsererror"
        function ajaxError(error, type, xhr, settings, deferred) {
            var context = settings.context;
            settings.error.call(context, xhr, type, error);
            if (deferred) deferred.rejectWith(context, [xhr, type, error]);
            triggerGlobal(settings, context, 'ajaxError', [xhr, settings, error || type]);
            ajaxComplete(type, xhr, settings)
        }

        //请求完成调用函数 status: "success", "notmodified", "error", "timeout", "abort", "parsererror"
        function ajaxComplete(status, xhr, settings) {
            var context = settings.context;
            settings.complete.call(context, xhr, status);
            triggerGlobal(settings, context, 'ajaxComplete', [xhr, settings]);
            ajaxStop(settings)
        }

        //执行自定义过滤函数
        function ajaxDataFilter(data, type, settings) {
            if (settings.dataFilter == empty) return data;
            var context = settings.context;
            return settings.dataFilter.call(context, data, type)
        }

        // Empty function, used as default callback
        function empty() {
        }

        $.ajaxJSONP = function (options, deferred) {
            if (!('type' in options)) return $.ajax(options)

            var _callbackName = options.jsonpCallback,
                callbackName = ($.isFunction(_callbackName) ?
                        _callbackName() : _callbackName) || ('Zepto' + (jsonpID++)),
                script = document.createElement('script'),
                originalCallback = window[callbackName],
                responseData,
                abort = function (errorType) {
                    $(script).triggerHandler('error', errorType || 'abort')
                },
                xhr = {abort: abort}, abortTimeout

            if (deferred) deferred.promise(xhr)

            $(script).on('load error', function (e, errorType) {
                clearTimeout(abortTimeout)
                $(script).off().remove()

                if (e.type == 'error' || !responseData) {
                    ajaxError(null, errorType || 'error', xhr, options, deferred)
                } else {
                    ajaxSuccess(responseData[0], xhr, options, deferred)
                }

                window[callbackName] = originalCallback
                if (responseData && $.isFunction(originalCallback))
                    originalCallback(responseData[0])

                originalCallback = responseData = undefined
            })

            if (ajaxBeforeSend(xhr, options) === false) {
                abort('abort')
                return xhr
            }

            window[callbackName] = function () {
                responseData = arguments
            }

            script.src = options.url.replace(/\?(.+)=\?/, '?$1=' + callbackName)
            document.head.appendChild(script)

            if (options.timeout > 0) abortTimeout = setTimeout(function () {
                abort('timeout')
            }, options.timeout)

            return xhr
        };

        //包含Ajax请求的默认设置的全局对象
        $.ajaxSettings = {
            // Default type of request
            type: 'GET',
            // Callback that is executed before request
            beforeSend: empty,
            // Callback that is executed if the request succeeds
            success: empty,
            // Callback that is executed the the server drops error
            error: empty,
            // Callback that is executed on request complete (both: error and success)
            complete: empty,
            // The context for the callbacks
            context: null,
            // Whether to trigger "global" Ajax events
            global: true,
            // Transport
            xhr: function () {
                return new window.XMLHttpRequest()
            },
            // MIME types mapping
            // IIS returns Javascript as "application/x-javascript"
            accepts: {
                script: 'text/javascript, application/javascript, application/x-javascript',
                json: jsonType,
                xml: 'application/xml, text/xml',
                html: htmlType,
                text: 'text/plain'
            },
            // Whether the request is to another domain
            crossDomain: false,
            // Default timeout
            timeout: 0,
            // Whether data should be serialized to string
            processData: true,
            // Whether the browser should be allowed to cache GET responses
            cache: true,
            //Used to handle the raw response data of XMLHttpRequest.
            //This is a pre-filtering function to sanitize the response.
            //The sanitized response should be returned
            dataFilter: empty
        };

        //根据媒体类型获取dataType:html,json,scirpt,xml,text等
        function mimeToDataType(mime) {
            if (mime) mime = mime.split(';', 2)[0];
            return mime && ( mime == htmlType ? 'html' :
                    mime == jsonType ? 'json' :
                        scriptTypeRE.test(mime) ? 'script' :
                            xmlTypeRE.test(mime) && 'xml' ) || 'text'
        }

        //将查询参数追加到URL后面
        function appendQuery(url, query) {
            if (query == '') return url;
            return (url + '&' + query).replace(/[&?]{1,2}/, '?')
        }

        // serialize payload and append it to the URL for GET requests
        function serializeData(options) {
            //对于非Get请求。是否自动将data转换为字符串
            if (options.processData && options.data && $.type(options.data) != "string")
                options.data = $.param(options.data, options.traditional);
            //get请求，将序列化的数据追加到url后面
            if (options.data && (!options.type || options.type.toUpperCase() == 'GET' || 'jsonp' == options.dataType))
                options.url = appendQuery(options.url, options.data), options.data = undefined
        }

        $.ajax = function (options) {
            var settings = $.extend({}, options || {}),
                deferred = $.Deferred && $.Deferred(),
                urlAnchor, hashIndex;
            //复制默认配置值到选项中(如果选项中没设置)
            for (key in $.ajaxSettings) if (settings[key] === undefined) settings[key] = $.ajaxSettings[key];

            //触发ajaxStart时间，可在document进行监听
            ajaxStart(settings);

            //判断是否跨域(通过页面url和接口url进行比较,判断ip协议和端口号是否相等)
            if (!settings.crossDomain) {
                urlAnchor = document.createElement('a');
                urlAnchor.href = settings.url;
                //解决ie的hack
                urlAnchor.href = urlAnchor.href;
                settings.crossDomain = (originAnchor.protocol + '//' + originAnchor.host) !== (urlAnchor.protocol + '//' + urlAnchor.host)
            }

            //未设置url，取当前地址栏(如果有hash，截掉hash)
            if (!settings.url) settings.url = window.location.toString();
            if ((hashIndex = settings.url.indexOf('#')) > -1) settings.url = settings.url.slice(0, hashIndex);
            serializeData(settings);

            ///\?.+=\?/.test(settings.url):有xxx.html?a=1?=cccc类似形式，为jsonp
            var dataType = settings.dataType, hasPlaceholder = /\?.+=\?/.test(settings.url);
            if (hasPlaceholder) dataType = 'jsonp';

            //不设置缓存，加时间戳 '_=' + Date.now()
            if (settings.cache === false || (
                    (!options || options.cache !== true) &&
                    ('script' == dataType || 'jsonp' == dataType)
                ))
                settings.url = appendQuery(settings.url, '_=' + Date.now());

            //如果是jsonp,调用$.ajaxJSONP,不走XHR，走script
            if ('jsonp' == dataType) {
                if (!hasPlaceholder)  //判断url是否有类似jsonp的参数
                    settings.url = appendQuery(settings.url,
                        settings.jsonp ? (settings.jsonp + '=?') : settings.jsonp === false ? '' : 'callback=?');
                return $.ajaxJSONP(settings, deferred)
            }

            //媒体类型
            var mime = settings.accepts[dataType],
                headers = {},
                //设置请求头的方法
                setHeader = function (name, value) {
                    headers[name.toLowerCase()] = [name, value]
                },
                //如果URL没协议，读取本地URL的协议
                protocol = /^([\w-]+:)\/\//.test(settings.url) ? RegExp.$1 : window.location.protocol,
                xhr = settings.xhr(),
                nativeSetHeader = xhr.setRequestHeader,
                abortTimeout;

            //将xhr设为只读Deferred对象，不能更改状态
            if (deferred) deferred.promise(xhr);

            //如果没有跨域(x-requested-with XMLHttpRequest 表明是AJax异步;x-requested-with null 表明同步,浏览器工具栏未显示,在后台request可以获取到)
            if (!settings.crossDomain) setHeader('X-Requested-With', 'XMLHttpRequest');
            setHeader('Accept', mime || '*/*');
            if (mime = settings.mimeType || mime) {
                //媒体数据源里对应多个，如 script: 'text/javascript, application/javascript, application/x-javascript',设置最新的写法
                if (mime.indexOf(',') > -1) mime = mime.split(',', 2)[0];
                //对Mozilla的修正
                xhr.overrideMimeType && xhr.overrideMimeType(mime)
            }
            /**
             * Content-Type: 内容类型指定响应的HTTP内容类型。决定浏览器将以什么形式、什么编码读取这个文件. 如果未指定ContentType，默认为TEXT/HTML。
             * 当action为get时候，浏览器用x-www-form-urlencoded的编码方式把form数据转换成一个字串（name1=value1&name2=value2...），然后把这个字串append到
             * url后面，用?分割，加载这个新的url；当action为post时候，浏览器把form数据封装到http body中，然后发送到server。
             * 如果method==get，则请求头部不用设置Content-Type，若method==post，则请求头部的Content-Type默认设置'application/x-www-form-urlencoded'
             * */
            if (settings.contentType || (settings.contentType !== false && settings.data && settings.type.toUpperCase() != 'GET'))
                setHeader('Content-Type', settings.contentType || 'application/x-www-form-urlencoded');

            //设置请求头
            if (settings.headers) for (name in settings.headers) setHeader(name, settings.headers[name])
            xhr.setRequestHeader = setHeader;

            xhr.onreadystatechange = function () {
                /**
                 0：请求未初始化（还没有调用 open()）。
                 1：请求已经建立，但是还没有发送（还没有调用 send()）。
                 2：请求已发送，正在处理中（通常现在可以从响应中获取内容头）。
                 3：请求在处理中；通常响应中已有部分数据可用了，但是服务器还没有完成响应的生成。
                 4：响应已完成；您可以获取并使用服务器的响应了。
                 */
                if (xhr.readyState == 4) {
                    xhr.onreadystatechange = empty;
                    //使用了闭包
                    clearTimeout(abortTimeout);
                    var result, error = false;
                    /**
                     * 根据状态来判断请求是否成功
                     * >=200 && < 300 表示成功
                     * 304 文件未修改 成功
                     * xhr.status == 0 && protocol == 'file:'  未请求，打开的本地文件，非localhost  ip形式
                     * */
                    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304 || (xhr.status == 0 && protocol == 'file:')) {
                        //getResponseHeader:从响应信息中获取指定的http头
                        dataType = dataType || mimeToDataType(settings.mimeType || xhr.getResponseHeader('content-type'));
                        //响应值
                        if (xhr.responseType == 'arraybuffer' || xhr.responseType == 'blob')
                            result = xhr.response;
                        else {
                            result = xhr.responseText;
                            try {
                                /**
                                 * http://perfectionkills.com/global-eval-what-are-the-options/
                                 * sanitize response accordingly if data filter callback provided
                                 * (1,eval)(result)  (1,eval)这是一个典型的逗号操作符，返回最右边的值
                                 * (1,eval)  eval 的区别是:前者是一个值，不可以再覆盖。后者是变量,如var a = 1; (1,a) = 1;会报错；
                                 * (1,eval)(result)  eval(result) 的区别是:前者变成值后，只能读取window域下的变量。而后者，遵循作用域链，从局部变量上溯到window域
                                 * 显然(1,eval)(result)  避免了作用域链的上溯操作，性能更好
                                 * */
                                result = ajaxDataFilter(result, dataType, settings);
                                if (dataType == 'script') (1, eval)(result);
                                else if (dataType == 'xml') result = xhr.responseXML;
                                else if (dataType == 'json') result = blankRE.test(result) ? null : $.parseJSON(result)
                            } catch (e) {
                                error = e
                            }
                            //解析出错，抛出 'parsererror'事件
                            if (error) return ajaxError(error, 'parsererror', xhr, settings, deferred)
                        }
                        //执行success
                        ajaxSuccess(result, xhr, settings, deferred)
                    } else {
                        //如果请求出错,xhr.status = 0 / null 执行abort,其他执行error
                        ajaxError(xhr.statusText || null, xhr.status ? 'error' : 'abort', xhr, settings, deferred)
                    }
                }
            };

            //执行请求前置器,若返回false则中断请求
            if (ajaxBeforeSend(xhr, settings) === false) {
                xhr.abort();
                ajaxError(null, 'abort', xhr, settings, deferred);
                return xhr
            }

            // 这是一个小技巧，默认async为true，若settings里面设置了，则为设置的值
            var async = 'async' in settings ? settings.async : true;
            //准备xhr请求
            xhr.open(settings.type, settings.url, async, settings.username, settings.password);

            //对象包含的属性被逐字复制到XMLHttpRequest的实例(如设置跨域凭证withCredentials)
            if (settings.xhrFields) for (name in settings.xhrFields) xhr[name] = settings.xhrFields[name]

            //设置请求头
            for (name in headers) nativeSetHeader.apply(xhr, headers[name])

            //超时处理：设置了settings.timeout，超时后调用xhr.abort()中断请求
            if (settings.timeout > 0) abortTimeout = setTimeout(function () {
                xhr.onreadystatechange = empty;
                xhr.abort();
                ajaxError(null, 'timeout', xhr, settings, deferred)
            }, settings.timeout);

            // avoid sending empty string (#319)
            xhr.send(settings.data ? settings.data : null);
            return xhr
        };

        //进行可选参数(data/success)校验
        function parseArguments(url, data, success, dataType) {
            if ($.isFunction(data)) dataType = success, success = data, data = undefined;
            if (!$.isFunction(success)) dataType = success, success = undefined;
            return {
                url: url
                , data: data
                , success: success
                , dataType: dataType
            }
        }

        //$.ajax的简写方式 get请求
        $.get = function (/* url, data, success, dataType */) {
            return $.ajax(parseArguments.apply(null, arguments))
        };

        //$.ajax的简写方式 post请求
        $.post = function (/* url, data, success, dataType */) {
            var options = parseArguments.apply(null, arguments);
            options.type = 'POST';
            return $.ajax(options)
        };

        //$.ajax的简写方式 获取JSON数据
        $.getJSON = function (/* url, data, success */) {
            var options = parseArguments.apply(null, arguments);
            options.dataType = 'json';
            return $.ajax(options)
        };

        /**
         * 通过GET Ajax载入远程HTML内容代码并插入至当前的dom元素中
         * 可以使用匹配selector选择器的HTML内容来更新集合,如$('#some_element').load('/foo.html #bar')
         * 如果没有给定CSS选择器，将使用完整的返回文本
         * 在没有选择器的情况下，任何javascript块都会添加。如果带上选择器，匹配选择器内的script将会被删除
         * */
        $.fn.load = function (url, data, success) {
            if (!this.length) return this;
            var self = this, parts = url.split(/\s/), selector,
                options = parseArguments(url, data, success),
                callback = options.success;
            if (parts.length > 1) options.url = parts[0], selector = parts[1];
            // response.replace(rscript, "") 过滤出script标签
            //$('<div>').html(response.replace(rscript, ""))  innerHTML方式转换成DOM
            options.success = function (response) {
                self.html(selector ?
                    $('<div>').html(response.replace(rscript, "")).find(selector)
                    : response);
                //dom操作后执行成功回调函数(注:callback存储的是自定义成功回调函数,options.success后面被重新赋值为ajax回调函数,不会死循环)
                callback && callback.apply(self, arguments)
            };
            $.ajax(options);
            return this
        };

        var escape = encodeURIComponent;

        //序列化
        function serialize(params, obj, traditional, scope) {
            var type, array = $.isArray(obj), hash = $.isPlainObject(obj);
            $.each(obj, function (key, value) {
                type = $.type(value);
                //如果是第二次递归调用的话，需要处理一下key的值
                if (scope) key = traditional ? scope :
                    scope + '[' + (hash || type == 'object' || type == 'array' ? key : '') + ']';
                // 处理[{name:'',value:''},{name:'',value:''}]这种类型
                if (!scope && array) params.add(value.name, value.value);
                // 处理嵌套对象如{'key':[]}或者{'key':{}}
                else if (type == "array" || (!traditional && type == "object"))
                    serialize(params, value, traditional, key);
                else params.add(key, value)
            })
        }

        /**
         * 序列化一个对象，在Ajax请求中提交的数据使用URL编码的查询字符串表示形式。
         * 如果shallow设置为true。嵌套对象不会被序列化，嵌套数组的值不会使用放括号在他们的key上。
         * */
        $.param = function (obj, traditional) {
            var params = [];
            params.add = function (key, value) {
                if ($.isFunction(value)) value = value();
                if (value == null) value = "";
                this.push(escape(key) + '=' + escape(value))
            };
            serialize(params, obj, traditional);
            return params.join('&').replace(/%20/g, '+')
        }
    })(Zepto);
```
### 二、源码分析
#### 1、事件处理
ajaxStart/ajaxStop: 绑定在 document 上，通过全局变量 $.active 变量判断是否触发自定义相应事件；
ajaxBeforeSend: ajax 请求前执行自定义拦截函数，同时执行全局 ajaxBeforeSend 函数；如果同时满足触发 ajaxSend 自定义事件，否则触发自定义 abort 事件，执行 ajaxError 函数；
ajaxSuccess/ajaxError: ajax 成功/失败回调函数，触发相应自定义函数后调用 ajaxComplete 函数；
ajaxComplete: ajax 请求完成回调函数，触发自定义事件，同时触发 ajaxStop 事件。
#### 2、判断是否跨域,确定 settings.crossDomain 的值
调用两次 document.createElement('a') 创建两个 a 标签，一个设置为当前 url 地址，一个设置为接口地址，比较两个 a 标签的端口号和 ip 协议判断是否跨域。
#### 3、清除 ajax 缓存(settings.catch === false)
在 url 后面添加时间戳参数，防止 ajax 请求缓存。
#### 4、ajax 超时处理
ajax 执行时设置一个定时器，时间为超时时间，如果超过则执行 xhr.abort，并且触发 timeout 自定义事件；如果在超时时间内请求成功，则清除定时器，onreadystatechange 函数内部使用了闭包。