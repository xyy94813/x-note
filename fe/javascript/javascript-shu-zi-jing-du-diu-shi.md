# JavaScript 数字精度丢失

由于计算机的二进制实现和位数限制导致有些数字无法有限表示，就像 10 / 3 这样的结果无法表示一样。

JavaScript 的数字类型（Number）遵循 [IEEE 754](https://zh.wikipedia.org/wiki/IEEE_754) 规范，采用**双精度存储（double precision）**，占用 64 bit 空间。大多数编程语言的都基于这个规范。

根据 IEEE 754 规范 JavaScript 数字类型的内存结构如下：

* 1位表示符号位
* 11位表示指数
* 52位表示尾数

JavaScript 数字类型能表示的最大整数也就是 $$ 2^{53} = 9007199254740992 $$

当长度不足时，只能模仿十进制的四舍五入的策略，但是由于二进制只有 0 和 1，于是就变成了 0 舍 1 入。

所以在计算机中就会出现以下的奇怪现象：

```js
console.log(0.1 + 0.2); // 0.30000000000000004

console.log(0.3 - 0.2); // 0.09999999999999998

console.log(8.7 * 100); // 869.9999999999999
console.log(8.8 * 100); // 880.0000000000001

console.log(0.2 / 1000000); // 2.0000000000000002e-7
console.log((0.2/1000000) === 0.0000002); // false

console.log(9999999999999999 === 10000000000000001); // true
```



## 解决方案

无论是 `toFixed()` 还是先乘倍数再缩小回原来的倍数，依旧无法完美的解决这个问题

```js
(0.1 + 0.2).toFixed(1); // '0.3'
(1.335).toFixed(2); // 1.33

((0.1 * 10) + (0.2 * 10)) / 10; // 0.3

```

完美的解决思路应该是将浮点数拆分成 [`Number.MIN_SAFE_INTEGER`, `Number.MAX_SAFE_INTEGER`] 区间内的值再进行计算。

可以参考以下库的实现方式：

* [bignumber.js](https://github.com/MikeMcl/bignumber.js)
* [decimal.js](https://github.com/MikeMcl/decimal.js)
* [big.js](https://github.com/MikeMcl/big.js)
* [safe-float](https://github.com/vincenting/safe-float)







