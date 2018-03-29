# JavaScript 数据结构与类型

JavaScript 目前包含六种**原始数据类型\(primitives data type\)** // 2018-03-07

* boolean
* null
* undefined
* number
* string
* Symbol \(ES6 引入\)

以及两种**基本数据结构\(复合数据结构\)**

* object
* function

> 无论是 Map, Set, RegExp, Array 等全是基于object

#### 补充说明：

在使用 `typeof` 判断一个变量的类型时需要注意这个问题

```js
typeof null === 'object'; // true, 这是一个内部bug
```



