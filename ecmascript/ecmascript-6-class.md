# ECMAScript 6 class

ES6 正式引入了 Class 的概念，可以通过`class`关键字去声明一个类。在此之前，`class`一直作为 JavaScript 的保留字，如果想要在 JavaScript 生成实例对象，只能通过构造函数的方式。

```js
function Point(x, y) {
    this.x = x;
    this.y = y;
}
Point.prototype.toString = function () {
    return `[${this.x}, ${this.y}]`;
}
let p = new Point(1, 2);
p.toString(); // [1, 2]

// 新的 class 写法只是让对象原型的写法更加清晰、更像面向对象编程的语法
class PointB {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    toString() {
       return `[${this.x}, ${this.y}]` 
    }
}
let p2 = new PointB(1, 2);
p.toString(); // [1, 2]
```

ES6 的类，可以看作是构造函数的语法糖

```js
class A {}
typeof A; // "function"
A === A.prototype.constructor; // true
```

## 声明 class

一般声明

```js
class A {}
```

class  表达式

```js
const A = class {}
```

声明继承关系（不支持多继承）

```js
class A {}
class B extends A {}
```

## 调用 class

类必须使用`new`操作符调用，如果不使用`new`操作符调用，则会抛出一个`TypeError`

```js
class A {}
A(); // Uncaught TypeError: Class constructor A cannot be invoked without 'new'
```

不传递参数时可以省略括号

```js
class A {}
let a = new A;
```

## class 的方法

`class`的普通方法

```js
class A {
    method() {}
}
```

`class`的静态方法

```js
class A {
    static staticMethod() {
        return 1;
    }
}
A.staticMethod(); // 1
```

`class`的私有方法

```js
// 私有变量和方法仍在提案中，可以利用 Symbol 实现私有方法
const bar = Symbol('bar');
const snaf = Symbol('snaf');

export default class myClass{

  // 公有方法
  foo(baz) {
    this[bar](baz);
  }

  // 私有方法
  [bar](baz) {
    return this[snaf] = baz;
  }

  // ...
};

// 提案中的私有方法声明方式
class A {
    #method() {}
}
```

`class`内的`getter`和`setter`

```js
class A {
    constructor(val) {
        this._val = val || null;
    }
    get value() {
        return this._val;
    }
    set value(val) {
        if (this._val !== val) {
            this._val = val;
            console.log('value changed');
        }
    }
}
let a = new A;
a.value; // null
a.value = 1; // value changed
a.value = 1;
```

`class`内的`Generator`方法

```js
class A {
  constructor(...args) {
    this.args = args;
  }
  * [Symbol.iterator]() {
    for (let arg of this.args) {
      yield arg;
    }
  }
}

for (let x of new A('hello', 'world')) {
  console.log(x);
}
// hello
// world
```

类的内部定义的所有方法都是不可枚举的

```js
class A {
    method1() {}
    method2() {}
}
let a = new A;
Object.keys(Reflect.getPrototypeOf(a)); // []

function B () {}
B.prototype.method1 = function () {}
B.prototype.method2 = function () {}
let b = new B;
Object.keys(Reflect.getPrototypeOf(b)); // ["method1", "method2"]
```

`class`的方法内的`this`并不会与实例绑定

```js
class A {
    method1() {
        this.method2();
    }
    method2() {
        console.log('method2 of class "A"');
    }
}
a = new A;
const { method1 } = a;
method1(); // TypeError: Cannot read property 'method2' of undefined
```

## class 的静态属性、实例属性和私有属性

ES6 中 class 不能在class 内部直接定义静态属性、实例属性和私有属性。ES7 开始允许在 class 内部直接声明静态属性、实例属性实。class 的私有属性，仍处于提案中。

```js
class A {
    consturctor() {
        this.x = 1; // 只能在构造函数内声明实例属性
    }
}
A.staticProp = 1; // 声明 A 的静态属性

// ES7 支持的新语法
class B {
    static staticProp = 1;
    x = 1;    
}
```

私有属性仍在提案中

```js
// 通过 Symbol 实现私有属性
const pVal = Symbol('privateVal');

class A {
    constructor(x) {
        this[pVal] = undefined;
    }
    get value() {
        return this[pVal];
    }
    set value(val) {
        this[pVal] = val;
    }
}

// 提案中的私有属性声明
class A {
    #x;
    constructor(x = 0) {
        #x = +x; // 写成 this.#x 亦可
    }
    get x() { return #x }
    set x(value) { #x = +value }
}
```

## 变量提升

`class`不存在变量提升

```js
a = new A; // ReferenceError: A is not defined
class A {}
```

## 严格模式

类的内部默认采用严格模式

```js
class A {
    method1() {
        function fn () {
            console.log(this);
        }
        fn();
    }
}
a.method1(); // undefined
```



