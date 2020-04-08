# JavaScript 数据结构与类型

JavaScript 目前包含六种**原始数据类型\(primitives data type\)** // 2018-03-07

- boolean
- null
- undefined
- number
- string
- Symbol \(ES6 引入\)
- bigint

以及两种**基本数据结构\(复合数据结构\)**

- object
- function

> 无论是 Map, Set, RegExp, Array 等全是基于 object

#### 补充说明：

在使用 `typeof` 判断一个变量的类型时需要注意这个问题

```js
typeof null === 'object'; // true, 这是一个内部bug
```

## bigint

`bigint` 能表示大于 $$ 2^(53) - 1 $$ 的整数

```js
const bigInt = 1111111111111111111111111111111111111111n; // 字面量声明
const bigInt2 = new BigInt('1'); // 内置对象生命
```

使用 typeof 测试时， BigInt 对象返回 "bigint"

```js
typeof 1n === 'bigint'; // true
typeof BigInt('1') === 'bigint'; // true
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
