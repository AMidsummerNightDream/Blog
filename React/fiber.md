## React 16 Fiber源码速览

1. 允许在render函数中返回节点数组和字符串
2. 提供更好的错误处理
3. 支持自定义DOM属性
4. React 16是一次重写，在保持API不变的情况下，将核心架构改为代号为Fiber的异步渲染架构。新架构带来的变化有
  1. 体积减小
  2. 服务端渲染速度大幅提升
  3. 更responsive的界面
Fiber这个架构并不是突然冒出来的。Facebook的工程师在设计React之初就设想未来的UI渲染会是异步的。从setState()的设计和React内部的事务机制可以看出
Fiber架构的代码放在原来的React仓库中，并且可以通过运行时判断来切换新老架构，方便测试和部署。因此Fiber的开发是一个渐进的过程，


### Fiber概念简介

#### reconciler VS renderer
Reconciler就是我们所说的Virtul DOM，用于计算新老View的差异。React 16之前的reconciler叫Stack reconciler。Fiber是React新的reconciler。Renderer则是和平台相关的代码，负责将View的变化渲染到不同的平台上。DOM， Canvas，Native，VR，WebGL等，都有自己的renderer。reconciler是React的核心代码，是各个平台公用的。这次React的reconciler更新到Fiber架构是一次重量级的核心架构的更换。

由reconciler和renderer两个概念引出的是phase的概念。Phase指的是React组件渲染时的阶段。第一阶段是reconciliation，这以阶段做的Fiber的update，然后产出的是effect list（可以想象成将老的View更新到新的状态所需要做的DOM操作列表）。这一阶段是没有副作用的，因此这个过程可以被打断。然后恢复执行。第二阶段是commit阶段。Reconciliation产生的effect list只有在commit之后才会生效，也是真正应用到DOM中。这一阶段往往不会执行太长时间，因此是同步的，这样也避免了组件内视图层结构和DOM不一致。


### Fiber是什么
一个Fiber就是一个POJO对象，代表了组件上需要做的工作。一个React Element可以对应一个或多个Fiber节点

在render函数中创建的ReactElement树在第一次渲染的时候会创建一棵一模一样的Fiber节点树。不同的ReactElement类型对应不同的Fiber节点类型。一个ReactElement的工作就是由它对应的Fiber节点来负责。React 16组件实例，会发现有一个_reactInternalFiber属性指向它对应的Fiber实例

虽然React的代码中其实没有明确的Virtual DOM概念，但Fiber和我们概念中的Virtual DOM树是等价的

Fiber给React的渲染带来了重要的变化。React内部有事务的概念。之前React渲染相关的事务是连续的，一旦开始就会run to completion。现在React的事务则是由一系列Fiber的更新组成的。因此React可以在多个帧中断断续续的更新Fiber，最后commit变化

一个ReactElement可以对应不止一个Fiber。因为Fiber在update的时候，会从原来的Fiber（我们称为current）clone出一个新的Fiber（我们称为alternate）。两个Fiber diff出的变化（side effect）记录在alternate上。所以一个组件在更新时最多会有两个Fiber与其对应，在更新结束后alternate会取代之前的current成为新的current节点

### Fiber节点的数据结构
``` js
{
  tag: TypeOfWork, //fiber的类型
  alternate: Fiber|null, //在fiber更新时克隆出的镜像fiber，对fiber的修改会标记在这个fiber上

  return: Fiber|null,// 指向fiber树中的父节点
  child: Fiber|null, //指向第一个子节点

  sibling: Fiber| null, //指向兄弟节点

  effectTag: TypeOfSideEffect, // side effect类型，下文会介绍
  nextEffect: Fiber | null, // 单链表结构，方便遍历fiber树上有副作用的节点
  pendingWorkPriority: PriorityLevel, // 标记子树上待更新任务的优先级
}
```
在实际的渲染过程中，Fiber节点构成了一棵树。这棵树在数据结构上是通过单链表的形式构成的，Fiber节点上的child和sibling属性分别指向了这个节点的第一个子节点和相邻的兄弟节点。这样就可以遍历整个Fiber树

### TypeOfWork
代表React中不同类型的fiber节点
``` js
{
  InteterminateComponent: 0,
  FunctionalComponent: 1,
  ClassComponent: 2,
  HostRoot: 3, // Root of a host tree. Could be nested inside another node.
  HostPortal: 4, // A subtree. Could be an entry point to a different renderer.
  HostComponent: 5,
  HostText: 6,
  CoroutineComponent: 7,
  CoroutineHandlerPhase: 8,
  YieldComponent: 9,
  Fragment: 10,
}

```
ClassComponent
应用层面的React组件，ClassComponent是一个继承自React.Component类的实例

HostRoot
ReactDOM.render()时的根节点

HostComponent
React中常见的抽象节点，是ClassComponent的组成部分。具体实现取决于React运行的平台。在浏览器环境下就代表DOM节点，可以理解为所谓的虚拟DOM节点。HostComponent中的Host，具体操作逻辑是由Host环境注入的

TypeOfSideEffect
{
  NoEffect: 0,          
  PerformedWork: 1,   
  Placement: 2, // 插入         
  Update: 4, // 更新           
  PlacementAndUpdate: 6, 
  Deletion: 8, // 删除   
  ContentReset: 16,  
  Callback: 32,      
  Err: 64,         
  Ref: 128,          
}

Priority
Priority指的是Fiber中一个work的优先级。这是React源码中的对Priority类型的定义：
{
  NoWork: 0, // No work is pending.
  SynchronousPriority: 1, // For controlled text inputs. Synchronous side-effects.
  TaskPriority: 2, // Completes at the end of the current tick.
  HighPriority: 3, // Interaction that needs to complete pretty soon to feel responsive.
  LowPriority: 4, // Data fetching, or result from updating stores.
  OffscreenPriority: 5, // Won't be visible but do the work in case it becomes visible.
}
我们可以把Priority分为同步和异步两个类别，同步优先级的任务会在当前帧完成，包括SynchronousPriority和TaskPriority。异步优先级的任务则可能在接下来的几个帧中被完成，包括HighPriority、LowPriority以及OffscreenPriority。

### React 16 Fiber源码目录结构
React库的入口、组件的基类ReactComponent、ReactElement.createElement函数等等所有平台公用的代码位于src/isomorphic下。
我们关注的Fiber代码位于src/renderers/shared/fiber下。我们先来看看src/renderers下面有什么：

可以看到src/renderers下的代码就是上文介绍的renderer，分dom、native、art等等平台。那我们再看看src/renderers/shared目录下有什么：
src/renderers/shared其实就是reconciler相关的代码了。可以看到里面有fiber和stack新老两大reconciler（在笔者发文时，Stack reconciler已经完成了它的使命，相关的代码已经被移除了）

这些就是React fiber的核心代码了。Fiber节点的定义在ReactFiber.js中，Fiber的reconciler构造函数在ReactFiberReconciler.js中，Fiber节点的工作流程由ReactFiberBeginWork.js、ReactFiberCommitWork.js和ReactFiberCompleteWork.js组成。Fiber的子节点reconcile逻辑在ReactChildFiber.js中，ReactFiberScheduler.js则是调度相关的逻辑。接下来就让我们通过具体的场景，来分析React 16的源码吧！