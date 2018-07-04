#深入Redux应用架构

从Flux身上我们领略到数据在store， action creator， dispatcher 及 React组件之间单向流动的美妙特性，但在它受到越来越多关注的同时，我们也发现了Flux的一些问题与不足。因此，优化和扩展Flux架构的方案不断涌现，其中不乏许多高质量的作品，入reflux， fluxor等。但是很快，一个后起之秀脱颖而出，短短数月就在Github上收获近万star，他就是Redux。
为什么Redux在短时间内就受到了如此高的追捧？Redux和Flux相比有什么异同？如何从零开始搭建一个Redux应用。

## Redux简介
现在我们就Redux是什么，Redux的三大原则和Redux的核心API开始介绍Redux，并说明Redux如何与React结合使用，以及它在Flux基础上的改变。

### Redux是什么
我们都知道Flux本身既不是库，也不是框架，而是一种应用的架构思想。而Redux呢，它的核心代码可以理解成一个库，但同时也强调与Flux类似的架构思想。

从设计上看，Redux参考了Flux的设计，但是对Flux许多冗余的部分（如dispatcher）做了简化，同时将ELm语言中函数式编程的思想融合其中。

非常有意思的是，Redux是从一个实验开始的，作者Dan Abramov并没有想到Redux会变得如此重要又被广泛使用，他只是为了通过Flux思想解决他的热重载及时间旅行的问题而已。

Redux本身非常简单，它的设计思想与React有异曲同工之妙，均是希望用最少的API实现最核心的功能。

Redux本身只是把自己定位成一个“可预测的状态容器”，

“Redux”本身指redux这个npm包，它提供若干API让我们使用reducer创建store，并能够更新store中的数据或获取store中最新的状态。而“Redux”应用则是指使用了redux这个npm包并结合了视图层（如React）及其它前端应用必备组件（路由库， Ajax请求库）组成的完整类Flux思想的前端应用。

### Redux三大原则
想要理解Redux， 必须要知道Redux设计和使用的三大原则
1.单一数据源
在传统的MVC架构中，我们可以根据需要创建无数个Model，而Model之间可以互相监听，触发事件甚至循环或嵌套触发事件，在这些Redux中都是不允许的。
因为在Redux的思想里， 一个应用永远只有唯一的数据源。我们的第一反应可能是：如果有一个复杂应用，强制要求唯一的数据源岂不是会产生一个特别庞大的JavaScript对象。

实际上，使用单一数据源的好处在于整个应用状态都保存在一个对象中，这样我们可以随时提取整个应用的状态进行持久化（比如实现一个针对整个应用的即时保存功能）。此外，这样的设计也为服务端渲染提供了可能。

至于我们担心的数据源对象过于庞大的问题，可以用Redux提供的工具函数combineReducers解决。
2.状态是只读的
这一点和Flux的思想不谋而合，不同的是在Flux中，因为store没有setter而限制了我们直接修改应用状态的能力，而在Redux中这样的限制被执行得更加测彻底，因为我们压根没有store。
在Redux中，我们并不会自己用代码来定义一个store。取而代之的是，我们定义一个reducer，它的功能是根据当前触发的action对当前应用的状态（state）进行迭代，这里我们并没有直接修改应用的状态，而是返回一份全新的状态
Redux提供的createStore方法会根据reducer生成store。最后，我们可以利用store.dispatch方法来达到修改状态的目的。
3.状态修改均由纯函数完成
这是Redux与Flux在表现上的最大不同。在Flux中，我们在actionCreator里调用AppDispatcher.dispatch方法来触发action，这样不仅有冗余的代码，而且因为直接修改了store中的数据，将导致无法保存每次数据变化前后的状态。
在Redux里，我们通过定义reducer来确定状态的修改，而每一个reducer都是纯函数，者意味着它没有副作用，即接受一定的输入，必定会得到一定的输出。

这样设计的好处不仅在于reducer里对状态的修改变得简单，存粹，可测试，更有意思的是，Redux利用每次返回的状态生成酷炫的时间旅行调试方式，让跟踪每一次因为触发action而改变状态的结果成为了可能。

### Redux核心API
Redux的核心是一个store，这个store由Redux提供的createStore（reducers[, initialState])方法生成。从函数签名看出，要想生成store，必须要传入reducers，同时也可以传入第二个可选参数初始化状态（initialState）
在继续了解createStore之前，让我们先认识下reducers。在上一章介绍Flux时我们说到，Flux的核心思想之一就是不直接修改数据，而是分发一个action来描述发生的改变。那么，在Redux里由谁来修改数据呢？
在Redux里，负责影响action并修改数据的角色就是reducer。reducer本质上是一个函数，其函数签名为reducer（previousState, action)=>newState.可以看出，reducer在处理action的同时，还需要接受一个previousState参数。所以，reducer的职责就是根据previousState和action计算出新的newState。

在实际应用中，reducer在处理previousState时，还需要一个特殊的非空判断。很显然，reducer第一次执行的时候，并没有任何的previousState，而reducer的最终职责是返回一个新的state，因此需要在这种特殊情况下返回一个定义好的initialState：

``` js
// MyReducer.js
const initialState = {
  todos: [],
};

// 我们定义的todos这个reducer在第一次执行的时候，回返回{todos:[]}作为初始化状态
function todos(previousState = initialState, action) {
  switch (action.type) {
    case 'XXX': {
      //
    }
    default:
      return previousState;
  }
}
```
根据Dan Abramov的说法，Redux这个名字就是来源于Reduce + Flux，可见reducer在整个Redux架构中拥有举足轻重的作用。
下面就是Redux中最核心的API--createStore：
``` js
import { createStore } from 'redux';
const store = createStore(reducers);

```
通过createStore方法创建的store是一个对象，它本身又包含4个方法。
1. getState（）：获取store中当前的状态
2. dispatch（action）： 分发一个action，并返回这个action，这是唯一能改变store中数据的方式
3. subscribe（listener）：注册一个监听者，它在store发生变化时被调用。
4. replaceReducer（nextReducer）：更新当前store里的reducer，一般只会在开发模式中调用该方法。

在实际使用中，我们最常用的是getState()和dispatch()这两个方法，至于subscribe（）和replaceReducer（）方法，一般在Redux与某个系统（如React）做桥接的时候使用。

### 与React绑定
前面说到Redux