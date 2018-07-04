# 插件
Vue提供了插件机制，可以在全局添加一些功能。它们可以简单到几个方法，属性，也可以很复杂，比如一整套组件库。本章将介绍几个官方的核心插件，然后通过实战开发一个插件。
注册插件需要一个公开的方法install，它的第一个参数是Vue构造器，第二个参数是一个可选对象
``` js
MyPlugin.install = function(Vue, options) {
  //全局注册组件（指令等功能资源类似）
  Vue.component('component-name', {
    //组件内容
  })
  //添加实例方法
  Vue.prototype.$Notice = function () {
    // 逻辑
  }
  //添加全局方法或属性
  Vue.globalMethod = function () {
    // 逻辑
  }
  // 添加全局混合
  Vue.mixin({
    mounted: function () {
      //逻辑
    }
  })
}
```

通过Vue.use（）来使用插件：
``` js
Vue.use(MyPlugin)
//
Vue.use(MyPlugin, {
  //参数
})
```
绝大多数情况下，开发插件主要是通过NPM发布后给别人使用的，在自己的项目中可以直接在入口调用以上方法，无须多一步注册和使用的步骤。

## 前端路由与vue-router
### 什么是前端路由
上一章介绍webpack时提到它的主要使用场景是单页面富应用（SPA），而SPA的核心就是前端路由。那什么是路由呢？通俗的讲，就是网址，再专业一点，就是每次GET或者POST等请求在服务端有一个专门的正则配置列表，然后匹配到具体的一条路径后，分发到不同的Controller，进行各种操作，最终将和html或数据返回给前端，这就完成了一次IO。
当然，目前绝大多数的网站都是这种后端路由，也就是多页面的，这样的好处有很多，比如页面可以在服务端渲染好直接返回给浏览器，不用等待前端加载任何js和css就可以直接显示网页内容，再比如对SEO的友好等。后端路由的缺点也是很明显的，就是模版是由后端来维护或改写的。前端开发者需要安装整套的后端服务，必须时还要学习像PHP或Java这些非前端语言来改写html结构，所以html和数据，逻辑混为一谈，维护起来即臃肿又麻烦。
然后有了前后端分离的开发模式，后端只是提供API来返回数据，前端通过Ajax获取数据后，再用一定的方式渲染到页面里，这么做的优点就是前后端做的事情分得很清楚，后端专注在数据上，前端专注在交互和可视化上，如果今后再开发移动端APP，那就正好能使用一套API了。当然，缺点也很明显，首先就是首屏渲染需要时间来加载css和js，这种开发模式被很多公司认同，也出现了很多前端技术栈，比如jQuery+artTemplate+Seajs(requirejs)+gulp为主的开发模式。在Node.js出现后，这种现象有了改善，就是所谓的大前端，得益于Node.js和JavaScript的语言特点，html模版完全由前端来控制，同步或异步渲染完全由前端自行决定，并且由前端维护一套模版，这就是为什么在服务端使用artTemplate，React以及Vue 2的原因。SPA，在前后端分离的基础上，加一层前端路由。
前端路由，即由前端来维护一个路由规则。实现有两种，一种是利用url的hash，就是常说的锚点（#），JavaScript通过hashChange事件来监听url的改变，IE7及以下需要用轮询；另一种就是HTML5的History模式，它使url看起来像普通网站那样以“/”分割，没有“#”，但页面并没有跳转，不过使用这种模式需要服务端支持，服务端在接收到所有请求后，都指向同一个html文件，不然会出现404，因此，SPA只有一个html，整个网站所有内容都在这一个html里，通过JavaScript处理。
前端路由的优点有很多，比如页面持久性，像大部分音乐网站，你都可以在播放歌曲的同时跳转到别的页面，而音乐没有中断。再比如前后端彻底分离。前端路由的框架通用的有Director，不过更多还是结合具体框架来用，比如Angular的ngRouter，React的ReactRouter，以及本节要介绍的Vue的vue-router
如果要独立开发一个前端路由，需要考虑到页面的可插拔，页面的生命周期，内存管理等问题。

Routers里面每一项的path属性就是指定当前匹配的路径，component是映射的组件，上例的写法，webpack会把每一个路由都打包为一个js文件，在请求该页面时，才去加载这个页面的js，也就是异步实现的懒加载（按需加载）。这样做的好处是不需要在打开首页时就把所有页面的内容全部加载进来，只有访问时才加载。如果非要一次性加载


