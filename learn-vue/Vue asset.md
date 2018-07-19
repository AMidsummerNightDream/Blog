## Vue.component( id, [definition] )
- 参数：
  - {string} id
  - {Function | Object} [definition]
- 用法：
注册或获取全局组件。注册还会自动使用给定的id设置组件的名称

``` js
// 注册组件，传入一个扩展过的构造器
Vue.component('my-component', Vue.extend({ /* ... */ }))
// 注册组件，传入一个选项对象 (自动调用 Vue.extend)
Vue.component('my-component', { /* ... */ })
// 获取注册的组件 (始终返回构造器)
var MyComponent = Vue.component('my-component')
```
## Vue.directive( id, [definition] )
- 参数：
  - {string} id
  - {Function | Object} [definition]
- 用法：
注册或获取全局指令。

``` js
// 注册
Vue.directive('my-directive', {
  bind: function () {},
  inserted: function () {},
  update: function () {},
  componentUpdated: function () {},
  unbind: function () {}
})
// 注册 (指令函数)
Vue.directive('my-directive', function () {
  // 这里将会被 `bind` 和 `update` 调用
})
// getter，返回已注册的指令
var myDirective = Vue.directive('my-directive')
```

## Vue.filter( id, [definition] )

- 参数：
  - {string} id
  - {Function} [definition]
- 用法：
注册或获取全局过滤器。

``` js
// 注册
Vue.filter('my-filter', function (value) {
  // 返回处理后的值
})
// getter，返回已注册的过滤器
var myFilter = Vue.filter('my-filter')
```
## 源码

``` js
var ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
/**
 * Create asset registration methods
 */
ASSET_TYPES.forEach(function (type) {
  Vue[type] = function (id, definition) {
    if (!definition) {
      return this.options[type + 's'][id]
    } else {
      if (type === 'component' && config.isReservedTag(id)) {
        warn(
          'Do not use built-in or reserved HTML elements as component ' +
          'id: ' + id
        )
      }
      if (type === 'component' && isPlainObject(definition)) {
        definition.name = definition.name || id
        definition = this.options._base.extend(definition)
      }
      if (type === 'directive' && typeof definition === 'function') {
        definition = { bind: definition, update: definition }
      }
      this.options[type + 's'][id] = definition
      return definition
    }
  }
})
```