# ECMAScript 7

ES7 相比 ES6 新增的功能并没有那么多，只增加了两个新的功能

* Array.prototype.includes
* 指数运算符（`**`）

## Array.prototype.includes\(val\)

`Array.prototype.includes()`正式进入`stage-4`。用于判断 Array 实例中，是否包含某个具体的值。

```js
const arr = [1, 2, 3];
arr.includes(1); // true
arr.includes(4); // false
```

`Array.prototype.includes()`与`Array.prototype.indexOf()`类似，主要不同的是`Array.prototype.includes()`能够判断数组是否存在`NaN`。

```js
const arr = [NaN];
arr.includes(NaN); // true
arr.indexOf(NaN); // -1
```

此外，`Array.prototype.includes()`不区分`+0`和`-0`

```js
[-0].includes(+0); // true
```

## 指数运算符（\*\*）

ES7 正式收入的指数运算符由 Rick Waldron 提出。

```js
6 ** 2; // 36
```

在此之前，只能通过内置的`Math.pow()`进行指数运算

```js
Math.pow(6, 2); // 36
```

扩展的运算符`**=`与`+=`运算符类似

```js
let num = 3;
num **= 2; // 9
num **= 2; // 81
```

指数运算符的优先级非常高，要高于`*`运算符

```js
2 ** 2 * 2; // 8
2 ** (2 * 2); // 16
```



