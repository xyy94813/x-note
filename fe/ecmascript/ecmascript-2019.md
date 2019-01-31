# ECMAScript 2019

以下新特性被加入 stage4

* Object 的新 API
* Array 的新 API
* String 的新 API
* `JSON.stringify` 对 unicode 字符集进行优化

## Object 的新 API

**Object.fromEntries(iterable)**

```js
obj = { abc: 1, def: 2, ghij: 3 };
res = Object.fromEntries(
  Object.entries(obj)
  .filter(([ key, val ]) => key.length === 3)
  .map(([ key, val ]) => [ key, val * 2 ])
);

// res is { 'abc': 2, 'def': 4 }
```

## Array 的新 API

**Array.prototype.flat(depth)**

```js
const array = [1, [2, [3]]];
array.flat();
// [1, 2, [3]]

array.flat(Infinity);
// [1, 2, 3]
```

**Array.prototype.flatMap(fn)**

```js
let arr1 = [1, 2, 3, 4];
arr1.flatMap(x => [x * 2]);
// [2, 4, 6, 8]
// only one level is flattened
arr1.flatMap(x => [[x * 2]]);
// [[2], [4], [6], [8]]
```

## String 的新 API

**String.prototype.trimStart**

```js
console.log('   Hello world!   '.trimStart());
// "Hello world!   "
```
**String.prototype.trimEnd**

```js
console.log('   Hello world!   '.trimEnd());
// "   Hello world!"
```

## `JSON.stringify` 对 unicode 字符集进行优化

`JSON.stringify` 堆 no-bmp 字符的转义进行了优化

```js
// Non-BMP characters still serialize to surrogate pairs.
JSON.stringify('𝌆')
// '"𝌆"'
JSON.stringify('\uD834\uDF06')
// '"𝌆"'

// Unpaired surrogate code units will serialize to escape sequences.
JSON.stringify('\uDF06\uD834')
// '"\\udf06\\ud834"'
JSON.stringify('\uDEAD')
// '"\\udead"'
```
