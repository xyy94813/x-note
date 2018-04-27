# ECMAScript 6 Generator

Generator 是 ES6 提供的一种异步编程解决方案。

## 定义生成器函数

可以使用构造函数`GeneratorFunction`或`function*`表达式定义**生成器函数**。

`GeneratorFunction`并不是一个全局对象，可以通过一下代码获取

```js
const GeneratorFunction = Reflect.getPrototypeOf(function* (){}).constructor;
```

通过`GeneratorFunction`定义生成器函数

> 语法：new GeneratorFunction \(\[arg1\[, arg2\[, ...argN\]\],\] functionBody\)

```js
const GeneratorFunction = Reflect.getPrototypeOf(function* (){}).constructor;

let g = new GeneratorFunction('a', 'yield a * 2');
let gObj = g(10);
gObj.next(); // {value: 20, done: false}
```

通过`function*`表达式定义生成器函数

```js
function* gen (x) {
    yield 1;
    return x;
}

let genObj = gen(10);

genObj.next(); // 执行 yield 1，返回 {value: 1, done: false}
genObj.next(); // 执行 return x，返回 {value: 10, done: true}
genObj.next(); // 执行完毕 {value: undefined, done: true}
```

## 调用生成器函数

调用一个**生成器函数（generator function）**并不会马上执行里面的语句，而是返回一个**生成器对象，**该对象符合**可迭代协议**和**迭代器协议**。

当这个生成器对象的`next()`方法被首次（后续）调用时，其内的语句会执行到第一个（后续）出现`yield`的的位置为止，`yield`后紧跟迭代器要返回的值。如果是`yield*`表达式，则表示将执行权移交给另一个生成器函数（当前生成器暂停执行）。

```js
function* anotherGenerator(i) {
  yield i + 1;
  yield i + 2;
  yield i + 3;
}

function* generator(i){
  yield i;
  yield* anotherGenerator(i);// 移交执行权
  yield i + 10;
}

var gen = generator(10);

console.log(gen.next().value); // 10
console.log(gen.next().value); // 11
console.log(gen.next().value); // 12
console.log(gen.next().value); // 13
console.log(gen.next().value); // 20
```

当在生成器函数中使用`return`时，会导致生成器立即变为完成状态，即调用`next()`方法返回的对象的`done`属性的值为`true`。如果`return`后面跟了一个值，那么这个值会作为当前调用`next()`方法返回的对象的属性`value`的值。

```js
function* gen (x) {
    yield 1;
    y = yield 'foo';
    yield y;
    return x;
    yield "unreachable";
}


let genObj = gen(10);

genObj.next(); // 执行 yield 1，返回 {value: 1, done: false}
genObj.next(); // 执行 yield 'foo'，返回 {value: "foo", done: false}
genObj.next(100); // 将 100 赋给上一条 yield 'foo' 的左值，即执行 y=100，返回 {value: 100, done: false}
genObj.next(); // 执行 return x，返回 {value: 10, done: true}
genObj.next(); // 执行完毕 {value: undefined, done: true}
genObj.next(); // 执行完毕 {value: undefined, done: true}
```

## 迭代器对象的方法

**Generator.prototype.next\(val\)**

迭代器对象符合可迭代协议和迭代器协议，所以一定包含该方法

**Generator.prototype.return\(val\)**

返回给定的值并结束生成器

```js
function* gen () {
    yield 1;
    yield 2;
}
let genObj = gen();
genObj.next(); // {value: 1, done: false}
genObj.return(10); // {value: 10, done: true}
genObj.next(); // {value: undefined, done: true}
```

**Generator.prototype.throw\(exception\)**

用来向生成器抛出异常，并恢复生成器的执行

```js
function* gen() {
  while(true) {
    try {
       yield 42;
    } catch(e) {
      console.log("Error caught!");
    }
  }
}

var g = gen();
g.next(); // { value: 42, done: false }
g.throw(new Error("Something went wrong")); // "Error caught!"
g.return(2); // {value: 2, done: true}
g.throw(new Error("Something went wrong"));  // Uncaught Error: Something went wrong
```



