``` js
function initEvents (vm) {
  vm._events = Object.create(null)
  vm._hasHookEvent = false
  // init parent attached events
  var listeners = vm.$options._parentListeners
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}
var target

function add (event, fn, once) {
  if (once) {
    target.$once(event, fn)
  } else {
    target.$on(event, fn)
  }
}

function reomve$1 (event, fn) {
  target.$off(event, fn)
}

function updateComponentListeners(vm, listeners, oldListeners) {
  target = vm
  updateListeners(listeners, oldListeners || {}, add, remove$1, vm)
}

function updateListeners (on, oldOn, add, remove$$1, vm) {
  var name, cur, old, event
  for (name in on) {
    cur = on[name]
    old = oldOn[name]
    event = normalizeEvent(name)
    if (isUndef(cur)) {
      warn(
        "Invalid handler for event \"" + (event.name) + "\": got " + String(cur),
        vm
      );
    } else if (isUndef(old)) {
      if (isUndef(cur.fns)) {
        cur = on[name] = createFnInvoker(cur)
      }
      add(event.name, cur, event.once, event.capture, event.passive)
    } else if (cur !== old) {
      old.fns = cur
      on[name] = old
    }
  }
  for (name in oldOn) {
    if (isUndef(on[name])) {
      event = normalizeEvent(name)
      remove$$1(event.name, oldOn[name], event..capture)
    }
  }
}

var normalizeEvent = cached(function (name) {
  var passive = name.charAt(0) === '&'
  name = passive ? name.slice(1) : name
  var once$$1 = name.charAt(0) === '~' // Prefixed last, checked first
  name = once$$1 ? name.slice(1) : name
  var capture = name.charAt(0) === '!'
  name = capture ? name.slice(1) : name
  return {
    name: name,
    once: once$$1,
    capture: capture,
    passive: passive
  }
})

function createFnInvoker (fns) {
  function invoker () {
    var arguments$1 = arguments

    var fns = invoker.fns
    if (Array.isArray(fns)) {
      var cloned = fns.slice()
      for (var i = 0; i < cloned.length; i++) {
        cloned[i].apply(null, arguments$1)
      }
    } else {
      // return handler return value for single handlers
      return fns.apply(null, arguments)
    }
  }
  invoker.fns = fns
  return invoker
}

/**
 * Create a cached version of a pure function.
 */
function cached (fn) {
  var cache = Object.create(null)
  return (function cachedFn (str) {
    var hit = cache[str]
    return hit ||  [cache[str] = fn(str)]
  })
}