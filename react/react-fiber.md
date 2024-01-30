# React Fiber

React Fiber 是对 React 核心算法的不断重新实现，目标是提高 React 在动画，布局和手势等领域的适用性。

React Fiber 的功能包括：

- **支持增量渲染，将渲染工作分成多个块并将其分布到多个帧中的能力**（主要功能）；
- 当出现新的更新时，支持暂停，中止和复用；
- 支持为不同类型的更新分配优先级；
- 新的并发原函数（primitives）。

## Scheduling

> _scheduling_ -- 确定何时应执行 work 的过程。  
> _work_ -- 必须执行的任何计算. Work 通常是更新 (e.g. setState) 的结果.

React 的设计原则文档中提到：

- 在用户界面中，无需立即应用每个更新。实际上，这样做可能是浪费的，导致帧下降并降低用户体验。
- 不同类型的更新具有不同的优先级-动画更新需要比数据存储中的更新更快地完成。
- 基于推送的方法要求应用程序（您，程序员）决定如何安排工作。基于拉的方法使框架（React）变得智能，并为您做出那些决定。

修改 React 的核心算法以利用 Scheduling 是 Fiber 背后的驱动思想。

## Fiber

已经确定，Fiber 的主要目标是使 React 能够利用 scheduling 的优势。具体来说，需要能够

- 暂停 work，稍后再回来
- 为不同类型的 work 分配优先级
- 重用以前完成的 work
- 中止 work，如果不再需要

为了做到这一点，首先需要一种将 work 分解为单元的方法。从某种意义上讲 Fiber 代表 **work 的单元**。

首先了解 React 组件作为数据函数的概念

> React 的核心前提是 UI 只是 data 到 data 的投影。相同的输入给出相同的输出。一个简单的纯函数。

```js
v = fn(d);
```

因此，呈现 React 应用程序类似于调用一个函数，该函数的主体包含对其他函数的调用，并依此类推。当考虑 Fiber 时，这种类比很有用。

计算机通常使用调用堆栈来跟踪程序执行的方式。
执行函数时，新的堆栈帧将添加到堆栈中。
该堆栈帧表示该函数执行的工作。

如果一次执行太多 work，可能会导致动画掉帧并显得断断续续。而且，如果最新的更新取代了某些 work，则这些 work 可能是不必要的。
这是 UI 组件和 function 之间的比较失败的地方，因为与一般 function 相比，组件具有更多特定的关注点。

较新的浏览器（和 React Native）实现了有助于解决此确切问题的 API：`requestIdleCallback` 安排在空闲期间调用的低优先级函数，而 `requestAnimationFrame` 安排在下一个动画帧上调用的高优先级函数。

为了使用这些 API，您需要一种将渲染工作分解为增量单位的方法。
如果仅依赖调用堆栈，它将继续工作直到堆栈为空。

如果可以自定义调用堆栈的行为来优化呈现 UI，那不是很好吗？？？？
如果可以随意中断调用堆栈并手动操作堆栈帧，那不是很好吗？？？？

这就是 React Fiber 的目的。**Fiber 是堆栈的重新实现，专门用于 React 组件**。可以将单个 Fiber 视为**虚拟堆栈帧**。

重新实现堆栈的优点是，可以将堆栈帧保留在内存中，并根据需要（以及在任何时候）执行它们。

除了 Scheduling 之外，手动处理堆栈帧还可以释放并发和错误边界等功能。

### Fiber 的结构

> 注意，随着 React 的发展，Fiber 的结构可能会存在变化

具体来说，Fiber 是一个 JavaScript 对象，其中包含有关组件，其输入和其输出的信息。

Fiber 相当于虚拟堆栈的帧，也相当于组件的实例。

```ts
interface FiberProperty = {
  type: string | () => {};
  key: string;
  child: Fiber;
  sibling: Fiber;
  return: Fiber;
  pendingProps: Object;
  memoizedProps: Object;
  pendingWorkPriority: Number;
  alternate: 'flush' | 'work-in-progress';
  output: HostComponent;
};

class Fiber<P = {}> extends FiberProperty<P> {}
```

#### `type` 和 `key`

Fiber 的 type 和 key 的作用与 React Element 的作用相同。（实际上，当从 Element 创建 Fiber 时，这两个字段将直接复制。）

Fiber 的 type 描述了它所对应的组件。对于复合组件，type 是函数或类组件本身。对于基本组件（eg：`div`，`span`），类型为字符串。

