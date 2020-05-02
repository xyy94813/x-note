# JavaScript 数据结构与类型

JavaScript 目前包含七种**原始数据类型\(primitives data type\)** // 2018-03-07

- boolean
- null
- undefined
- number
- string
- Symbol \(ES6 引入\)
- bigint \(ES 2020\)

以及两种**基本数据结构\(复合数据结构\)**

- object
- function

> 无论是 Map, Set, RegExp, Array 等全是基于 object

> Note：
> `typeof null === 'object'` 返回 true，内部 BUG 且无法修复。
> `typeof NaN === 'number'` 且 `NaN !== NaN`，只能通过 `Number.isNaN` 判断。`NaN` 的含义是无法表示的数字。

## Why typeof(null) is object

早期的 JavaScript 使用 32 bit 存值，包括一个表示类型的标记(1-3 位)和值的实际数据。

类型标记存储在低位上。

- `1` Int
- `000` Object
- `010` Double
- `100` String
- `110` Boolean

特殊值

- `undefined` 用整数 `−2**30` 次方，溢出的方式表示
- `null` 机器空指针，标志位也是 `000`。

`typeof` 是基于类型的标记做的判断，因此 `typeof null === 'object'`

```C++
// From: https://dxr.mozilla.org/mozilla-central/source/js/public/Value.h#57
// Use enums so that printing a JS::Value in the debugger shows nice
// symbolic type tags.

enum JSValueType : uint8_t {
  JSVAL_TYPE_DOUBLE = 0x00,
  JSVAL_TYPE_INT32 = 0x01,
  JSVAL_TYPE_BOOLEAN = 0x02,
  JSVAL_TYPE_UNDEFINED = 0x03,
  JSVAL_TYPE_NULL = 0x04,
  JSVAL_TYPE_MAGIC = 0x05,
  JSVAL_TYPE_STRING = 0x06,
  JSVAL_TYPE_SYMBOL = 0x07,
  JSVAL_TYPE_PRIVATE_GCTHING = 0x08,
  JSVAL_TYPE_BIGINT = 0x09,
  JSVAL_TYPE_OBJECT = 0x0c,

  // This type never appears in a Value; it's only an out-of-band value.
  JSVAL_TYPE_UNKNOWN = 0x20
};
```

## bigint

`bigint` 能表示大于 $$ 2^(53) - 1 $$ 的整数

```js
const bigInt = 1111111111111111111111111111111111111111n; // 字面量声明
const bigInt2 = new BigInt("1"); // 内置对象生命
```

使用 typeof 测试时， BigInt 对象返回 "bigint"

```js
typeof 1n === "bigint"; // true
typeof BigInt("1") === "bigint"; // true
```

`bigint` 只能与 `bigint` 进行运算，不能直接与 `number` 进行运算。

```js
const a = 0.1;
const b = 1n;
const c = 2n;
const d = a + b; // Uncaught TypeError: Cannot mix BigInt and other types, use explicit conversions
const e = b + c; // 3n
```

由于 `bigint` 存在符号，所以不支持无符号位移运算符 `>>>` 和 `<<<`。
为了兼容 asm.js ，BigInt 不支持单目 (+) 运算符。

```js
1n >>> 1; // Uncaught TypeError: Cannot mix BigInt and other types, use explicit conversions at <anonymous>:1:4
```

`bigint` 能够直接与 `number` 进行比较运算。
但是不是严格相等的，而是宽松相等的。

```js
0n === 0; // ↪ false
0n == 0; // ↪ true

1n < 2; // ↪ true
2n > 1; // ↪ true
2n > 2; // ↪ false
2n >= 2; // ↪ true
```

bigint 转换成 bool 时与 number 类似

```js
0n || 12n || 3n; // 12n
```
