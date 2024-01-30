# ECMAScript 8 内存共享与原子性

由于`Web Worker`的出现，我们可以通过 JavaScript 的内置对象 `Worker`的创建一个后台任务，然后执行一些复杂的任务以避免主线程被堵塞，换句话说， JavaScript 现在是能够支持多线程的。

```js
// main js
var w = new Worker("myworker.js")
w.postMessage("hi");     // send "hi" to the worker
w.onmessage = function (ev) {
  console.log(ev.data);  // prints "ho"
}

// myworker.js
onmessage = function (ev) {
  console.log(ev.data);  // prints "hi"
  postMessage("ho");     // sends "ho" back to the creator
}
```

目前，由于每一个`worker`实例都拥有一个属于自己的独立的全局环境，`worker`和 创建`worker`的对象之间只能够通过`postMessage(str)`进行数据共享，并且只能传递字符串。

但是，实际开发中，共享的数据往往是十分复杂的，仅能够传递字符串并不能满足日常的开发。于是，**Lars T. Hansen** 提出一种能够通过[内存共享](https://github.com/tc39/ecmascript_sharedmem/blob/master/README.md)的方式，使得`worker`和`worker creator`之间能够更加方便的进行通信。

这个提案，最终在 ES8 中进入到 Stage-4 中。

目前这个功能涉及到两个新的内置对象：

* `SharedArrayBuffer`
* `Atomics`

## ~~SharedArrayBuffer~~

`SharedArrayBuffer` 对象用来表示一个通用的，固定长度的原始二进制数据缓冲区，类似于 `ArrayBuffer`对象，它们可以用来在共享内存上创建视图。与`ArrayBuffer`不同的是，`SharedArrayBuffer`不能被分离。

`worker creator`可以通过`SharedArrayBuffer`构造函数创建一个指定大小的共享内存。然后通过全局的`postMessage()`函数共享这块内存。在`woker`内部，通过访问事件的`data`属性来访问这块内存。

```js
// main js
var sab = new SharedArrayBuffer(1024);  // 创建 1KiB shared memory
var w = new Worker("myworker.js")
w.postMessage(sab)

// myworker.js
var sab;
onmessage = function (ev) {
   sab = ev.data;  // 1KiB shared memory, the same memory as in the parent
}
```

> 不过，由于能够轻易地利用《[边信道（side-channel）读取未授权内存的攻击技术](https://googleprojectzero.blogspot.com/2018/01/reading-privileged-memory-with-side.html)》中提到的[Spectre](https://spectreattack.com/)漏洞——一种利用现代 CPU 使用的执行优化功能的新攻击技术，`SharedArrayBuffer`功能将在 Chrome 和 FireFox 的新版本中禁用，并将逐渐被所有浏览器禁用。

## Atomics

`Atomics` 对象提供了一组静态方法用来对`SharedArrayBuffer`对象进行原子操作。

多个共享内存的线程能够同时读写同一位置上的数据。原子操作会确保正在读或写的数据的值是符合预期的，即下一个原子操作一定会在上一个原子操作结束后才会开始，其操作过程不会中断。

**相关API**

> Atomics 对象目前已经进入到 Stage-4 的状态，但是 MDN 文档中所有提到的 Atomics 的 API 均处于草案的状态（2018-05-02）

**Atomics.add\(typedArray, index, value\)**

将指定位置上的数组元素与给定的值相加，并返回相加前该元素的值。

```js
var sab = new SharedArrayBuffer(1024);
var ta = new Uint8Array(sab);

Atomics.add(ta, 0, 12); // returns 0, the old value
Atomics.load(ta, 0); // 12
```

**Atomics.and\(typedArray, index, value\)**

将指定位置上的数组元素与给定的值相与，并返回与操作前该元素的值。

```js
var sab = new SharedArrayBuffer(1024);
var ta = new Uint8Array(sab);
ta[0] = 5;

Atomics.and(ta, 0, 1); // returns 0, the old value
Atomics.load(ta, 0);  // 1
```

**Atomics.compareExchange\(typedArray, index, expectedValue, replacementValue\)**

如果数组中指定的元素与给定的值相等，则将其更新为新的值，并返回该元素原先的值。

```js
var sab = new SharedArrayBuffer(1024);
var ta = new Uint8Array(sab);
ta[0] = 7;

Atomics.compareExchange(ta, 0, 7, 12); // returns 7, the old value
Atomics.load(ta, 0); // 12
```

**Atomics.exchange\(typedArray, index, value\)**

将数组中指定的元素更新为给定的值，并返回该元素更新前的值。

```js
var sab = new SharedArrayBuffer(1024);
var ta = new Uint8Array(sab);

Atomics.exchange(ta, 0, 12); // returns 0, the old value
Atomics.load(ta, 0); // 12
```

**Atomics.load\(typedArray, index\)**

返回数组中指定元素的值。

```js
var sab = new SharedArrayBuffer(1024);
var ta = new Uint8Array(sab);

Atomics.add(ta, 0, 12);
Atomics.load(ta, 0); // 12
```

**Atomics.store\(typedArray, index, value\)**

将数组中指定的元素设置为给定的值，并返回该值。

```js
var sab = new SharedArrayBuffer(1024);
var ta = new Uint8Array(sab);

Atomics.store(ta, 0, 12); // 12
```



