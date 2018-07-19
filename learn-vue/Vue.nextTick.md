#Vue.nextTick([callback, context])

- 参数
  - {Function} [callback]
  - {Object} [context]
- 用法：

在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

``` js
// 修改数据
vm.msg = 'Hello'
// DOM 还没有更新
Vue.nextTick(function () {
  // DOM 更新了
})
```
> 2.1.0 起新增：如果没有提供回调且在支持 Promise 的环境中，则返回一个 Promise。请注意 Vue 不自带 Promise 的 polyfill，所以如果你的目标浏览器不原生支持 Promise (IE：你们都看我干嘛)，你得自己提供 polyfill。

``` js
/**
 * Defer a task to execute it asynchronously.
 */
var nextTick = (function () {
  var callbacks = [];
  var pending = false;
  var timerFunc;

  function nextTickHandler () {
    pending = false;
    var copies = callbacks.slice(0);
    callbacks.length = 0;
    for (var i = 0; i < copies.length; i++) {
      copies[i]();
    }
  }

  // An asynchronous deferring mechanism.
  // In pre 2.4, we used to use microtasks (Promise/MutationObserver)
  // but microtasks actually has too high a priority and fires in between
  // supposedly sequential events (e.g. #4521, #6690) or even between
  // bubbling of the same event (#6566). Technically setImmediate should be
  // the ideal choice, but it's not available everywhere; and the only polyfill
  // that consistently queues the callback after all DOM events triggered in the
  // same loop is by using MessageChannel.
  if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
    timerFunc = function () {
      setImmediate(nextTickHandler)
    }
  } else if (typeof MessageChannel !== 'undefined' && (isNative(MessageChannel) ||  
    MessageChannel.toString() === '[object MessageChannelConstructor]'
  )) {
    var channel = new MessageChannel()
    var port = channel.port2
    channel.port.onmessage = nextTickHandler
    timerFunc = function () {
      port.portMessage(1)
    }
  } else {
    if (typeof Promise !== 'undefined' && isNative(Promise)) {
      // use microtask in non-DOM environments, e.g. Weex
      var p = Promise.resolve()
      timeFunc = function () {
        p.then(nextTickHandler)
      }
    }
  } else {
    // fallback to setTimeout
    timerFunc = function () {
      setTimeout(nextTickHandler, 0)
    }
  }

  return function queueNextTick (cb, ctx) {
    var _resolve
    callbacks.push(function () {
      if (cb) {
        try {
          cb.call(ctx)
        } catch (e) {
          handlerError(e, ctx, 'nextTick')
        }
      } else if (_resolve) {
        _resolve(ctx)
      }
    })
    if (!pending) {
      pending = true
      timeFunc()
    }
    if (!cb && typeof Promise !== 'undefined') {
      return new Promise(function (resolve, reject) {
        _resolve = resolve
      })
    }
  }
})()