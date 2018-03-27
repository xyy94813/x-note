# ECMAScript 6 Symbol

ES6 引入了新的基本数据类型`Symbol`，对于每一个`Symbol`的实例都是唯一的。

```js
Symbol('A') === Symbol('A'); // false
```

对象的属性名现在可以是三种类型：

* 数字类型
* 字符串类型
* `Symbol`实例

```js
const obj = {
    [1]: 1,
    ['a']: 'a',
    [Symbol('A')]: 'symbol'
}
```

> `Symbol`作为属性名，该属性不会被`for...in`、`for...of`遍历，`Object.keys()`、`Object.getOwnPropertyNames()`、`JSON.stringify()`也无法返回。只能通过`Object.getOwnPropertySymbols()`方法，获取指定对象的所`Symbol`属性名。

`Symbol` 函数不能使用 new 命令

```js
const symbolA = new Symbol('A'); // TypeError: Symbol is not a constructor
```

`Symbol`函数可以接受一个字符串作为参数作为对`Symbol`实例的描述，**仅仅是描述**。如果接受一个对象则会调用该对象的`toString()`方法。

```js
const strSymbol = Symbol('A'); // Symbol(A)
const numSymbol = Symbol(1); // Symbol(1)
const boolSymbol = Symbol(true); // Symbol(true)
const obj = {
    toString() {
        return 'obj to string'
    }
}
const objSymbol = Symbol(obj); // Symbol(obj to string)
const objSymbol2 = Symbol({}); // Symbol([object Object])
const arrSymbol = Symbol([1, 2]); // Symbol(1,2)
```

`Symbol`实例不能转换成数值，所以不能进行数值运算

```js
const symbol1 = Symbol(1);
Number(symbol1); // TypeError: Cannot convert a Symbol value to a number
let n = 1 + symbol1; // TypeError: Cannot convert a Symbol value to a number
```

`Symbol`可以转为字符串或布尔值

```js
const symbol1 = Symbol(1);
Boolean(symbol1); // true
String(symbol1); // "Symbol(1)"
```

## 相关API

**Symbol.for**

接受一个字符串，如果给定的 key 已经存在则返回现有的`Symbol`对象，如果不存在则返回一个新的`Symbol`实例.

```js
const symbolA1 = Symbol('A'); // Symbol(A)
const symbolA2 = Symbol.for('A'); // Symbol(A)
const symbolA3 = Symbol.for('A'); // Symbol(A)

symbolA1 === symbolA2; // false
symbolA2 === symbolA3; // true
```

> `Symbol.for()`与`Symbol()`都能够产生新的`Symbol`。不过，`Symbol.for()`产生的实例会登记在全局环境中供搜索。

**Symbol.keyFor**

返回一个已登记的`Symbol`类型值的key

```js
var symbolA = Symbol.for('A');
Symbol.keyFor(symbolA); // "A"
var symbolB = Symbol("B");
Symbol.keyFor(symbolB) // undefined
```

**Symbol.hashInstance**

`Symbol.hasInstance()`属性指向一个内部方法。当其他对象使用`instanceof`运算符，判断是否为该对象的实例时，会调用这个方法。

```js
class MyClass {
  [Symbol.hasInstance](foo) {
    return foo instanceof Array;
  }
}
[1, 2, 3] instanceof new MyClass() // true
```

**Symbol.isConcatSpreadable**

对象使用`Array.prototype.concat()`时，是否允许展开

```js
let arr1 = ['c', 'd'];
['a', 'b'].concat(arr1, 'e'); // ['a', 'b', 'c', 'd', 'e']
arr1[Symbol.isConcatSpreadable]; // undefined

let arr2 = ['c', 'd'];
arr2[Symbol.isConcatSpreadable] = false;
['a', 'b'].concat(arr2, 'e'); // ['a', 'b', ['c','d'], 'e']

let obj = {length: 2, 0: 'c', 1: 'd'};
['a', 'b'].concat(obj, 'e'); // ['a', 'b', obj, 'e']
obj[Symbol.isConcatSpreadable] = true;
['a', 'b'].concat(obj, 'e'); // ['a', 'b', 'c', 'd', 'e']
```



