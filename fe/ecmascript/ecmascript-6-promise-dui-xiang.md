# ECMAScript 6 Promise 对象

`Promise`最早由社区提出并实现，例如：[when](https://github.com/cujojs/when)，[q](https://github.com/kriskowal/q) 等。主要是为了解决 JavaScript 中异步编程的**地狱回调**问题。

ES6 中的`Promise`对象是一个代理对象（代理一个值），被代理的值在`Promise`对象创建时可能是未知的。它允许你为异步代码执行结果的成功和失败分别绑定相应的处理方法（handlers）。这让异步方法可以像同步方法那样拥有返回值，但是并非立即返回执行结果。

Promise 是异步编程的一种解决方案。Promise 简单的说就是一个容器，里面保存着未来才会结束的事件（通常是异步操作）。

> 不要和惰性求值混淆：某些语言中有毒性求值和延时计算的特性，也被称之为 “Promise”。例如，编程语言 Scheme。javaScript 中的 “Promise” 表示一种已经发生的状态。在 JavaScript 中可以考虑无参数的“箭头函数“：`f = () => 表达式`创建惰性求值的表达式，使用`f()`求值。

`Promise`的几种状态：

* pending：初始状态；
* fulfilled：操作成功;
* rejected：操作失败。

> 如果一个`promise`对象处在`fulfilled`或`rejected`状态而不是`pending`状态，那么它也可以被称为`settled`状态。`resolved`表示`promise`对象处于`fulfilled`状态。

`Promise`一旦新建就会立即执行。并且，一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise`对象的状态改变只有两种可能：从`Pending`变为`Resolved`和从`Pending`变为`Rejected`。只要这两种情况发生，状态就凝固了，不会再变。

![](https://mdn.mozillademos.org/files/8633/promises.png)

## 调用时机

[Promises/A+ 规范](https://promisesaplus.com/)中指出`onFulfilled`和`onRejected`只有在执行环境堆栈平台仅包含**平台代码**时才可被调用（平台代码指的是引擎、环境以及`promise`等实施代码）。

并且，要确保 和`onRejeted`方法异步执行，且应该在`then`方法被调用的那一轮事件循环之后的新执行栈中执行。这个事件队列可以采用**macro-task（宏任务）**机制或**micro-task（微任务）**机制实现。由于`promise`的实施代码本身就是平台代码，故代码自身在处理在处理程序时可能已经包含一个任务调度队列。

> 宏任务和微任务是异步任务的两种分类。在挂起任务时，JavaScript 引擎会将所有任务按照类别分到这两个队列中，首先在宏任务的队列（这个队列也被叫做**task queue**）中取出第一个任务，执行完毕后取出微任务队列中的所有任务顺序执行；之后再取 macrotask 任务，周而复始，直至两个队列的任务都取完。
>
> 两个类别的具体分类如下：
>
> * macro-task: script（整体代码）, `setTimeout`, `setInterval`, \`setImmediate,  I/O, UI rendering
> * micro-task: `process.nextTick`, `Promises`（浏览器实现的原生 Promise）, `Object.observe`, `MutationObserver`

参考代码：

```js
setTimeout(() => {
    console.log(0);
}, 0);

setImmediate(() => console.log(1));

setTimeout(() => {
    console.log(2);
}, 0);

Promise.resolve().then(() => console.log(3));
new Promise((resolved, rejected) => {
    console.log(4);
    resolved();
})
.then(() => console.log(5));

console.log(6);


// 结果  4,6,3,5,1,0,2

// 对于 Promise.prototype.then(fn) ,  fn  会被放入Promise的 Job Queue，这个队列的执行优先级低于JS栈，高于ScriptJobs
// 前置知识点 Event Loop  +  task queue
```

## 相关API

**Promise.all\( iterable \)**

返回一个新的`promise`对象，该`promise`对象在`iterable`里所有的`promisse`对象都成功的时候才会触发成功，一旦有任何一个`iterable`里面的`promise`对象失败则立即触发该`promise`对象的失败。这个新的`promise`对象在出发成功状态以后，会把一个包含`iterable`里所有`promise`返回值的数组作为成功回调的返回值，顺序和`iterable`的顺序保持一致。如果这个新的`promise`对象出发了失败状态，它会把`iterable`里第一个触发失败的`promise`对象的错误信息作为它的失败错误信息。

**Promise.race\( iterable \)**

当`iterable`参数里的任意一个子`promise`被成功或失败后，父`promise`马上也会用子`promise`的成功返回值或失败详情作为参数调用父`promise`绑定的相应句柄，并返回该`promise`对象。

**Promise.reject\( reason \)**

调用`Promise`的`rejected`句柄，并返回这个`Promise`对象。

**Promise.resolve \( value \)**

用成功值`value`完成一个`Promise`对象。如果该`value`为可继续的（**thenable，即带有**`then`**方法**），返回的`Promise`对象会跟随这个`value`，采用这个`value`的最终状态；否则的话返回值会采用这个`value`**满足（fullfil）返回**的`Promise`对象。

```js
//Thenable在callback之前抛出异常
//Promiserejects
varthenable = {
    then(resolve){
        thrownewTypeError("Throwing");
        resolve("Resolving");
    }
};
let p2 = Promise.resolve(thenable);
p2.then(function(v){
//不会被调用
}, function(e){
    console.log(e); //TypeError: Throwing
});
//Thenable在callback之后抛出异常
//Promiseresolves
varthenable={
    then(resolve){
        resolve("Resolving");
        thrownewTypeError("Throwing");
    }
};
let p3=Promise.resolve(thenable);
p3.then(function(v){
    console.log(v);//输出"Resolving"
}, function(e){
    //不会被调用
});
```

**Promise.prototype.then\(onFulfilled, onRejected\)**

返回一个**新的**`Promise`对象。它最多包含两个参数，成功和失败的回调句柄。

如果省略这两个参数，或者提供非函数，那么将创建一个没有其他处理程序的新`Promise`，只是采用`Promise`的最终状态，`then`被调用。如果省略第一个参数或提供的不是函数，创建的新`promise`简单的采用`Promise`的完成状态，`then`被调用（如果它变为完成）。如果省略第二个参数或者提供的不是函数，创建的新`Promise`简单地采用`Promise`的拒绝状态，`then`被调用（如果它被拒绝）。

**Promise.prototype.catch\(onRejected\)**

返回一个`Promise`，只处理拒绝的情况。

它的行为与调用`Promise.prototype.then(undefined, onRejected)`相同

