# 选项 （options)

## 选项 / 数据

### data
- 类型：`Object | Function`
- 限制：组件的定义只接受 `function`
- 详细：
Vue 实例的数据对象。Vue将会递归将data的属性转换为getter/setter，从而让data的属性能够响应数据变化。对象必须是存纯粹的对象（含有零个或多个key/value对）：浏览器API创建的原生对象，原型上的属性会被忽略。大概来说， data应该只能是数据- 不推荐观察拥有状态行为的对象。

一旦观察过，不需要再次在数据对象上添加响应式属性。因此推荐在实例创建之前，就声明所有的根级响应式属性。
实例创建之后，可以通过vm.$data访问原始数据对象。Vue实例也代理了data对象上所有的属性，因此访问vm.a等价于vm.$data.a

以_h或$开头的属性不会被Vue实例代理，因为它们可能和Vue内置的属性，API方法冲突。你可以使用vm.$data._property的方式访问这些属性

当一个组件被定义，data必须声明为返回一个初始数据对象的函数，因为组件可能被用来创建多个实例。如果data仍然是一个存粹的对象，则所有实例将共享引用同一个数据对象。通过提供data函数，每次创建一个新实例后，我们能够调用data函数，从而返回初始数据的一个全新副本数据对象

如果需要，可以通过将 vm.$data 传入 JSON.parse(JSON.stringify(...)) 得到深拷贝的原始数据对象。

``` js
var data = { a: 1 }
// 直接创建一个实例
var vm = new Vue({
  data: data
})
vm.a // => 1
vm.$data === data // => true
// Vue.extend() 中 data 必须是函数
var Component = Vue.extend({
  data: function () {
    return { a: 1 }
  }
})
```
### props 
- 类型：`Array<string> | Object`
- 详细：props 可以是数组或对象，用于接收来自父组件的数据。props 可以是简单的数组，或者使用对象作为替代，对象允许配置高级选 项，如类型检测、自定义校验和设置默认值。
示例：
``` js
// 简单语法
Vue.component('props-demo-simple', {
  props: ['size', 'myMessage']
})
// 对象语法，提供校验
Vue.component('props-demo-advanced', {
  props: {
    // 检测类型
    height: Number,
    // 检测类型 + 其他验证
    age: {
      type: Number,
      default: 0,
      required: true,
      validator: function (value) {
        return value >= 0
      }
    }
  }
})
```

### propsData
- 类型：`{ [key: string]: any }`
- 限制：只用于 new 创建的实例中。
- 详细：创建实例时传递 props。主要作用是方便测试。
- 示例：
``` js
var Comp = Vue.extend({
  props: ['msg'],
  template: '<div>{{ msg }}</div>'
})
var vm = new Comp({
  propsData: {
    msg: 'hello'
  }
})
```

### computed
- 类型：`{ [key: string]: Function | { get: Function, set: Function } }`
- 详细：计算属性将被混入到 Vue 实例中。所有 getter 和 setter 的 this 上下文自动地绑定为 Vue 实例。
计算属性的结果会被缓存，除非依赖的响应式属性变化才会重新计算。注意，如果实例范畴之外的依赖 (比如非响应式的 not reactive) 是不会触发计算属性更新的。
- 示例：
``` js
var vm = new Vue({
  data: { a: 1 },
  computed: {
    // 仅读取
    aDouble: function () {
      return this.a * 2
    },
    // 读取和设置
    aPlus: {
      get: function () {
        return this.a + 1
      },
      set: function (v) {
        this.a = v - 1
      }
    }
  }
})
vm.aPlus   // => 2
vm.aPlus = 3
vm.a       // => 2
vm.aDouble // => 4
```
### methods
- 类型：`{ [key: string]: Function }`
- 详细：methods 将被混入到 Vue 实例中。可以直接通过 VM 实例访问这些方法，或者在指令表达式中使用。方法中的 this 自动绑定为 Vue 实例。
- 示例：
``` js
var vm = new Vue({
  data: { a: 1 },
  methods: {
    plus: function () {
      this.a++
    }
  }
})
vm.plus()
vm.a // 2
```

### watch
- 类型：`{ [key: string]: string | Function | Object }`
- 详细：一个对象，键是需要观察的表达式，值是对应回调函数。值也可以是方法名，或者包含选项的对象。Vue 实例将会在实例化时调用 $watch()，遍历 watch 对象的每一个属性。
- 示例：
``` js
var vm = new Vue({
  data: {
    a: 1,
    b: 2,
    c: 3,
    d: 4
  },
  watch: {
    a: function (val, oldVal) {
      console.log('new: %s, old: %s', val, oldVal)
    },
    // 方法名
    b: 'someMethod',
    // 深度 watcher
    c: {
      handler: function (val, oldVal) { /* ... */ },
      deep: true
    },
    // 该回调将会在侦听开始之后被立即调用
    d: {
      handler: function (val, oldVal) { /* ... */ },
      immediate: true
    }
  }
})
vm.a = 2 // => new: 2, old: 1
```

