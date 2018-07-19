#Vue Global API

## 源码
``` js
function initGlobalAPI (Vue) {
  // config
  var configDef = {};
  configDef.get = function () { return config; };
  {
    configDef.set = function () {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      );
    };
  }
  Object.defineProperty(Vue, 'config', configDef);

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  Vue.util = {
    warn: warn,
    extend: extend,
    mergeOptions: mergeOptions,
    defineReactive: defineReactive
  };

  Vue.set = set;
  Vue.delete = del;
  Vue.nextTick = nextTick;

  Vue.options = Object.create(null);
  ASSET_TYPES.forEach(function (type) {
    Vue.options[type + 's'] = Object.create(null);
  });

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue;

  extend(Vue.options.components, builtInComponents);

  initUse(Vue);
  initMixin$1(Vue);
  initExtend(Vue);
  initAssetRegisters(Vue);
}

initGlobalAPI(Vue$3);
```
`initGlobalAPI`
## 全局配置

`Vue.config`是一个对象，包含Vue的全局配置。可以在启动应用之前修改下列属性：
- silent 
  - 类型： boolean
  - 默认值： false
  - 用法：
  ``` js
    Vue.config.silent = true
   ```
   取消 Vue 所有的日志与警告。

- optionMergeStrategies
  - 类型： { [key: string]: Function }
  - 默认值： {}
  - 用法:
  ``` js
  Vue.config.optionMergeStrategies._my_option = function (parent, child, vm) {
    return child + 1
  }
  const Profile = Vue.extend({
    _my_option: 1
  })
  // Profile.options._my_option = 2
  ```
  自定义合并策略的选项。
  合并策略选项分别接受第一个参数作为父实例，第二个参数为子实例，Vue 实例上下文被作为第三个参数传入。

- devtools
  - 类型： `boolean`
  - 默认值： `true` (生产版为 `false`)
  - 用法：
  ``` js
    // 务必在加载 Vue 之后，立即同步设置以下内容
    Vue.config.devtools = true
  ```
  配置是否允许 vue-devtools 检查代码。开发版本默认为 true，生产版本默认为 false。生产版本设为 true 可以启用检查。

- errorHandler 
  - 类型： `Function`
  - 默认值： `undefined`
  - 用法：
  ``` js
    Vue.config.errorHandler = function (err, vm, info) {
    // handle error
    // `info` 是 Vue 特定的错误信息，比如错误所在的生命周期钩子
    // 只在 2.2.0+ 可用
    }
  ```
  指定组件的渲染和观察期间未捕获错误的处理函数。这个处理函数被调用时，可获取错误信息和 Vue 实例。

- warnHandler
  - 类型： `Function`
  - 默认值： `undefined`
  - 用法：
  ``` js
  Vue.config.warnHandler = function (msg, vm, trace) {
    // `trace` 是组件的继承关系追踪
  }
  ```
  为 Vue 的运行时警告赋于一个自定义句柄。注意这只会在开发者环境下生效，在生产环境下它会被忽略。

- ignoredElements 
  - 类型： `Array<string | RegExp>`
  - 默认值： `[]`
  - 用法：
  ``` js
  Vue.config.ignoredElements = [
    'my-custom-web-component',
    'another-web-component',
    // 用一个 `RegExp` 忽略所有“ion-”开头的元素
    // 仅在 2.5+ 支持
    /^ion-/
  ]
  ```
  须使 Vue 忽略在 Vue 之外的自定义元素 (e.g. 使用了 Web Components APIs)。否则，它会假设你忘记注册全局组件或者拼错了组件名称，从而抛出一个关于 Unknown custom element 的警告。


- keyCodes
  - 类型： `{ [key: string]: number | Array<number> }`
  - 默认值： `{}`
  - 用法：
  ``` js
  Vue.config.keyCodes = {
    v: 86,
    f1: 112,
    // camelCase 不可用
    mediaPlayPause: 179,
    // 取而代之的是 kebab-case 且用双引号括起来
    "media-play-pause": 179,
    up: [38, 87]
  }
  ```
  ``` html
  <input type="text" @keyup.media-play-pause="method">
  ```
  给 v-on 自定义键位别名。
  
- performance
  - 类型： boolean
  - 默认值： false
  - 用法：


- productionTip
  - 类型： `boolean`
  - 默认值： `true`
  - 用法：
  设置为 false 以阻止 vue 在启动时生成生产提示。

## 全局 API
``` js
// initGlobalAPI
Vue.config
Vue.util = {
	warn,
	extend,
	mergeOptions,
	defineReactive
}
Vue.set = set
Vue.delete = del
Vue.nextTick = nextTick
Vue.options = {
	components: {
    KeepAlive
    // Transition 和 TransitionGroup 组件在 runtime/index.js 文件中被添加
    // Transition,
    // TransitionGroup
  },
  directives: Object.create(null),
	// 在 runtime/index.js 文件中，为 directives 添加了两个平台化的指令 model 和 show
	// directives:{
	//	model,
  //	show
	// },
	filters: Object.create(null),
  _base: Vue 
}

// global-api/use.js  -> initUse
Vue.use = function (plugin: Function | Object) {}

// global-api/mixin.js -> initMixin
Vue.mixin = function (mixin: Object) {}

// global-api/extend.js -> initExtend
Vue.cid = 0
Vue.extend = function (extendOptions: Object): Function {}

// global-api/assets.js -> initAssetRegisters
Vue.component =
Vue.directive =
Vue.filter = function (
  id: string,
  definition: Function | Object
): Function | Object | void {}

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

Vue.version = '__VERSION__'

// entry-runtime-with-compiler.js
Vue.compile = compileToFunctions
```