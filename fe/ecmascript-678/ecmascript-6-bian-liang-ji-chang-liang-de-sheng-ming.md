# ECMAScript 6 变量及常量的声明

> ES6规范中，引入了新的关键字 `let` 用于声明变量，以及 `const` 用于声明常量

## let

`let` 与 `var` 的用法类似

```javascript
let a = 123;
```

不同之处在于：

> let 的作用于只在其所属的代码块

```js
var a = [];
for (var i = 0; i < 3; i++) {
    var c = i;
    a[i] = function() {
        console.log(c);
    };
}
a.forEach(item => {
    item();
})
// 4
// 4
// 4

var b = [];
for (var i = 0; i < 3; i++) {
    let c = i;
    b[i] = function() {
        console.log(c);
    };
}
b.forEach(item => {
    item();
})
// 0
// 1
// 2
```

> let 不会发生 “变量提升” 的现象

```js
function f1() {
    let n = 5;
    if (true) {
        let n = 10;
    }
    console.log(n); // 5
}
f1();
```

> let 不允许在相同作用域内重复声明同一个变量

```javascript
{
    let a = 10;
    var a = 1;
}
// Uncaught SyntaxError: Identifier 'a' has already been declared
{
    let a = 10;
    let a = 1;
}
// Uncaught SyntaxError: Identifier 'a' has already been declared
```

## const

const 用于常量声明，一旦声明，参数的值就不能修改

```javascript
const PI = 3.1415;
console.log(PI) // 3.1415
PI = 3;
console.log(PI) // 3.1415
```



