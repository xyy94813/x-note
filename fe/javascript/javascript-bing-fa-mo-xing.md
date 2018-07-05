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

### 队列

一个 JavaScript 运行时包含了一个待处理的**消息队列（message queue）**。每一个消息都有一个为了处理这个消息相关联的函数。

在事件循环时，runtime （运行时）总是从最先进入队列的一个消息开始处理队列中的消息。正因如此，这个消息就会被移出队列，并将其作为输入参数调用与之关联的函数。

函数的处理会一直进行，直到执行栈再次为空；然后事件循环（event loop）将会处理队列中的下一个消息（如果还有的话）。

## 事件循环

之所以称为事件循环，是因为它经常被用于类似如下的方式来实现：

```js
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

> 在零延迟调用 setTimeout 时，其并不是过了给定的时间间隔后就马上执行回调函数。其等待的时间基于队列里正在等待的消息数量。

HTML5 规范中的 Event Loop：
1. 对于每个浏览器环境，至多有一个 Event Loop
2. 一个 Event Loop 可以有一个或多个任务队列（task queue）
3. 一个任务队列是一列有序的任务（task），

> micro-task 在 ES2015 规范中称为 Job。 其次，macro-task 代指 task。

### 宏任务队列（macro-tasks）
// TODO

### 微任务队列（micro-tasks）
// TODO

## 浏览器与 Node 在事件循环上的区别
// TODO




