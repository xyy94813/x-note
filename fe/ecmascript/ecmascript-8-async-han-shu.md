# ECMAScript 8 async 函数

ES8 正式将 async  函数记入 state-4 阶段

## async 函数的声明

一般声明

```js
async function fn() {}
```

`async` 函数表达式

```js
const fn = async function () {};
```

`async` 方法

```js
let obj = { async method() {} }
```

`async` 箭头函数

```js
const foo = async () => {};
```

`class` 内部声明 `async` 函数

```js
class A {
    async method() {}
}
```

`async` 函数总会返回 `Promise` 实例

```js
async function asyncFunc() {
    return 123;
}
asyncFunc().then(x => console.log(x)); // 123

async function asyncFunc() {
    throw new Error('Problem!');
}
asyncFunc().catch(err => console.log(err)); // Error: Problem!
```

## 通过 await 操作符处理异步计算

`await`操作符（只允许在 `async` 函数中）后面是一个 **Promise** 对象（如果不是，将立即转成`resolved`的 Promise ），`await` 会等待 Promise 从 `pending` 状态变更后，再继续往后执行。

* 如果 Promise 的状态是`resolved`，`awiat`的结果就是 resolved value
* 如果 Promise 的状态是`rejected`，`await`将会抛出 rejection valuse

```js
async function asyncFunc() {
    const result = await 1;
    console.log(result + 1);
}

// 等同于
function asyncFunc() {
    return Promise.resolve(1)
    .then(result => {
        console.log(result + 1);
    });
}

async function f() {
  const result = await Promise.reject('出错了');
  console.log(result);
  await Promise.resolve('hello world'); // 不会执行
}
```

处理异常

```js
async function asyncFunc() {
    try {
        await otherAsyncFunc();
    } catch (err) {
        console.error(err);
    }
}

// 等同于:
function asyncFunc() {
    await otherAsyncFunc().catch(err => {
        console.error(err);
    });
}

// 等同于:
function asyncFunc() {
    return otherAsyncFunc()
    .catch(err => {
        console.error(err);
    });
}
```

对比 `Promise.all()`。await 是队列进行的，`Promise.all()`是并行进行的。

```js
let counter = 0
function fn1 (msg) {
  return new Promise((resolve, reject) => {
    if (!counter) {
      setTimeout(() => {
        counter++
        resolve(counter)
      }, 1000)
      console.log(msg)
    } else {
      reject(new Error(`cannot excute [${msg}], because counter > 0`))
    }
  })
}

async function asyncFn () {
  await fn1('async fn 1').catch(err => { console.log(err.msg) })
  await fn1('async fn 2').catch(err => { console.log(err.msg) })
}

asyncFn()

Promise.all([
  fn1('Promise.all 1'),
  fn1('Promise.all 2')
])

// async fn 1
// Promise.all 1
// Promise.all 2
// cannot excute async [fn 2], because counter > 0
```



