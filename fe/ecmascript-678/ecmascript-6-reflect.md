# ECMAScript 6 Reflect

`Reflect`是一个内置的对象，提供拦截 JavaScript 操作的方法。这些方法与**处理器对象（handler）**的方法相同。

> `Reflect`不是一个函数对象，不可对其使用`new`操作符

ES6 推出`Reflect`的主要目的是，将一些明显属于语言范畴的方法，转移到`Reflect`对象上，例如：`Object.defineProperty`、`Function.prototype.apply()`，未来类似的功能也将只在`Reflect`对象上实现。其次，`Reflect`的方法返回值是一个布尔值，对于对象的操作更加合理，不需要使用`try...catch`进行捕获。此外，是某些命令式的操作，转为函数式的行为，比如：`delete obj[prop]`使用`Reflect.deleteProperty(obj, prop)`替代。

由于总能在`Reflect`对象上找到`Proxy`支持的拦截方法，`Proxy`可以通过调用`Reflect`对应的方法作为修改行为的基础，`Proxy`和`Reflect`的结合使得对对象的操作更加的便捷。

```js
const proxy = new Proxy({}, {
    get(...args) {
        const rest = Reflect.get(...args);
        if (rest) {
            console.log('get property success');
        } else {
            console.log('get property failed');
        }
        return rest;
    }
});

Reflect.get(proxy, 'a'); // get property failed

Reflect.set(proxy, 'a', 1);
Reflect.get(proxy, 'a'); //get property success
```

### 相关API

#### Reflect.apply\(target, context, args\)

对一个函数进行调用操作，同时可以传入一个数组作为调用参数.

```js
Reflect.apply(Array.prototype.slice, [1, 2, 3], [0, 2]); // [1, 2]
```

#### Reflect.construct\(target, args\[, newTarget\]\)

对构造函数进行[`new`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)操作.

> newTarget 参考 `new.target`

```js
const arrNum = Reflect.construct(Array, [], Number);

Array.isArray(arrNum); // true
Refelect.getPrototypeOf(arrNum) === Number.prototype; // true
```

#### Reflect.defineProperty\(target, prop, descriptor\)

定义对象属性，与`Object.defineProperty()`类似

```js
const obj = {};
const rest = Reflect.defineProperty(obj, 'a', { value: 1 }); // true
console.log(rest, obj.a); // true 1

Reflect.preventExtensions(obj);
const rest2 = Reflect.defineProperty(obj, 'b', { value: 1 }); // true
console.log(rest2, obj.b); // false undefined
```

#### Reflect.deleteProperty\(target, propKey\)

删除目标对象的属性，`delete`操作符的函数级操作

```js
const obj = { a: 1 };
Reflect.defineProperty(obj, 'b', {
    configurable: false,
    value: 2 
});
const rest1 = Reflect.deleteProperty(obj, 'a');
const rest2 = Reflect.deleteProperty(obj, 'b');

console.log(rest1, obj.a); // true undefined
console.log(rest2, obj.b); // false 2
```

#### Reflect.get\(target, propKey\[, receiver\]\)

获取对象身上某个属性的值

```js
const obj = {
    a: 1,
    [Symbol.for('b')]: 2,
    get c() {
        return 3;
    }
};

Reflect.get(obj, 'a'); // 1
Reflect.get(obj, Symbol.for('b')); // 2
Reflect.get(obj, 'c'); // 3


Reflect.get([1, 2], 1); // 2

function fn () { }
fn.prototype.z = 'parent';
Reflect.get(new fn(), 'z'); // parent
```

#### Reflect.getOwnPropertyDescriptor\(target, propKey\)

获取对象属性描述器。

```js
const obj = { a: 1 };
Reflect.defineProperty(obj, 'b', {
    configurable: false,
    value: 2 
});

Reflect.getOwnPropertyDescriptor(obj, 'a'); // {value: 1, writable: true, enumerable: true, configurable: true}
Reflect.getOwnPropertyDescriptor(obj, 'b'); // {value: 2, writable: false, enumerable: false, configurable: false}

```

#### Reflect.getPrototypeOf\(target\)

获取对象原型

```js
class A {}
const obj = new A();
Reflect.getPrototypeOf(obj) === A.prototype;

function fn () { }
const obj2 = new fn();
Reflect.getPrototypeOf(obj2) === fn.prototype; // true
```

#### Reflect.has\(target, propKey\)

检查对象是否存在某个属性

```js
const obj = {
    a: 1,
    [Symbol.for('b')]: 2,
    get c() {
        return 3;
    }
};

Reflect.has(obj, 'a'); // true
Reflect.has(obj, Symbol.for('b')); // true
Reflect.has(obj, 'c'); // true
Reflect.has(obj, 'd'); // false
```

#### Reflect.isExtensible\(target\)

检查对象是否可扩展

```js
const obj = {};

Reflect.isExtensible(obj); // true
Reflect.preventExtensions(obj);
Reflect.isExtensible(obj); // false
```

#### Reflect.ownKeys\(target\)

返回一个包含所有自身属性（不包含继承属性）的数组

```js
const obj1 = {
    a: 1,
    [Symbol.for('b')]: 2,
    get c() {
        return 3;
    }
};
Reflect.ownKeys(obj1); // ["a", "c", Symbol(b)]

function fn () {
    this.x = 1
}
fn.prototype.z = 'z';
const obj2 = new fn();
Reflect.ownKeys(obj2); // ["x"]
```

#### Reflect.preventExtensions\(target\)

将对象标记为不再可扩展

```js
const obj = {};
obj.a = 1;
Reflect.preventExtensions(obj); // true
obj.b = 1;

console.log(obj); // { a: 1 }
```

#### Reflect.set\(target, propKey, val\[, receiver\]\)

给目标对象分配属性的值

```js
const obj = {};
Reflect.set(obj, 'a', 1); // true

Reflect.defineProperty(obj, 'a', { 
    writeable: false,
}); // true
Reflect.set(obj, 'a', 2); // false

Reflect.preventExtensions(obj); // true
Reflect.set(obj, 'b', 1); // false
```

#### Reflect.setPrototypeOf\(target, prototype\)

修改对象的原型

```js
function fn () {}
const obj = new fn();

Reflect.getPrototypeOf(obj) === fn.prototype; // true

Reflect.setPrototypeOf(obj, Array.prototype); // true
Reflect.getPrototypeOf(obj) === Array.prototype; // true

Reflect.preventExtensions(obj); // true
Reflect.setPrototypeOf(obj, Number.prototype); // true
```



