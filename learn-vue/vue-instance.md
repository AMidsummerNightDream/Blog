# Vue 实例的设计

``` js
// Vue.prototype._init
vm._uid = uid++
//避免Vue实例对象被观测(observed)
vm._isVue = true 
vm.$options
vm._renderProxy = vm  // 渲染函数作用域代理
vm._self = vm

// src/core/instance/lifecycle.js  -> initLifecycle(vm)
vm.$parent = parent
vm.$root = parent ? parent.$root : vm

vm.$children = []
vm.$refs = {}

vm._watcher = null
vm._inactive = null
vm._directInactive = false
vm._isMounted = false
vm._isDestroyed = false
vm._isBeingDestroyed = false

// src/core/instance/events.js  -> initEvents(vm)
vm._events = Object.create(null)
vm._hasHookEvent = false

// src/core/instance/render.js   -> initRender(vm)
vm._vnode = null // the root of the child tree
vm._staticTrees = null // v-once cached trees

vm.$vnode
vm.$slots
vm.$scopedSlots

vm._c
vm.$createElement

vm.$attrs
vm.$listeners

// src/core/instance/state.js  -> initState(vm)
vm._watchers = []
vm._data

// src/core/instance/lifecycle.js -> mountComponent()
vm.$el

// src/core/instance/state.js -> initComputed()
vm._computedWatchers = Object.create(null)

// src/core/instance/state.js -> initProps()
vm._props = {}

// src/core/instance/inject.js -> initProvide()
vm._provided
```
## 实例属性
- vm.$options
  - 类型：`Object`
  - 只读
  - 详细：
  用于当前 Vue 实例的初始化选项。需要在选项中包含`自定义属性`时会有用处：
  ``` js
  new Vue({
    customOption: 'foo',
    created: function () {
      console.log(this.$options.customOption) // => 'foo'
    }
  })
  ```
- vm.$parent
  - 类型：`Vue instance`
  - 只读
  - 详细：
  父实例，如果当前实例有的话。

- vm.$root
  - 类型：`Vue instance`
  - 只读
  - 详细：
  当前组件树的根 Vue 实例。如果当前实例没有父实例，此实例将会是其自己。


- vm.$children
  - 类型：`Array<Vue instance>`
  - 只读
  - 详细：
  用于当前 Vue 实例的初始化选项。需要在选项中包含`自定义属性`时会有用处：
  当前实例的直接子组件。需要注意 $children 并不保证顺序，也不是响应式的。如果你发现自己正在尝试使用 $children 来进行数据绑定，考虑使用一个数组配合 v-for 来生成子组件，并且使用 Array 作为真正的来源。

- vm.$refs
  - 类型：`Object`
  - 只读
  - 详细：
  一个对象，持有已注册过 ref 的所有子组件。

- vm.$slots
  - 类型：`{ [name: string]: ?Array<VNode> }`
  - 只读
  - 详细：
  用来访问被插槽分发的内容。每个具名插槽 有其相应的属性 (例如：slot="foo" 中的内容将会在 vm.$slots.foo 中被找到)。default 属性包括了所有没有被包含在具名插槽中的节点。
  在使用渲染函数书写一个组件时，访问 vm.$slots 最有帮助。

  ``` html
  <blog-post>
  <h1 slot="header">
    About Me
  </h1>
  <p>Here's some page content, which will be included in vm.$slots.default, because it's not inside a named slot.</p>
  <p slot="footer">
    Copyright 2016 Evan You
  </p>
  <p>If I have some content down here, it will also be included in vm.$slots.default.</p>.
</blog-post>
```
``` js
Vue.component('blog-post', {
  render: function (createElement) {
    var header = this.$slots.header
    var body   = this.$slots.default
    var footer = this.$slots.footer
    return createElement('div', [
      createElement('header', header),
      createElement('main', body),
      createElement('footer', footer)
    ])
  }
})
```
- vm.$scopedSlots
  - 类型：`{ [name: string]: props => VNode | Array<VNode> }`
  - 只读
  - 详细：
  用来访问作用域插槽。对于包括 默认 slot 在内的每一个插槽，该对象都包含一个返回相应 VNode 的函数。
  vm.$scopedSlots 在使用渲染函数开发一个组件时特别有用。

- vm.$attrs
  - 类型：`{ [key: string]: string }`
  - 只读
  - 详细：
  包含了父作用域中不被认为 (且不预期为) props 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 props 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件——在创建更高层次的组件时非常有用