## 选项 / DOM

### el
- 类型：`string | HTMLElement`
- 限制：只在由 new 创建的实例中遵守。
- 详细：提供一个在页面上已存在的 DOM 元素作为 Vue 实例的挂载目标。可以是 CSS 选择器，也可以是一个 HTMLElement 实例。
在实例挂载之后，元素可以用 vm.$el 访问。
如果这个选项在实例化时有作用，实例将立即进入编译过程，否则，需要显式调用 vm.$mount() 手动开启编译。

### template
- 类型：`string`
- 详细: 一个字符串模板作为 Vue 实例的标识使用。模板将会 替换 挂载的元素。挂载元素的内容都将被忽略，除非模板的内容有分发插槽。
如果值以 # 开始，则它将被用作选择符，并使用匹配元素的 innerHTML 作为模板。常用的技巧是用 <script type="x-template"> 包含模板。

### render
- 类型：`(createElement: () => VNode) => VNode`
- 详细：字符串模板的代替方案，允许你发挥 JavaScript 最大的编程能力。该渲染函数接收一个 createElement 方法作为第一个参数用来创建 VNode。
如果组件是一个函数组件，渲染函数还会接收一个额外的 context 参数，为没有实例的函数组件提供上下文信息。

### renderError
- 类型：`(createElement: () => VNode, error: Error) => VNode`
- 详细：只在开发者环境下工作。
当 render 函数遭遇错误时，提供另外一种渲染输出。其错误将会作为第二个参数传递到 renderError。这个功能配合 hot-reload 非常实用。
- 示例：
``` js
new Vue({
  render (h) {
    throw new Error('oops')
  },
  renderError (h, err) {
    return h('pre', { style: { color: 'red' }}, err.stack)
  }
}).$mount('#app')
```
## 选项 / 生命周期钩子
所有的生命周期钩子自动绑定 this 上下文到实例中，因此你可以访问数据，对属性和方法进行运算。这意味着 你不能使用箭头函数来定义一个生命周期方法 (例如 created: () => this.fetchTodos())。这是因为箭头函数绑定了父上下文，因此 this 与你期待的 Vue 实例不同，this.fetchTodos 的行为未定义。
### beforeCreate
- 类型：`Function`
- 详细：在实例初始化之后，数据观测 (data observer) 和 event/watcher 事件配置之前被调用。

### created
- 类型：`Function`
- 详细: 在实例创建完成后被立即调用。在这一步，实例已完成以下的配置：数据观测 (data observer)，属性和方法的运算，watch/event 事件回调。然而，挂载阶段还没开始，$el 属性目前不可见

### beforeMount
- 类型：`Function`
- 详细：在挂载开始之前被调用：相关的 render 函数首次被调用.该钩子在服务器端渲染期间不被调用。

### mounted
- 类型：`Function`
- 详细：el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用该钩子。如果 root 实例挂载了一个文档内元素，当 mounted 被调用时 vm.$el 也在文档内。
注意 mounted 不会承诺所有的子组件也都一起被挂载。如果你希望等到整个视图都渲染完毕，可以用 vm.$nextTick 替换掉 mounted：
``` js
mounted: function () {
  this.$nextTick(function () {
    // Code that will run only after the
    // entire view has been rendered
  })
}
```
### beforeUpdate
类型：`Function`
详细: 数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁之前。你可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程。该钩子在服务器端渲染期间不被调用。

### updated
- 类型：`Function`
- 详细：
由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。
当这个钩子被调用时，组件 DOM 已经更新，所以你现在可以执行依赖于 DOM 的操作。然而在大多数情况下，你应该避免在此期间更改状态。如果要相应状态改变，通常最好使用计算属性或 watcher 取而代之。
注意 updated 不会承诺所有的子组件也都一起被重绘。如果你希望等到整个视图都重绘完毕，可以用 vm.$nextTick 替换掉 updated：
``` js
updated: function () {
  this.$nextTick(function () {
    // Code that will run only after the
    // entire view has been re-rendered
  })
}
```
### activated
- 类型：`Function`
- 详细：`keep-alive` 组件激活时调用。
该钩子在服务器端渲染期间不被调用。
### deactivated
- 类型：`Function`
- 详细：keep-alive 组件停用时调用。
该钩子在服务器端渲染期间不被调用。

### beforeDestroy
- 类型：`Function`
- 详细：实例销毁之前调用。在这一步，实例仍然完全可用。
该钩子在服务器端渲染期间不被调用。

### destroyed
- 类型：`Function`
- 详细: Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。
该钩子在服务器端渲染期间不被调用。

### errorCaptured
- 类型：`(err: Error, vm: Component, info: string) => ?boolean`
- 详细：当捕获一个来自子孙组件的错误时被调用。此钩子会收到三个参数：错误对象、发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回 false 以阻止该错误继续向上传播。

