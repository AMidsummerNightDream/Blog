## markOnce(tree, index, key)

``` js
function markOnce (tree, index, key) {
  markStatic(tree, ("__once__" + index + (key ? ("_" + key) : "")), true)
  return tree
}

function markStatic(tree, key, isOnce) {
  if (Array.isArray(tree)) {
    for (var i = 0; i < tree.length; i++) {
      if (tree[i] && typeof tree[i] !== 'string') {
        markStaticNode(tree[i], (key + "_" + i), isOnce)
      }
    }
  } else {
    markStaticNode(tree, key, isOnce)
  }
}

function markStaticNode (node, key, isOnce) {
  node.isStatic = true
  node.key = key
  node.isOnce = isOnce
}

/**
 * Convert a input value to a number for persistence.
 * If the conversion fails, return original string.
 */
function toNumber (val) {
  var n = parseFloat(val);
  return isNaN(n) ? val : n
}

/**
 * Runtime helper for rendering v-for lists.
 */
function renderList (val, render) {
  var ret, i, l, keys, key
  if (Array.isArray(val) || typeof val === 'string') {
    ret = new Array(val.length)
    for (i = 0, l = val.length; i < l; i++) {
      ret[i] = render(val[i], i)
    }
  } else if (typeof val === 'number') {
    ret = new Array(val)
    for (i = 0; i < val; i++) {
      ret[i] = render(i + 1, i)
    }
  } else if (isObject(val)) {
    keys = Object.keys(val)
    ret = new Array(keys.length)
    for (i = 0, l = keys.length; i < l; i++) {
      key = keys[i]
      ret[i] = render(val[key], key, i)
    }
  }
  if (isDef(ret)) {
    (ret)._isVList = true
  }
  return ret
}

/**
 * Runtime helper for rendering <slot>
 */
function renderSlot (name, fallback, props, bindObject) {
  var scopedSlotFn = this.$scopedSlots[name]
  if (scopedSlotFn) { //scoped slot
    props = props || {}
    if (bindObject) {
      if (!isObject(bindObject)) {
        warn(
          'slot v-bind without argument expects an Object',
          this
        )
      }
      props = extend(extend({}, bindObject), props)
    }
    return scopedSlotFn(props) || fallback
  } else {
    var slotNodes = this.$slots[name]
    // warn duplicate slot usage
    if (slotNodes && slotNodes._rendered ) {
      warn(
        "Duplicate presence of slot \"" + name + "\" found in the same render tree " +
        "- this will likely cause render errors.",
        this
      )
    }
    slotNodes.rendered = true
  }
  return slotNodes || fallback
}

/**
 * Runtime helper for rendering static trees.
 */
function renderStatic (index, isInFor) {
  // static trees can be rendered once and cached on the contructor options
  // so every instance shares the same cached trees
  var renderFns = this.$options.staticRenderFns
  var cached = renderFns.cached || (renderFns.cached = [])
  var tree = cached[index]
  // if has already-rendered static tree and not inside v-for,
  // we can reuse the same tree by doing a shallow clone.
  if (tree && !isInFor) {
    return Array.isArray(tree)
      ? cloneVNodes(tree)
      : cloneVNode(tree)
  }
  // otherwise, render a fresh tree
  tree = cached[index] = renderFns[index].call(this._renderProxy, null, this)
  markStatic(tree, ("__static__" + index), false)
  return tree
}

/**
 * Runtime helper for resolving filters
 */
function resolveFilter (id) {
  return resolveAsset(this.$options, 'filters', id, tree) || identity
}

/**
 * Resolve an asset.
 * This function is used because child instances need access
 * to assets defined in its ancestor chain.
 */
function resolveAsset (options, type, id, warnMissing) {
  if (typeof id !== 'string') {
    return
  }
  var assets = options[type]
  // check local registration variations first
  if (hasOwn(assets, id)) { return assets[id] }
  var camelizeId = camelize(id)
  if (hasOwn(assets, camelizedId)) { return assets[camelizedId] }
  var PascalCaseId = capitalize(camelizedId)
  if (hasOwn(assets, PascalCaseId)) { return assets[PascalCaseId] }
  // fallback to prototype chain
  var res = assets[id] || assets[camelizedId] || assets[PascalCaseId]
  if (warnMissing && !res) {
    warn(
      'Failed to resolve ' + type.slice(0, -1) + ': ' + id,
      options
    )
  }
  return res
}

/**
 * Runtime helper for checking keyCodes from config.
 * exposed as Vue.prototype._k
 * passing in eventKeyName as last argument separately for backwards compat
 */
function checkCodeCodes (eventKeyCode, builtInAlias, eventKeyName) {
  var keyCodes = config.keyCodes[key] || builtInAlias
  if (keyCodes) {
    if (Array.isArray(keyCodes)) {
      return keyCodes.indexOf(eventKeyCode) === -1
    } else {
      return keyCodes !== eventKeyCode
    }
  } else if (eventKeyName) {
    return hyphenate(eventKeyName) !== key
  }
}

/**
 * Runtime helper for merging v-bind="object" into a VNode's data.
 */
function bindObjectProps (data, tag, value, asProp, isSync) {
  if (value) {
    if (!isObject(value)) {
      warn(
        'v-bind without argument expects an Object or Array value',
        this
      )
    } else {
      if (Array.isArray(value)) {
        value = toObject(value)
      }
      var hash
      var loop = function (key) {
        if (key === 'class' || key === 'style' || isReservedAttribute(key)){
          hash = data
        } else {
          var type = data.attrs && data.attrs.type
          hash = asProp || config.mustUseProp(tag, type, key)
            ? data.domProps || (data.domProps = {})
            : data.attrs || (data.attrs = {})
        }
        if (!(key in hash)) {
          hash[key] = value[key]

          if (isSync) {
            var on = data.on || (data.on = {})
            on[("update:" + key)] = function ($event) {
              value[key] = $event
            }
          }
        }
      } 
      for (var key in value) loop ( key )
    }
  }
  return data
}

function createTextVNode (val) {
  return new VNode(undefined, undefined, undefined, String(val))
}

var createEmptyVNode = function (text) {
  if ( text === void 0 ) text = ''

  var node = new VNode()
  node.text = text
  node.isComment = true
  return node
}

function resolveScopedSlots (fns, res) {
  res = res || {}
  for (var i = 0; i < fns.length; i++) {
    if (Array.isArray(fns[i])) {
      resolveScopedSlots(fns[i], res)
    } else {
      res[fns[i].key] = fns[i].fn
    }
  }
  return res
}

function bindObjectListeners (data, value) {
  if (value) {
    if (!isPlainObject(value)) {
      warn(
        'v-on without argument expects an Object value',
        this
      )
    } else {
      var on = data.on = data.on ? extend({}, data.on) : {}
      for (var key in value) {
        var existing = on[key]
        var ours = value[key]
        on[key] = existing ? [].contact(ours, existing) : ours
      }
    }
  }
  return data
}