``` js
const Routers = [
  {
    path: '/index',
    component: (resolve) => require(['./views/index.vue'], resolve)
  },
  {
    path: '/about',
    component: (resolve) => require(['./views/about.vue'], resolve)
  }
];

```
``` js
{
  path '/index',
  component: require('./views/index.vue')
}
```
使用了异步路由后，编译出的每个页面的js都叫做chunk（块），它们命名默认是0.main.js,1.main.js...可以在webpack配置的出口output里通过设置chunkFilename字段修改chunk命名
``` js
output: {
  publicPath: '/dist/',
  filename: '[name].js',
  chunkFilename: '[name].chunk.js'
}
```
有了chunk后，在每个页面（.vue）文件里写的样式也需要配置后才会打包进main.css，否则仍然会通过JavaScript动态创建<·style>标签的形式写入。
``` js
\\ webpack.config.js
plugins: [
  new ExtractTextPlugin({
    filename: '[name].css',
    allChunks: true
  })
]
```
运行网页时，router-view会根据当前路由动态渲染不同的页面组件。网页中一些公共部分，比如顶部导航栏，侧边导航栏，底部的版权信息，这些也可以直接写在app.vue里，与router-view同级。路由切换时，切换的是router-view挂载的组件，其它的内容并不会发生变化。

### 高级用法
在SPA项目中，如何修改网页的标题？
网页标题是通过title来显示的，但是SPA只有一个固定的html，切换到不同页面时，标题并不会变化，但是可以通过JavaScript修改title的内容
``` js
window.document.title = '要修改的标题'
```
那么问题就来了，在Vue工程里，在哪里，在什么时候修改标题呢？比较容易想到的一个办法是，在每个.vue文件里，通过mounted钩子修改。这种办法没有问题，但是页面多了维护起来会很麻烦，而且这些逻辑都是重复的。
比较理想的一个思路是，在页面发生路由改变时，统一设置。vue-router提供栏导航钩子beforeEach和afterEach，它们会在路由即将改变前和改变后触发，所以设置标题可以在beforeEach钩子完成。

``` js
const router = new VueRouter(RouterConfig);
router.beforeEach((to, from, next) => {
  window.document.title = to.meta.title;
  next();
});
```
导航钩子有3个参数
to： 即将要进入的目标路由对象
from： 当前导航即将要离开的路由对象
next： 调用该方法后，才能进入下一个钩子
路由列表的meta字段可以自定义一些信息，比如我们将每个页面的title写入了meta来同意维护，beforeEach钩子可以从路由对象to里获取meta信息，从而改变标题
有了这两个钩子，还能做很多事情来提升用户体验。比如一个页面较长，滚动到某个位置，再跳转到另一个页面，滚动条默认是在上一个页面停留的位置，而好的体验肯定是能返回顶端。通过afterEach钩子就可以实现
``` js
router.afterEach((to, from, next) => {
  window.scrollTo(0, 0);
})
```
类似需求还有，从一个页面过渡到另一个页面时，可以出现一个全局的Loading动画，等到新页面加载完成后再结束动画
next（）方法还可以设置参数，比如下面的场景
某些页面需要校验是否登陆，如果登录了就可以访问，否则跳转到登陆页。这里我们通过localStorage来简易判断是否登陆，示例代码如下

``` js
router.beforeEach((to, from, next) => {
  if (window.localStorage.getItem('token')) {
    next();
  } else {
    next('/login');
  }
});
```
next()函数参数设置为false时，可以取消导航，设置为具体的路径可以导航到指定的页面
正确地使用好导航钩子可以方便实现一些全局的功能，而且便于维护。更多的可能需要在业务中不断探索
## 状态管理与Vuex
### 状态管理与使用场景
Vuex所解决的问题与bus类似，它作为Vue的一个插件来使用，可以更好地管理和维护整个项目的组件状态。

一个组件可以分为数据（model）和视图（view），数据更新时，视图也会自动更新。在视图中可以绑定一些事件，它们触发methods里指定的方法，从而可以改变数据，更新视图，这是一个组件基本的运行模式。

在实际业务中，经常有跨组件共享数据的需求，因此Vuex的设计就是用来统一管理组件状态的，它定义了一系列规范来使用和操作数据，使用组件应用更加高效。

使用Vuex会有一定的门槛和复杂性，它的主要使用场景是大型单页应用，适合多人协同开发。如果你的项目不是很复杂，或者希望短期内见效，你需要认真考虑是否真的有必要使用Vuex，
每一个框架诞生都是用来解决具体问题的。虽然bus可能很好地解决了跨组件通信，但它在数据管理，维护，架构设计上还只是一个简单的组件，而Vuex却能更优雅和高效地完成状态管理