- vm.$listeners
  - 类型：`{ [key: string]: Function | Array<Function> }`
  - 只读
  - 详细：
  包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用。

- vm.$el
  - 类型：`HTMLElement`
  - 只读
  - 详细：
  Vue 实例使用的根 DOM 元素。


- vm.$data
  - 类型：`Object`
  - 详细：
  Vue 实例观察的数据对象。Vue 实例代理了对其 data 对象属性的访问。

- vm.$props
  - 类型：`Object`
  - 详细：
  当前组件接收到的 props 对象。Vue 实例代理了对其 props 对象属性的访问。

- vm.$isServer
  - 类型：boolean
  - 只读
  - 详细：当前 Vue 实例是否运行于服务器。

## 实例方法 / 事件
### vm.$on( event, callback )
- 参数：
  - `{string | Array<string>} event` (数组只在 2.2.0+ 中支持)
  - `{Function} callback`
- 用法：监听当前实例上的自定义事件。事件可以由vm.$emit触发。回调函数会接收所有传入事件触发函数的额外参数。
- 示例：
``` js
vm.$on('test', function (msg) {
  console.log(msg)
})
vm.$emit('test', 'hi')
```

### vm.$once( event, callback )
- 参数：
  - `{string} event`
  - `{Function} callback`
- 用法：监听一个自定义事件，但是只触发一次，在第一次触发之后移除监听器。

### vm.$off( [event, callback] )
- 参数：
  - `{string | Array<string>}` event (只在 2.2.2+ 支持数组)
  - `{Function} [callback]`
- 用法：

移除自定义事件监听器。

如果没有提供参数，则移除所有的事件监听器；

如果只提供了事件，则移除该事件所有的监听器；

如果同时提供了事件与回调，则只移除这个回调的监听器。

### vm.$emit( event, […args] )
- 参数：
  - `{string} event`
  - `[...args]`
触发当前实例上的事件。附加参数都会传给监听器回调。


## 实例方法 / 生命周期

### vm.$mount( [elementOrSelector] )
- 参数：
  - `{Element | string} [elementOrSelector]`
  - `{boolean} [hydrating]`
- 返回值：vm - 实例自身

- 用法：

如果 Vue 实例在实例化时没有收到 el 选项，则它处于“未挂载”状态，没有关联的 DOM 元素。可以使用 vm.$mount() 手动地挂载一个未挂载的实例。

如果没有提供 elementOrSelector 参数，模板将被渲染为文档之外的的元素，并且你必须使用原生 DOM API 把它插入文档中。

这个方法返回实例自身，因而可以链式调用其它实例方法。

``` js
var MyComponent = Vue.extend({
  template: '<div>Hello!</div>'
})
// 创建并挂载到 #app (会替换 #app)
new MyComponent().$mount('#app')
// 同上
new MyComponent({ el: '#app' })
// 或者，在文档之外渲染并且随后挂载
var component = new MyComponent().$mount()
document.getElementById('app').appendChild(component.$el)
```

### vm.$forceUpdate()
用法：迫使 Vue 实例重新渲染。注意它仅仅影响实例本身和插入插槽内容的子组件，而不是所有子组件。

### vm.$nextTick( [callback] )
- 参数：`{Function} [callback]`
- 用法：

将回调延迟到下次 DOM 更新循环之后执行。在修改数据之后立即使用它，然后等待 DOM 更新。它跟全局方法 Vue.nextTick 一样，不同的是回调的 this 自动绑定到调用它的实例上。

2.1.0 起新增：如果没有提供回调且在支持 Promise 的环境中，则返回一个 Promise。请注意 Vue 不自带 Promise 的 polyfill，所以如果你的目标浏览器不是原生支持 Promise (IE：你们都看我干嘛)，你得自行 polyfill。

``` js
new Vue({
  // ...
  methods: {
    // ...
    example: function () {
      // 修改数据
      this.message = 'changed'
      // DOM 还没有更新
      this.$nextTick(function () {
        // DOM 现在更新了
        // `this` 绑定到当前实例
        this.doSomethingElse()
      })
    }
  }
})
```

### vm.$destroy()
- 用法：

完全销毁一个实例。清理它与其它实例的连接，解绑它的全部指令及事件监听器。

触发 beforeDestroy 和 destroyed 的钩子。
在大多数场景中你不应该调用这个方法。最好使用 v-if 和 v-for 指令以数据驱动的方式控制子组件的生命周期。
