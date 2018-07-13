# JavaScript 并发模型

JavaScript 的并发模型基于“事件循环”。**事件循环（Event Loop）**模型的特性在于，JavaScript 永不阻塞。通常由事件或者回调函数进行 I/O 处理。

JavaScript 运行在浏览器中，是**单线程**的（不考虑 Web Worker 的情况下），每个 `window` 一个 JS 线程，因此只有在每个特定的时刻只有特定的代码能够执行，并且阻塞后续代码。

> 尽管 HTML5 提出了 Web Worker 标准，允许 JavaScript 脚本创建多个线程，但是子线程完全受主线程控制，且不能够操作 DOM。所以，Web Worker 标准并没有改变 JavaScript 单线程的本质。

浏览器是**事件驱动（Event Driven）**的，浏览器中很多的行为是异步的原因是，浏览器会创建事件并放入事件队列中，等待当前代码（所有代码）执行完成。

## 基本概念

![](https://developer.mozilla.org/files/4617/default.svg)

### 栈

函数调用形成了一个栈帧。假设有如下代码：

```js
function foo(b) {
  let a = 10;
  return a + b + 11;
}

function bar(x) {
  let y = 3;
  return foo(x * y);
}

console.log(bar(7));
```

当调用`bar`时，创建了第一个栈帧，栈帧中包含了`bar`的参数和局部变量。当`bar`调用`foo`时，创建第二个栈帧，栈帧中包含了`foo`的参数和局部变量，并被压到第一个栈帧之上。当`foo`返回时，最上层的栈帧就被弹出（剩下`bar`函数的调用帧 ）。当`bar`返回的时候，栈就空了。

### 堆

对象被分配在一个堆中，即用以表示一个大部分非结构化的内存区域。

### 消息队列

一个 JavaScript 运行时包含了一个**消息队列（message queue）**。每一个消息都有一个为了处理这个消息相关联的函数。

当 JavaScript 创建一个异步任务时，不会立刻放入调用栈中，而是先放入消息队列中，等待调用栈的函数清空后再从消息队列中将等待执行的函数加入调用栈中执行。

不同来源的异步任务加入到不同的消息队列中。

> 后面提到 task queue 和 job queue 都被视为一种 message queue。

## 浏览器的事件循环

之所以称为事件循环，是因为它经常被用于类似如下的方式来实现：

```js
while(ture) {
  const task = queue.pop();
  excute(task);
}
```

![](http://lynnelv.github.io/img/article/event-loop/callstack.png)

在事件循环时，runtime （运行时）总是从最先进入队列的一个消息开始处理队列中的消息。正因如此，这个消息就会被移出队列，并将其作为输入参数调用与之关联的函数。函数的处理会一直进行，直到执行栈再次为空；然后 Event Loop 将会处理队列中的下一个消息（如果还有的话）。

> 此外，在零延迟调用 setTimeout 等函数添加异步任务时，并不是过了给定的时间间隔后就马上执行回调函数。其等待的时间基于队列里正在等待的消息数量。  
> setTimeout 等本质上是 JS 线程调用了其它线程，其它线程在条件达成时把任务塞入队列。

### 任务队列与渲染管道

**渲染管道（Rendering Pipeline）**负责在浏览器窗口中绘制内容。绘制的时候必然会堵塞线程，所以当没有任务在执行时，渲染管道才可以运行。如果任务执行的时间很长的时候就会导致渲染管道必须等待。在 60 FPS（每秒绘制 60 次，大约 16ms 就要进行一次绘制）的情况下，如果你的任务执行时间大于 16ms 就会导致页面的卡顿。

事件循环的代码可能就变成了以下实现：

```js
while(ture) {
  const task = queue.pop();
  excute(task);

  if (isRepaintTime()) {
    repaint();
  }
}
```

### 任务队列不止一个

HTML5 中的 Event Loop 规范：

1. 对于每个浏览器环境，至多有一个 Event Loop
2. 一个 Event Loop 可以有一个或多个**任务队列（task queue）**
3. 一个任务队列是一列有序的任务（task）

对于实现了多任务队列的浏览器：

* 所有任务队列可以按照任意的顺序执行
* 但是这些队列也得按照 FIFO 进行排序
* 来自同一资源的任务都进入到同样的队列中。

所以，事件循环的代码也就变成了这样的：

```js
while(ture) {
  const queue = getNextTaskQueue();
  const task = queue.pop();
  excute(task);

  if (isRepaintTime()) {
    repaint();
  }
}
```

### 微任务（micro-task）

任务还分为**宏任务（macro-task、task）**和**微任务（micro-task）**。

macro-task 来源：

* UI 渲染
* I/O
* setTimeout
* setInterval
* setImmediate

micro-task 来源：

* Promise.prototype.then
* Object.observe
* MutationObserver
* process.nextTick \(node\)

Eventloop 在执行完堆栈，或者一个 task 后，会优先询问 microtask queue，如果队列中有任务要执行，则执行，一直到队列为空，然后再执行下一个 task \(每一个 microtask、task 都遵循 run to complete 规则\)。

```js
while(ture) {
  const queue = getNextTaskQueue();
  const task = queue.pop();
  excute(task);

  while (microtaskQueue.hasTask()) {
    doMicrotask();
  }

  if (isRepaintTime()) {
    repaint();
  }
}
```

#### Jobs 和 Job Queues

在 ECMA2015 规范中提到 Job 属于 Micro-Task 的一类。  
并且，ECMA2015 规范中包含两类 Job：

* ScriptJobs
* PromiseJobs

ScriptJobs that validate and evaluate ECMAScript Script and Module source text.   
PromiseJobs that are responses to the settlement of a Promise.

### 动画帧回调队列

我们可以通过传递一个回调函数给`requestAnimationFrame`，然后将任务添加到动画队列中。  
在渲染管道重新渲染屏幕之前，动画队列并不会弹出任何的任务。  
当渲染管道准备重新渲染屏幕时，会先执行动画队列中**一部分的任务**，最后才进行屏幕的渲染。

此时事件循环的实现可能变成了这个样子：

```js
while(ture) {
  const queue = getNextTaskQueue();
  const task = queue.pop();
  excute(task);

  while (microtaskQueue.hasTask()) {
    doMicrotask();
  }

  if (isRepaintTime()) {
    animationTasks = animationQueue.copyTasks();

    for (let _task in animationTasks) {
      doAnimationTask(_task)
    }
    repaint();
  }
}
```

## Node 的事件循环

Node 的事件循环相对于浏览器要简单的多，因为 Node 没有 JS 解析器，没有可以点击的用户交互，没有动画框架的回调，没有渲染管道。

_“浏览器的事件循环就像旋转木马，Node 的则像过山车” -- Erin Zimmer._

Node 包含了三个（宏）任务队列：  
第一个队列，用于所有的事件回调。所以，所有的 XHR 请求，磁盘读写都会进入这个队列。  
第二个队列，用于代码检查阶段（check phase）  
第三个队列，用于所有的时间相关的回调。

> 事实上并不止这三个

可以通过调用`setImmediate`并解析回调来向检查阶段队列添加任务。由于实现方式的原因，`setImmediate(cb)` 要优先于`setTimeout(cb, 0)`执行`cb`。

Node 也有`Promise`，每个宏任务完成后，继续运行 `Promise` 微任务队列。

Node 中还存在一个特殊的微任务队列，即 `nextTick` 队列。`nextTick`队列要优先与`Promise`微任务队列。

所以相比于浏览器的事件循环，Node 的不同之处就在于`setImmediate`和`process.nextTick`这两个 API 的实现上。

> setImmediate\(fn\): do something on the next tick  
> process.nextTick\(fn\): do something immediately

所以 Node 的 Event Loop 实现可能是这样的:

```js
while (tasksAreWaiting()) {  
  queue = getNextQueue();

  while (queue.hasTask()) {
    task = queue.pop();
    excute(task);

    while (nextTickQueue.hasTask()) {
      doNextTickTask();
    }
    while (promiseQueue.hasTask()) {
      doPromiseTask();
    }
  }
}
```

## Web Worker 中的事件循环

每一个 Web Worker 都运行在自己的独立线程中，有着自己的堆栈以及任务队列。  
由于没有 Script 标签、UI，不能操 DOM。 Web Worker 的的事件循环更加的简单

## 参考资料

1. [Philip Roberts: What the heck is the event loop anyway? \| JSConf EU - YouTube](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
2. [Further Adventures of the Event Loop - Erin Zimmer - JSConf EU 2018](https://www.youtube.com/watch?v=u1kqx6AenYw)



