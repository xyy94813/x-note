# ECMAScript 6 变量的解构赋值

> ES6 允许按照一定模式，从数组和对象中快速的取值

## 解构数组

一般用法：

```js
// 按数组索引
let [ n1, n2, n3 ] = [ 1, 2, 3, 4, 5, 6 ];
console.log(n1, n2, n3); // 1, 2, 3

// 多重数组解构
let [ n1, [ [ n211 ], n22 ] ] = [1, [ [2], 3 ]];
console.log(n1, n211, n22); // 1, 2, 3

// 省略部分索引
let [ , , n3] = ['one', 'two', 'three'];
console.log(n3); // three

// 扩展运算符
let [first, ...others] = [1, 2, 3, 4];
console.log(first, others); // 1 [2, 3, 4]
```

只要某种数据结构实现了`Iterator` 接口，都可以采用数组形式的解构赋值。

```js
function* fibs() {
    let a = 0;
    let b = 1;
    while (true) {
        yield a;
        [a, b] = [b, a + b];
    }
}

let [first, second, third, fourth, fifth, sixth] = fibs();
console.log(first, second, third, fourth, fifth, sixth); // 0 1 1 2 3 5
```

解构赋值允许指定默认值。

```js
/**
 * 当解构的值严格意义上为 undefined 时 val === undefined
 * 会采用默认值
 **/
let [a, b, c = 3, d = 4, e] = [1, null, undefined];
console.log(a, b, c, d, e); // 1 null 3 4 undefined
```

## 解构对象

一般用法。

```js
// 根据属性名解构，区分大小写
let { a, b } = { a: 1, b: 2 };
console.log(a, b); // 1 2

// 解构对象时，能够采用别名
let { 
    a: aliasA,
} = { a: 1 };
console.log(a, aliasA); // undefiend 1

// 和数组一样，能够适用于嵌套对象
let obj = {
  a: [
    'Hey',
    { b: 'Girl' }
  ]
};
let { 
  a: [
    x, 
    { 
      b:y 
    }
  ] 
} = obj;
console.log(x, y); // Hey Girl
```

## 解构字符串

> 由于 ES6 里 `String` 对象实现了 `Iterator` 接口，所以能够对字符串按数组解构的方式进行解构

```js
// 一般用法 
const [c1, c2, c3] = 'Hey';
console.log(c1, c2, c3); // H e y

// 使用扩展运算符
const [...str] = 'Hey';
console.log(str); // Hey
```

## 解构数值和布尔值

解构赋值时，如果等号右边时数值或布尔值，会先转换成对应的对象类型。

```js
let { toString: numToString } = 123;
numToString === Number.prototype.toString // true
let { toString: boolToString } = true;
boolToString === Boolean.prototype.toString // true
```

## 函数参数的解构赋值

```js
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}
move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```



