# JavaScript 函数

JavaScript 函数是参数化的。函数的定义会包括一个形参的标识符列表**（arguments）**，这些参数在函数体中像局部变量一样工作。函数调用会为形参提供实参的值。函数使用它们实参的值来计算返回值，成为该函数调用表达式的值。每次的函数调用都会拥有本次**调用上下文（context）**，即 `this` 关键字的值。

> **在 JavaScript 里，函数即对象，程序可以随意操控它们。**

JavaScript 可以把函数赋值给变量，或作为参数传递给其他函数（回调函数）。因为函数就是对象，所以可以给它们设置属性，甚至调用它们的方法。

```js
function func() {
    func.method(func.attr);
}
func.attr = 'a';
func.method = function (a) {
    console.log(a);
}

func(); // a
```

JavaScript 的函数可以嵌套在其他函数中定义，这样它们就可以访问它们被定义时所出的作用域中的任何变量**（闭包）**。

```js
function func() {
    const x = 'can you see me?'
    return function innerFunc() {
        console.log(x);
    }
}

func()(); // can you see me?
```

> 如果函数挂载在一个对象上，作为对象的一个属性，就称它为对象的方法。用于初始化一个新建对象的函数成为**构造函数（constructor）**。

JavaScript 函数调用的四种方式：

* 作为函数
* 作为方法
* 作为构造函数
* 通过`call`或者`apply`方法间接调用

几种方式调用最主要的区别，就是函数内部上下文的区别。

```js
function func(x, y, z) {
    console.log(this, x, y, z);
}

func(1, 2, 3); // window or undefined (取决于是否启用·严格模式·), 1, 2, 3

const obj = {
    func,
}
obj.func(1, 2, 3); // obj, 1, 2, 3

func.call(obj, 1, 2, 3); // obj, 1, 2, 3
func.apply(obj, [1, 2, 3]); // obj, 1, 2, 3
```

不过，构造函数调用和普通的函数调用以及方法调用在实参处理、调用上下文和返回值方面都有不同。

> 作为构造函数调用时，会创建一个新的空对象，这个对象继承自构造函数的`prototype`属性。构造函数会试图初始化这个新创建的对象，并将其作为上下文。

```js
function func() {
    'use strict'
    this.args = [...arguments];
}
func.prototype.a = '123';
const obj = {
    func,
}

const f1 = new func(1);
const f2 = new func; // 作为构造函数调用时，允许没有形参和圆括号
const f3 = new obj.func(1);

console.log(f1.args, f1.a); // [1], '123'
console.log(f2.args, f2.a); // [], '123'
console.log(f3.args, f3.a, obj.args); // [], '123', undefined
```

## 方法链

> 当方法的返回值是一个对象，这个对象还可以在调用它的方法。这种方法调用序列通常称为**“链”**或者**“级联”**。这个序列中每一次调用的结果都是另一个表达式的组成部分

```js
[1, 2, 3, 4, 5, 6, 7, 8, 9]
    .map(item => item * 3)
    .filter((item) => item > 10); // 常见的方法的链式调用
```

_方法的链式调用和构造函数的链式调用不等价_



## 补充说明

通过`bind`函数绑定上下文后的函数，无法再次通过`bind` 函数绑定上下文

```js
function func() {
 console.log(this)
}

const newFunc = func.bind([]).bind({});
newFunc(); // []
```