## 选项 / 资源

### directives
- 类型：`Object`
- 详细：包含 Vue 实例可用指令的哈希表。

### filters
- 类型：`Object`
- 详细：包含 Vue 实例可用过滤器的哈希表。

### components
- 类型：`Object`
- 详细：包含 Vue 实例可用组件的哈希表。

## 选项 / 组合
### parent
- 类型：`Vue instance`
- 详细：指定已创建的实例之父实例，在两者之间建立父子关系。子实例可以用 this.$parent 访问父实例，子实例被推入父实例的 $children 数组中。

### mixins
- 类型：`Array<Object>`

- 详细：
mixins 选项接受一个混合对象的数组。这些混合实例对象可以像正常的实例对象一样包含选项，他们将在 Vue.extend() 里最终选择使用相同的选项合并逻辑合并。举例：如果你混合包含一个钩子而创建组件本身也有一个，两个函数将被调用。
Mixin 钩子按照传入顺序依次调用，并在调用组件自身的钩子之前被调用。

### extends
- 类型：`Object | Function`

- 详细：
允许声明扩展另一个组件(可以是一个简单的选项对象或构造函数)，而无需使用 Vue.extend。这主要是为了便于扩展单文件组件。
这和 mixins 类似，区别在于，组件自身的选项会比要扩展的源组件具有更高的优先级。

- 示例：
``` js
var CompA = { ... }
// 在没有调用 `Vue.extend` 时候继承 CompA
var CompB = {
  extends: CompA,
  ...
}
```
### provide / inject
- 类型：
`provide：Object | () => Object`
`inject：Array<string> | { [key: string]: string | Symbol | Object }`
- 详细：
provide 和 inject 主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码中。
这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。如果你熟悉 React，这与 React 的上下文特性很相似。
provide 选项应该是一个对象或返回一个对象的函数。该对象包含可注入其子孙的属性。在该对象中你可以使用 ES2015 Symbols 作为 key，但是只在原生支持 Symbol 和 Reflect.ownKeys 的环境下可工作。

inject 选项应该是一个字符串数组或一个对象，该对象的 key 代表了本地绑定的名称，value 为其 key (字符串或 Symbol) 以在可用的注入中搜索。

## 选项 / 其它
### name
- 类型：string
- 限制：只有作为组件选项时起作用。
- 详细：
允许组件模板递归地调用自身。注意，组件在全局用 Vue.component() 注册时，全局 ID 自动作为组件的 name。

指定 name 选项的另一个好处是便于调试。有名字的组件有更友好的警告信息。另外，当在有 vue-devtools，未命名组件将显示成 <AnonymousComponent>，这很没有语义。通过提供 name 选项，可以获得更有语义信息的组件树。

### delimiters
- 类型：`Array<string>`

- 默认值：`["{{", "}}"]`

- 限制：这个选项只在完整构建版本中的浏览器内编译时可用。

详细：改变纯文本插入分隔符。
``` js
new Vue({
  delimiters: ['${', '}']
})
```

### functional
- 类型：boolean

- 详细：使组件无状态 (没有 data ) 和无实例 (没有 this 上下文)。他们用一个简单的 render 函数返回虚拟节点使他们更容易渲染。
 
## model
- 类型：{ prop?: string, event?: string }

- 详细：允许一个自定义组件在使用 v-model 时定制 prop 和 event。默认情况下，一个组件上的 v-model 会把 value 用作 prop 且把 input 用作 event，但是一些输入类型比如单选框和复选框按钮可能像使用 value prop 来达到不同的目的。使用 model 选项可以回避这些情况产生的冲突。

``` js
Vue.component('my-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    // this allows using the `value` prop for a different purpose
    value: String,
    // use `checked` as the prop which take the place of `value`
    checked: {
      type: Number,
      default: 0
    }
  },
  // ...
})
```
``` html
<my-checkbox v-model="foo" value="some value"></my-checkbox>
```
``` html
<my-checkbox
  :checked="foo"
  @change="val => { foo = val }"
  value="some value">
</my-checkbox>
```

### inheritAttrs

- 类型：boolean

- 默认值：true

- 详细：
默认情况下父作用域的不被认作 props 的特性绑定 (attribute bindings) 将会“回退”且作为普通的 HTML 特性应用在子组件的根元素上。当撰写包裹一个目标元素或另一个组件的组件时，这可能不会总是符合预期行为。通过设置 inheritAttrs 到 false，这些默认行为将会被去掉。而通过 (同样是 2.4 新增的) 实例属性 $attrs 可以让这些特性生效，且可以通过 v-bind 显性的绑定到非根元素上。

注意：这个选项不影响 class 和 style 绑定。

### comments
- 类型：boolean

- 默认值：false

- 限制：这个选项只在完整构建版本中的浏览器内编译时可用。

- 详细：当设为 true 时，将会保留且渲染模板中的 HTML 注释。默认行为是舍弃它们。