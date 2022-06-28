# 为什么 React 的 Diff 算法不采用 Vue 的双端对比算法

### 前言

其实为什么 React 不采用 Vue 的双端对比算法，React 官方已经在源码的注释里已经说明了，我们来看一下 React 官方是怎么说的。

```javascript
function reconcileChildrenArray(
returnFiber: Fiber,
 currentFirstChild: Fiber | null,
 newChildren: Array<*>,
 expirationTime: ExpirationTime,
): Fiber | null {
    // This algorithm can't optimize by searching from boths ends since we
    // don't have backpointers on fibers. I'm trying to see how far we can get
    // with that model. If it ends up not being worth the tradeoffs, we can
    // add it later.

    // Even with a two ended optimization, we'd want to optimize for the case
    // where there are few changes and brute force the comparison instead of
    // going for the Map. It'd like to explore hitting that path first in
    // forward-only mode and only go for the Map once we notice that we need
    // lots of look ahead. This doesn't handle reversal as well as two ended
    // search but that's unusual. Besides, for the two ended optimization to
    // work on Iterables, we'd need to copy the whole set.

    // In this first iteration, we'll just live with hitting the bad case
    // (adding everything to a Map) in for every insert/move.

    // If you change this code, also update reconcileChildrenIterator() which
    // uses the same algorithm.
｝
```

大概的意思就是说：

React 不能通过双端对比进行 diff 算法优化是因为目前 Fiber 上没有设置反向链表，而且想知道就目前这种方案能持续多久，如果目前这种模式不理想的话，那么也可以增加双端对比算法。

即使是双端对比算法，我们也要对这种情况进行优化，我们应该使用 Map 这种数据结构方案去替代原来那种几乎没有什么变化也进行暴力比较的方案。它第一次搜索循环是通过 forward-only 这种模式（就是**只从左向右查找**），（第一次循环可能还没有结束，还有节点没有比对的时候）如果还要继续向前循环查找那么就要通过 Map 这种数据类型了。（就目前这个单向链表的数据结构，如果采用）双端对比查找算法比较难控制它反向查找的，但它确实是一种成功的算法。此外，双端对比算法的实现也在我们的工作迭代当中。

第一次迭代，我们就先将就使用这种不好的方案吧，每次新增/移动都要添加所有的数据到一个 Map 的数据类型对象中。

“we'd need to copy the whole set”，这一句，每一个单词都懂，但就是不知道他想说什么，所以就不翻译了，有知道的大神吗？

**本人水平有限，错漏难免，如有错漏，恳请各位斧正。**

都说“双端对比算法”，那么双端对比算法，到底是怎么样的呢？

我们先来讲 React 中的 diff 算法，然后再讲 Vue3 中的 diff 算法，最后讲一下 Vue2 中的 diff 算法。

最后讲一下为什么 Vue 中不需要使用 Fiber 架构。


### Fiber 的结构

在 React15 以前 React 的组件更新创建虚拟 DOM 和 diff 的过程是不可中断，如果需要更新组件树层级非常深的话，在 diff 的过程会非常占用浏览器的线程，而我们都知道浏览器执行 JavaScript 的线程和渲染真实 DOM 的线程是互斥的，也就是同一时间内，浏览器要么在执行 JavaScript 的代码运算，要么在渲染页面，如果 JavaScript 的代码运行时间过长则会造成页面卡顿。
基于以上原因 React 团队在 React16 之后就改写了整个架构，将原来数组结构的虚拟DOM，改成叫 Fiber 的一种数据结构，基于这种 Fiber 的数据结构可以实现由原来不可中断的更新过程变成异步的可中断的更新。

Fiber 的数据结构主要长成以下的样子

```javascript
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // 作为静态数据结构的属性
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;

  // 用于连接其他Fiber节点形成Fiber树
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;

  // 作为动态的工作单元的属性
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  // 调度优先级相关
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // 指向该fiber在另一次更新时对应的fiber
  this.alternate = null;
}
```

Fiber 主要靠以下属性连成一棵树结构的数据的，也就是 Fiber 链表。

```javascript
// 指向父级Fiber节点
this.return = null;
// 指向子Fiber节点
this.child = null;
// 指向右边第一个兄弟Fiber节点
this.sibling = null;
```




### Fiber 链表的生成



### Reat 的 diff 算法





### Vue3 的 diff 算法





### Vue2 的 diff 算法



### React、Vue2、Vue3 算法实践

