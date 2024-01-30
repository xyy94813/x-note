# ECMAScript 2018

ECMAScript 2018 中依旧只做了少部分的变更，其中包括：

* 新增 Promise.prototype.finally()
* 新增 Rest/Spread Properties
* 新增 Asynchronous iteration
* 对字符串模版进行修订
* 正则表达式功能调整
    * s (dotAll) flag for regular expressions
    * RegExp named capture groups
    * RegExp Unicode property escapes
    * RegExp lookbehind assertions

## Promise.prototype.finally()

ECMAScript 2018 的重大变更之一，解决了原先许多的痛点。类似于同步代码块中的 `finally {}` 部分。

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


## 正则表达式功能调整

ES2018 为该 `RegExp` 对象增加了四个新功能，进一步提高了 JavaScript 的字符串处理能力。

### s (dotAll) flag for regular expressions

**正则表达式中的点（`.`） 存在两个限制。**

不能匹配星芒（非 BMP）字符，例如 emoji

> 星芒字符（astral characters）。 non-BMP 字符中的一种

```js
/^.$/.test('😀') // false
```
这个问题可以通过 `/u` 标志 (unicode 模式) 解决

```js
/^.$/u.test('😀') // true
```

与行终止符不匹配

> 以下字符被 ECMAScript 视为行终止符：
> U+000A LINE FEED (LF) (\n)
> U+000D CARRIAGE RETURN (CR) (\r)
> U+2028 LINE SEPARATOR
> U+2029 PARAGRAPH SEPARATOR

> 还有一些 newline-ish 字符不被 ECMAScript 视为行终止符：
> U+000B VERTICAL TAB (\v)
> U+000C FORM FEED (\f)
> U+0085 NEXT LINE

```js
/^.$/.test('\n');  // false
```
之前通过以下方式解决

```js
/^[^]$/.test('\n');  // true
/^[\s\S]$/.test('\n'); // true
/^[\d\D]$/.test('\n'); // true
```

ES2018 采用了以下提议用于解决上诉问题，单行模式中（`.`）能够匹配换行符(`\n`)

```js
/^.$/s.test('\n');  // false
```

### RegExp named capture groups 
 
支持在正则表达式中使用`(?<name>...)`语法命名捕获组

before:
```js
const re = /(\d{4})-(\d{2})-(\d{2})/;
const match= re.exec('2019-01-10');

console.log(match[0]);    // → 2019-01-10
console.log(match[1]);    // → 2019
console.log(match[2]);    // → 01
console.log(match[3]);    // → 10
```

now:
```js
const re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const match = re.exec('2019-01-10');

console.log(match.groups);          // → {year: "2019", month: "01", day: "10"}
console.log(match.groups.year);     // → 2019
console.log(match.groups.month);    // → 01
console.log(match.groups.day);      // → 10
```

要将命名的捕获组插入到方法的替换字符串中replace()，您需要使用`$<name>`构造。

```js 
const re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
'2019-01-10'.replace(re, '$<year>-02-$<day>') // 2019-02-01
```

正则表达式中的 `\k <name>` 表示：匹配先前由命名的捕获组名称匹配的字符串。 例如：

```js
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>$/;
RE_TWICE.test('abc!abc'); // true
RE_TWICE.test('abc!ab'); // false
```

### RegExp Unicode property escapes

ES2018 提供了一种称为 Unicode 属性转义的新类型转义序列，它在正则表达式中提供对完整 Unicode 的支持。

Unicode property escapes look like this:

* Match all characters whose property prop has the value value:
    `\p{prop=value}`
* Match all characters that do not have a property prop whose value is value:
    `\P{prop=value}`
* Match all characters whose binary property bin_prop is True:
    `\p{bin_prop}`
* Match all characters whose binary property bin_prop is False:
    `\P{bin_prop}`

> 假设您要在字符串中匹配 `Unicode` 字符`㉛`。虽然`㉛`被认为是一个数字，但是你不能将它与 `\d` 速记字符类匹配，因为它只支持 ASCII[0-9] 字符。另一方面，Unicode 属性转义可用于匹配 Unicode中 的任何十进制数

```js
console.log(/\d/u.test('㉛')); // false

console.log(/\p{Number}/u.test('㉛')); // true
/^\p{White_Space}+$/u.test('\t \n\r'); // true
/^\p{Script=Greek}+$/u.test('μετά'); // ture
```

### RegExp lookbehind assertions

JavaScript 以前只支持超前断言，现在能够支持后向断言`(?<=...)`

```js
'$foo %foo foo'.replace(/(?<=\$)foo/g, 'bar'); // '$bar %foo foo'
```