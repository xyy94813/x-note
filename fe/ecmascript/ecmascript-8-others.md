# ECMAScript 8 Others

ES8 除了 async 函数、内存共享和原子性这两个比较大的功能外，并无太大改动。

ES8 剩余功能变更：

* `Object.entries()` 
* `Object.values()`
* `String.prototype.padStart()` 
* `String.prototype.padEnd()`
* `Object.getOwnPropertyDescriptors()`
* 允许在参数定义和函数调用中使用尾随逗号

## Object.entries\(obj\) 

`Object.entries()` 方法返回一个给定对象自身可枚举属性的键值对数组，其排列与使用`for...in` 循环遍历该对象时返回的顺序一致，不过`Object.entries()`不会枚举原型链中的属性。

这个方法已经推行很久，终于在 ES8 中宣布正式进入 Stage-4 阶段

```js
const obj = { foo: 'bar', baz: 42 };
console.log(Object.entries(obj)); // [ ['foo', 'bar'], ['baz', 42] ]

// array like object
const obj = { 0: 'a', 1: 'b', 2: 'c' };
console.log(Object.entries(obj)); // [ ['0', 'a'], ['1', 'b'], ['2', 'c'] ]

// array like object with random key ordering
const anObj = { 100: 'a', 2: 'b', 7: 'c' };
console.log(Object.entries(anObj)); // [ ['2', 'b'], ['7', 'c'], ['100', 'a'] ]

// getFoo is property which isn't enumerable
const myObj = Object.create({}, { getFoo: { value() { return this.foo; } } });
myObj.foo = 'bar';
console.log(Object.entries(myObj)); // [ ['foo', 'bar'] ]

// non-object argument will be coerced to an object
console.log(Object.entries('foo')); // [ ['0', 'f'], ['1', 'o'], ['2', 'o'] ]

// iterate through key-value gracefully
const obj = { a: 5, b: 7, c: 9 };
for (const [key, value] of Object.entries(obj)) {
  console.log(`${key} ${value}`); // "a 5", "b 7", "c 9"
}

// Or, using array extras
Object.entries(obj).forEach(([key, value]) => {
  console.log(`${key} ${value}`); // "a 5", "b 7", "c 9"
});
```

## Object.values\(obj\)

`Object.values()` 方法返回一个给定对象自身可枚举属性值的数组，其排列与使用`for...in` 循环遍历该对象时返回的顺序一致，不过`Object.values()`同样不会枚举原型链中的属性。

```js
var obj = { foo: 'bar', baz: 42 };
console.log(Object.values(obj)); // ['bar', 42]

// array like object
var obj = { 0: 'a', 1: 'b', 2: 'c' };
console.log(Object.values(obj)); // ['a', 'b', 'c']

// array like object with random key ordering
// when we use numeric keys, the value returned in a numerical order according to the keys
var an_obj = { 100: 'a', 2: 'b', 7: 'c' };
console.log(Object.values(an_obj)); // ['b', 'c', 'a']

// getFoo is property which isn't enumerable
var my_obj = Object.create({}, { getFoo: { value: function() { return this.foo; } } });
my_obj.foo = 'bar';
console.log(Object.values(my_obj)); // ['bar']

// non-object argument will be coerced to an object
console.log(Object.values('foo')); // ['f', 'o', 'o']
```

## String.prototype.padStart\(targetLen\[, padStr\]\)

**`padStart()`**方法用另一个字符串填充当前字符串\(重复，如果需要的话\)，以便产生的字符串达到给定的长度。填充从当前字符串的开始\(左侧\)应用的。

```js
'abc'.padStart(10);         // "       abc"
'abc'.padStart(10, "foo");  // "foofoofabc"
'abc'.padStart(6,"123465"); // "123abc"
'abc'.padStart(8, "0");     // "00000abc"
'abc'.padStart(1);          // "abc"
```

## String.prototype.padEnd\(targetLen\[, padStr\]\)

**`padEnd()`** 方法会用一个字符串填充当前字符串（如果需要的话则重复填充），返回填充后达到指定长度的字符串。从当前字符串的末尾（右侧）开始填充。

```js
'abc'.padEnd(10);          // "abc       "
'abc'.padEnd(10, "foo");   // "abcfoofoof"
'abc'.padEnd(6, "123456"); // "abc123"
'abc'.padEnd(1);           // "abc"
```

> 一个有趣的思考：为什么`String.prototype.padStart()`和`String.prototype.padEnd()`不叫做`String.prototype.padLeft()`和`String.prototype.padRight()`？
>
> 主要是为了能够支持双向或是从右向左的语言

## Object.getOwnPropertyDescriptors\(obj\)

很明显，该方法与ES6 中的`Object.getOwnPropertyDescriptor()`类似，都是用来获取属性的描述器。只不过，`Object.getOwnPropertyDescriptors()` 方法用来获取的是所有自身属性的描述符。

```js
const obj = {
    [Symbol('foo')]: 123,
    get bar() { return 'abc' },
};
console.log(Object.getOwnPropertyDescriptors(obj));

// Output:
// { [Symbol('foo')]:
//    { value: 123,
//      writable: true,
//      enumerable: true,
//      configurable: true },
//   bar:
//    { get: [Function: bar],
//      set: undefined,
//      enumerable: true,
//      configurable: true } }
```

`Object.assign()`方法只能拷贝源对象的可枚举的自身属性，并且无法拷贝属性的特性，而且访问器属性会被转换成数据属性，也无法拷贝源对象的原型。配合`Object.create()`和`Reflect.getPrototypeOf()`实现一个浅拷贝函数：

```js
function clone (obj) {
  return Object.create(
    Reflect.getPrototypeOf(obj), 
    Object.getOwnPropertyDescriptors(obj) 
  );
}
```

## 允许在参数定义和函数调用中使用尾随逗号

如果看过 [Airbnb 的 JavaScript 的规范中的逗号部分](https://github.com/airbnb/javascript?utm_source=gold_browser_extension#commas)，瞬间就能理解这个功能需求是多么的有意义

```js
function foo(
  param1,
  param2,
) {}

foo(
  'abc',
  'def',
);
```