从概念上讲，type 是一个函数（如 `v = f（d）`），其执行由堆栈帧跟踪

与 type 一起，key 在 reconciliation 期间用于确定 Fiber 是否可以重复使用。

#### `child` 和 `sibling`

这两个字段指向其它的 Fiber，以描述 Fiber 树。

子 Fiber 对应于组件的 render 方法返回的值。

对于以下例子：

```jsx
function Parent() {
  return <Child />;
}
```

`Parent` 的子 fiber 相当于 `Child`

字段 `sibling` 说明了渲染返回多个子项的情况

```jsx
function Parent() {
  return [<Child1 />, <Child2 />];
}
```

子 Fiber 形成一个单链列表，其头是第一个子链。
对于上述示例，`Parent` 的 child 指向 `Child1`，`Child1` 的 sibling 指向 `Child2`

回到函数类比，可以将子 Fiber 视为**尾调用函数（tail-called function）**。

#### `return`

return fiber 是程序在处理完当前 Fiber 之后应返回的 Fiber。
从概念上讲，它与堆栈帧的返回地址相同。
也可以将其视为父 Fiber。

如果 Fiber 具有多个子 Fiber，则每个子 Fiber 的 return fiber 都是父 Fiber。因此，在之前的示例中，`Child1` 和 `Child2` 的返回 Fiber 为 `Parent`。

#### `pendingProps` and `memoizedProps`

从概念上讲，props 是函数的参数。Fiber 的 `pendingProps` 在执行开始时设置，`memoizedProps` 在结束时设置。

当传入的 `pendingProps` 等于 `memoizedProps` 时，它表示可以复用 Fiber 的先前输出，从而避免了不必要的工作。

#### `pendingWorkPriority`

一个数字，指示 Fiber 代表的 work 优先级。 `ReactPriorityLevel` 模块列出了不同的优先级及其代表的含义。

除 NoWork 为 `0` 外，数字越大表示优先级越低。例如，您可以使用以下功能来检查 Fiber 的优先级是否至少与给定级别一样高：

```js
// 此 function 仅用于说明；它不是 React Fiber 代码库的一部分。
function matchesPriority(fiber, priority) {
  return (
    fiber.pendingWorkPriority !== 0 && fiber.pendingWorkPriority <= priority
  );
}
```

调度程序使用优先级字段来搜索要执行的下一个 work 单元。

#### `alternate`

- **_flush_** -- 将 Fiber 输出渲染到屏幕上
- **_work-in-progress_** -- 尚未完成的 Fiber；从概念上讲，尚未返回的堆栈帧。

在任何时候，一个组件实例最多具有两个与其对应的 Fiber： `current, flushed fiber` 和 `work-in-progress fiber`。

current Fiber 的 alternate 是 work-in-progress ，而 work-in-progress 的 alternate 是 current Fiber。

使用称为 `cloneFiber` 的函数延迟创建 Fiber 的 alternate。并非总是创建一个新的对象，而是 `cloneFiber` 将尝试复用 Fiber 的 alternate（如果存在），以最大程度地减少分配。

应该将 alternate 字段视为实现细节，但是它经常在代码库中弹出，因此进行讨论很有价值。

#### `output`

**_host component_** React 应用程序的叶节点。它们特定于渲染环境（例如，在浏览器应用中，它们是`div`，`span`等）。在 JSX 中，它们使用小写标记名称表示。

从概念上讲，Fiber 的输出是函数的返回值。
正如之前所一直强调的。

每个 Fiber 最终都有输出，但是输出仅由 **host compoennt** 在叶节点上创建。然后将输出传送到树上。

输出是最终提供给渲染器的输出，以便可以将更改刷新到渲染环境。定义输出的创建和更新方式是渲染器的责任。

## Todo In Feature

- 调度程序如何找到要执行的下一个工作单元。
- 如何通过光纤树跟踪和传播优先级。
- 调度程序如何知道何时暂停和继续工作。
- 如何刷新工作并将其标记为完成。
- 副作用（例如生命周期方法）如何工作。
- 协程是什么，以及如何用于实现上下文和布局等功能。

## 参考

- [React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture)
- [React Reconciler](https://github.com/facebook/react/tree/master/packages/react-reconciler)
- [React Design Principles](https://reactjs.org/docs/design-principles.html)
- [React Basic Theoretical Concepts](https://github.com/reactjs/react-basic)
- [What's Next for React (ReactNext 2016)](https://youtu.be/aV1271hd9ew)
