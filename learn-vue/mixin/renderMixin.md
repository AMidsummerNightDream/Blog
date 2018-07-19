## renderMixin

``` js
// install runtime convenience helpers
installRenderHelpers(Vue.prototype)

function installRenderHelpers (target) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t =renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode
  target._e = createEmptyVNode
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
}

Vue.prototype.$nextTick = function (fn) {
  return nextTick(fm, this)
}

Vue.prototype._render = function () {
  var vm = this
  var ref = vm.$options
  var render = ref.render
  var _parentVnode = ref._parentVnode

  if (vm._isMounted) {
    // if the parent didn't update, the slot nodes will be the ones from
    // last render. They need to be cloned to ensure "freshness" for this render.
    for (var key in vm.$solts) {
      var slot = vm.$slots[key]
      if (slot._rendered) {
        vm.$slots[key] = cloneVNodes(slot, true /* deep */)
      }
    }
  }

  vm.$scopedSlots = (_parentVnode && _parentVnode.data.scopedSlots) || emptyObject

  // set parent vnode. this allows render functions to have access
  // to the data on the placeholder node.
  vm.$vnode = _parentVnode
  // render self
  var vnode
  try {
    vnode = render.call(vm._renderProxy, vm.$createElement)
  } catch (e) {
    handleError(e, vm, "render")
      // return error render result,
      // or previous vnode to prevent render error causing blank component
    if (vm.$options.renderError) {
      try {
        vnode = vm.$options.renderError.call(vm._renderProxy, vm.$createElement, e);
      } catch (e) {
        handleError(e, vm, "renderError");
        vnode = vm._vnode;
      }
    } else {
      vnode = vm._vnode;
    }
    // return empty vnode in case the render function errored out
    if (!(vnode instanceof VNode)) {
      if ("development" !== 'production' && Array.isArray(vnode)) {
        warn(
          'Multiple root nodes returned from render function. Render function ' +
          'should return a single root node.',
          vm
        )
      }
      vnode = createEmptyVNode();
    }
    // set parent
    vnode.parent = _parentVnode;
    return vnode
  }
}


// optimized shallow clone
// used for static nodes and slot nodes because they may be reused across
// multiple renders, cloning them avoids errors when DOM manipulations rely
// on their elm reference.
function cloneVNode (vnode, deep) {
  var cloned = new VNode(
    vnode.tag,
    vnode.data,
    vnode.children,
    vnode.text,
    vnode.elm,
    vnode.context,
    vnode.componentOptions,
    vnode.asyncFactory
  )
  cloned.ns = vnode.ns
  cloned.isStatic = vnode.isStatic
  cloned.key = vnode.key
  cloned.isComment = vnode.isComment
  cloned.isCloned = true
  if (deep && vnode.children) {
    cloned.children = cloneVNodes(vnode.children)
  }
  return cloned
}

function cloneVNodes (vnodes, deep) {
  var len = vnodes.length
  var res = new Array(len)
  for (var i = 0; i < len; i++) {
    res[i] = cloneVNode(vnodes[i], deep)
  }
  return res
}