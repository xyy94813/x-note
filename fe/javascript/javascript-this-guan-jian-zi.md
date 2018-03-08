# JavaScript this 关键字

> JavaScript 函数是参数化的：函数的定义会包括一个形参的标识符列表，这些参数在函数体中想局部变量一样工作。函数调用会为形参提供实参的值。函数使用它们实参的值来计算返回值，成为该函数调用表达式的值。除了实参之外，每次调用还是拥有另一个值--**本次调用的上下文**--也就是 `this` 关键字的值。
>
> 如果函数挂载在一个对象上，作为对象的一个属性，就称它为对象的方法。**当通过这个对象来调用函数时，该对象就是此次调用的上下文**，也就是该函数的 `this` 的值。



`this` 是一个关键字，不是变量，也不是属性名。

`this` 关键字没有作用域的限制，嵌套的函数不会从调用它的函数中继承 `this` 。如果嵌套函数作为**方法**调用，其 `this` 的值指向调用它的对象。如果嵌套函数作为**函数**调用，其 `this` 值不是全局对象（_非严格模式_）就是 `undefined` （_严格模式_）。



> 调用嵌套函数时 `this` 不会指向调用外层函数的上下文

```javascript
function fn(context) {
    const _this = this;
    console.log(this === context);
    function innerFn() {
        console.log(this === _this);
        console.log(this === window);
    }
    function innerFnWithStrict() {
        'use strict'
        console.log(this === undefined);
        console.log(this === window);
    }
    innerFn(); 
    innerFnWithStrict(); 
}

fn.call(fn, fn); // 依次输出 true false true true false

```

> 作为方法调用时 `this` 指向调用对象

```javascript
function method(context) {
    console.log(this === context);
}

const obj = {
    method,
}

obj.method(obj); // true

```

> ES6 引入的箭头函数则比较特殊，箭头函数的 this 作用域始终指向声明时的作用域。当一个函数的上下文改变后，其内部的嵌套函数的上下文也会随之改变

```javascript
function fn() {
    const _this = this;
    const innerArrowFn = () => {
        console.log(this === _this);
    }
    const obj = {
        method: innerArrowFn,
    }
    innerArrowFn();
    obj.method();
}

fn(); // true true
fn.call(fn); // true true

```



