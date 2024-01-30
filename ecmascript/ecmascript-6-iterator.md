# ECMAScript 6 Iterator

> ES6 里的`Iterator`并不是一种新的语法或是新的内置对象，而是一种协议（protocl），所有遵循了这个协议的对象都可以称之为“迭代器对象”。

ES6 规定，默认的 Iterator 接口需要部署在数据结构的`Symbol.iterator`属性上 —— _iterable protocol （可迭代协议）_，也就是说，对象必须包含`Symbol.iterator`方法。此外，并且该方法返回一个符合 _iterator protocol（迭代器协议）_规定的对象 。该对象包含 1 个`next()`方法，该方法返回一个对象 ，对象包含以下两个属性：

* value —— 值
* done —— 布尔值，标记是否结束遍历

```js
let obj = {
    [Symbol.iterator]: function* () {
        yield 1;
        yield 2;
    }
}
[...obj] // [1, 2]
```

原生具备 `Iterator` 接口的数据结构如下：

* Array
* Map
* Set
* String
* TypedArray
* 函数的 arguments 对象
* NodeList 对象

#### 默认使用 Iterator 的场景

`for...of`**语句**

```js
for (let v of [1, 2, 3, 4]) {
    if (v === 1) {
        continue;
    } else if (v === 4) {
        break;
    }
    console.log(v);
}
// 2 3
```

**解构赋值**

对数组和 Set 进行解构赋值时，会默认使用 Iterator

```js
let set = new Set().add('a').add('b').add('c');
let [a, b] = set; 
// a = 'a'
// b = 'b'
let [first, ...rest] = set;
// first = 'a'
// rest = ['b', 'c']
```

**扩展运算符\(**`...`**\)**

扩展运算符（`...`）也会调用默认的 Iterator 接口。

```js
function fn (x, y, z) {
    console.log(x, y, z);
}
let arr = [1, 2, 3];
fn(...arr); // 1 2 3
```

`yield*`**语法**

`yield*`后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口

```js
let generator = function* () {
  yield 1;
  yield* [2, 3];
  yield 4;
};

var iterator = generator();

iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: undefined, done: true }
```

**遍历数组**

数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。

