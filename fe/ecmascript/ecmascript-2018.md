# ECMAScript 2018

ECMAScript 2018 中依旧只做了少部分的变更，其中包括：

* 新增 Promise.prototype.finally()
* 新增 Rest/Spread Properties
* 新增 Asynchronous iteration
* 对字符串模版进行修订
* s (dotAll) flag for regular expressions
* RegExp named capture groups
* RegExp Unicode property escapes
* RegExp lookbehind assertions

## Promise.prototype.finally()

ECMAScript 2018 的重大变更之一，解决了原先许多的痛点。使用的类似于同步代码块中的 `finally {}` 部分。

```
try{
}catch(e){
}finally{
}
```

基本用法:

```js
  let connection;
  db.open()
    .then(conn => {
        connection = conn;
        return connection.select({ name: 'Jane' });
    })
    .then(result => {
        // Process result
        // Use `connection` to make more queries
    })
    // ···
    .catch(error => {
        // handle errors
    })
    .finally(() => {
        connection.close();
    });
```


## Rest/Spread Properties

在此之前，对象解构中的其余运算符 `...` 仅适用于数组解构和参数定义。对象文字中的扩展运算符 `...` 仅适用于数组文字以及函数和方法调用。

> 虽然大多数人用于该操作符处理对象已经很久了，但该公能在 ES2019 中才正式发布

rest operator `...` 会拷贝所有的可枚举属性

```js
const {foo, ...rest } = obj;
const _rest = { ...rest };

function func({ param1, param2, ...rest }) { // rest operator
    console.log('All parameters: ', { param1, param2, ...rest }); // spread operator
    return param1 + param2;
}
 
```

根据每个对象文字的顶级，您最多可以使用一次 rest `...` 运算符，它必须出现在结尾处：

```js
const {...rest, foo} = obj; // SyntaxError
const { foo, ...rest1, ...rest2 } = obj; // SyntaxError
```

在对象文字内部，扩展运算符（...）将其操作数的所有可枚举属性插入到通过文字创建的对象中：

```js
const obj = {foo: 1, bar: 2, baz: 3}
console.log({...obj, foo: true}) // {foo: true, bar: 2, baz: 3}
console.log({ foo: true, ...obj }) // {foo: 1, bar: 2, baz: 3}
```

属性的扩展运算符（`...`），类似于 `Object.assign()`

## Asynchronous iteration

ES6 内置了对同步迭代数据的支持。 但是异步传输的数据呢？ 例如，从文件或HTTP连接异步读取的文本行。

ES6 中同步迭代工作原理如下：

* Iterable：一个对象，通过一个键为 `Symbol.iterator` 的方法表示它可以迭代。
* Iterator：通过在 `iterable` 上调用`[Symbol.iterator]()` 返回的对象。 它将每个迭代元素包装在一个对象中，并通过其 `next()` 方法返回它 - 一次一个。
* IteratorResult：next（）返回的对象。 属性值包含一个迭代元素，在最后一个元素之后属性`done`为`true`（通常可以忽略值; 它几乎总是未定义）。

先前解释的迭代方式是同步的，它不适用于异步数据源。 例如，在以下代码中，readLinesFromFile（）无法通过同步迭代传递其异步数据：

```js
for (const line of readLinesFromFile(fileName)) {
    console.log(line);
}
```

Asynchronous iteration 是一种异步工作的新迭代协议：

异步迭代通过`Symbol.asyncIterator`标记。
异步迭代器的方法`next()`返回`IteratorResults`的`Promises`（直接与`IteratorResults`对比）。

也许同步迭代器能够为每一个可遍历元素返回一个 `Promise`?
但是， 无论迭代是否完成通常都是异步确定的。


### Async iteration 的接口

> 在 TypeScript 中, 接口看起来就像这样

```ts
interface AsyncIterable {
    [Symbol.asyncIterator]() : AsyncIterator;
}
interface AsyncIterator {
    next() : Promise<IteratorResult>;
}
interface IteratorResult {
    value: any;
    done: boolean;
}
```
### for-await-of

在 `async` 函数中，可以通过 `for-await-of` 便利异步便利器。

```js
async function f() {
    for await (const x of createAsyncIterable(['a', 'b'])) {
        console.log(x);
    }
}
// Output:
// a
// b
```

与 `await` 如何在异步函数中工作类似，如果 `next()` 返回拒绝，则循环抛出异常：

```js
function createRejectingIterable() {
    return {
        [Symbol.asyncIterator]() {
            return this;
        },
        next() {
            return Promise.reject(new Error('Problem!'));
        },
    };
}
(async function () { // (A)
    try {
        for await (const x of createRejectingIterable()) {
            console.log(x);
        }
    } catch (e) {
        console.error(e);
        // Error: Problem!
    }
})(); // (B)
```

> `for-of-await` 在模块和脚本的顶层不起作用。

如果 `for-of-await` 遍历一个同步遍历器，会将同步便利器转换为异步便利器处理。

```
async function main() {
    const syncIterable = [
        Promise.resolve('a'),
        Promise.resolve('b'),
    ];
    for await (const x of syncIterable) {
        console.log(x);
    }
}
main();

// Output:
// a
// b
```

### Asynchronous generators

普通（同步）生成器有助于实现同步迭代。 异步生成器对异步迭代执行相同的操作。

```
async function* createAsyncIterable(syncIterable) {
    for (const elem of syncIterable) {
        yield elem;
    }
}
```

> 通过在函数后面加一个星号将正常函数转换为普通生成器。异步函数通过执行相同操作变为异步生成器。

Asynchronous generators 工作原理：

普通生成器返回生成器对象 `genObj`。 每次调用 `genObj.next()`都会返回一个包含生成值的对象`{value，done}`。
异步生成器返回生成器对象 `genObj`。 每次调用 `genObj.next()` 都会返回一个`resolve`状态，并且值为对象 `{value，done}` 的`Promise`。


## 字符串模版功能修订

带标签的模版字符串应该允许嵌套支持常见转义序列的语言，移除对ECMAScript在带标签的模版字符串中转义序列的语法限制

使用标记的模板文字，您可以通过在模板文字之前提及函数来进行函数调用：

```js
String.raw`\u{4B}`; // '\\u{4B}'
```

[`String.raw`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/raw) 是一个所谓的标记函数。 标记函数在模板文字中接收两个版本的固定字符串片段（模板字符串）：

* `Cooked`: 转义序列被解析。 `\u{4B}` => 'K'.
* `Raw`: 转义称普通文本. `\u{4B}` => '\\u{4B}'.

```js
function tagFunc(tmplObj, substs) {
    return {
        Cooked: tmplObj,
        Raw: tmplObj.raw,
    };
}
tagFunc`\u{4B}`;  // { Cooked: [ 'K' ], Raw: [ '\\u{4B}' ] }
```

> 非法转义序列在"cooked"当中仍然会体现出来。它们将以 undefined 元素的形式存在于"cooked"之中：


## s (dotAll) flag for regular expressions

TODO

## RegExp named capture groups 

TODO

## RegExp Unicode property escapes

TODO

## RegExp lookbehind assertions

TODO
