``` js
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})

```
`initGlobalAPI(Vue)`的作用是添加全局API，如set， delete，nextTick，use，mixin，extend，component，directive等
``` js
initGlobalAPI(Vue$3)

function initGlobalAPI (Vue) {
  // config
  var configDef = {}
  configDef.get = function () { return config }
  configDef.set = function () {
    warn('Do not replace the Vue.config object, set individual fields instead.');
  }

  Object.defineProperty(Vue, 'config', configDef)

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  Vue.util = {
    warn: warn,
    extend: extend,
    mergeOptions: mergeOptions,
    defineReactive: defineReactive
  }

  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(function (type) {
    Vue.options[type + 's'] = Object.create(null)
  })

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue;

  extend(Vue.options.components, builtInComponents);

  initUse(Vue);
  initMixin$1(Vue);
  initExtend(Vue);
  initAssetRegisters(Vue);
}
function Vue$3(options) {
  if (!(this instanceof Vue$3)) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initGlobalAPI(Vue$3);

Object.defineProperty(Vue$3.prototype, '$isServer', {
  get: isServerRendering
});

Object.defineProperty(Vue$3.prototype, '$ssrContext', {
  get: function get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
});

Vue$3.version = '2.5.0';
```