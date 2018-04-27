# ECMAScript 6 new.target

`new.target`属性允许你检测函数或构造方法是否通过`new`运算符调用。在通过`new`运算符实例化的函数或构造方法内，`new.target`返回一个指向构造方法或函数的引用。而在正常的函数调用中，`new.target`为`undefined`。

`.`运算符通常用访问一个对象的属性，然而`new.target`中的`new`并不是一个真正的对象。

`new.target`属性适用于所有函数访问的元属性。

由于箭头函数自身不包含`this`，`argument`，`super`甚至是`new.target`，所以箭头函数中的`new.target`总是指向最近的函数。

示例：

```js
function Foo() {
  if (!new.target) throw "Foo() must be called with new";
}

Foo(); // throws "Foo() must be called with new"
new Foo(); 

class A {
  constructor() {
    console.log(new.target.name);
  }
}

class B extends A {
  constructor() {
    super();
  }
}

new A(); // A
new B(); // B


function fn () {
  const arrFn = () => {
    console.log(new.target.name)
  }
  return arrFn();
}

new fn(); // fn
fn(); // TypeError: Cannot read property 'name' of undefined
// target is undefined
```



