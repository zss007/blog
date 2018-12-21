---
title: node 之 ee-first
categories:
- node
---
捕获多个 event emitter 的多个事件触发，一旦某个事件被触发，则清除所有的回调函数。
<!--more-->
### 一、源码
```js
/**
 * Module exports.
 * @public
 */
module.exports = first

/**
 * Get the first event in a set of event emitters and event pairs.
 *
 * @param {array} stuff
 * @param {function} done
 * @public
 */
function first(stuff, done) {
  if (!Array.isArray(stuff))
    throw new TypeError('arg must be an array of [ee, events...] arrays')

  var cleanups = []

  for (var i = 0; i < stuff.length; i++) {
    var arr = stuff[i]

    if (!Array.isArray(arr) || arr.length < 2)
      throw new TypeError('each array member must be [ee, events...]')

    var ee = arr[0]

    for (var j = 1; j < arr.length; j++) {
      var event = arr[j]
      var fn = listener(event, callback)

      // listen to the event
      ee.on(event, fn)
      // push this listener to the list of cleanups
      cleanups.push({
        ee: ee,
        event: event,
        fn: fn,
      })
    }
  }

  function callback() {
    cleanup()
    done.apply(null, arguments)
  }

  function cleanup() {
    var x
    for (var i = 0; i < cleanups.length; i++) {
      x = cleanups[i]
      x.ee.removeListener(x.event, x.fn)
    }
  }

  function thunk(fn) {
    done = fn
  }

  thunk.cancel = cleanup

  return thunk
}

/**
 * Create the event listener.
 * @private
 */
function listener(event, done) {
  return function onevent(arg1) {
    var args = new Array(arguments.length)
    var ee = this
    var err = event === 'error'
      ? arg1
      : null

    // copy args to prevent arguments escaping scope
    for (var i = 0; i < args.length; i++) {
      args[i] = arguments[i]
    }

    done(err, ee, event, args)
  }
}
```
### 二、分析
##### 1、stuff 是一个二维数组，是`[ee, ...event]`的数组形式，done 只会被调用一次。
##### 2、让 ee 监听相应的时间，并添加到 cleanups 数组中。
##### 3、done 调用时参数为 err、ee、event、args，在 listener 中返回的 onevent 函数执行，并同时清空所有 event emitter 挂载的回调函数。
##### 4、返回函数 thunk，挂载 cleanup 函数，调用时清空所有 event emitter 挂载的回调函数。
### 三、案例
```js
var ee1 = new EventEmitter()
var ee2 = new EventEmitter()

var thunk = first([
  [ee1, 'close', 'end', 'error'],
  [ee2, 'error']
], function (err, ee, event, args) {
  // listener invoked
})

// cancel and clean up
thunk.cancel()
```