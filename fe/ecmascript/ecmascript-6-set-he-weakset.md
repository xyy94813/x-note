# ECMAScript 6 Set 和 WeakSet

`Set`和`WeakSet`都是 ES6 新增数据结构。

## Set

ES6  中的数据结构`Set`，与其它语言的 Set 对象基本一致，Set 内的所有成员的值都是唯一的。

```js
const mySet = new Set([
    1, 1, 
    '2', '2', 
    null, null, 
    false, false, 
    true, true, 
    undefined, undefined,
    {}, {},
    [], [],
    NaN, NaN,
]);

mySet.forEach((item) => console.log(item));
// 1
// '2'
// null
// false
// true
// undefined
// {}
// {}
// []
// []
// NaN
```

> Set 内部判断两个值是否不同，使用的算法叫做“Same-value-zero equality”，它类似于精确相等运算符（`===`），主要的区别是`NaN`等于自身，而精确相等运算符认为`NaN`不等于自身。

Set实现了iterator接口，因此能使用`...`扩展符转换成数组\(Array\)

```js
// 数组去重
let arr = [1, 2, 3, 1, 2, 3];
arr = [...new Set(arr)];
console.log(arr, arr.size); // [1, 2, 3] 3
```

### 常用 API

**Set.prototype.add\(val\)**

添加值，返回`Set`实例本身

```js
const mySet = new Set();
mySet
.add(1)
.add(2)
.add(3)
.add(4);
console.log(...mySet); // 1 2 3 4
```

**Set.prototype.delete\(val\)**

删除`Set`实例中的某个值，返回实例本身

```js
const mySet = new Set([1, 2, 3]);

mySet
.delete(1)
.delete(2);

console.log(...mySet); // 1
```

**Set.prototype.has\(val\)**

判断`Set`实例中是否存在某个值

```js
const obj = {};
const mySet = new Set([1, obj]);

mySet.has(1); // true
mySet.has({}); // false
mySet.has(obj); // true
```

**Set.prototype.clear\(\)**

清空Set中存的所有值

```js
const mySet = new Set([1, 2, 3]);
mySet.clear();
console.log(mySet.size); // 0
```

**Set.prototype.values\(\)**

返回一个`Iterator`对象，这个对象以插入的`Set`对象的顺序包含了原`Set`对象里的每个元素

```js
const mySet = new Set([1, 2, 3]);

mySet.values(); // SetIterator {1, 2, 3}
```

**Set.prototype.keys\(\)**

同`Set.prototype.values`行为一致

**Set.prototype.entries\(\)**

类似于`Object.prototype.entries`类似

**Set.prototype.forEach\(callback\)**

类似于`Array.prototype.forEach`

```js
const mySet = new Set([1, 2, 3, 4, 5]);

mySet.forEach((item) => console.log(item)); // 1 2 3 4 5
```

## WeakSet

`WeakSet`对象与`Set`大体一致，每个对象值都只能出现一次，不过，不同的是：

* `WeakSet`对象只能存放对象引用，不能存放值，而Set对象可以；
* `WeakSet`对象中存储的对象值都是弱引用，如果没有其他的变量或属性引用这个对象值，则这个对象值会被当成来记回收掉；
* `WeakSet`无法被枚举，没有办法拿到它所包含的元素。

使用WeakSet存储 DOM 节点，就不用担心这些节点从文档中移除时造成内存泄漏。

### 相关 API

**WeakSet.prototype.add\(val\)**

与 `Set.prototype.add()`行为类似

**WeakSet.prototype.delete\(val\)**

与 `Set.prototype.delete()`行为类似

**WeakSet.prototype.has\(val\)**

与 `Set.prototype.has()`行为类似

**WeakSet.prototype.clear\(\)**

与 `Set.prototype.clear()`行为类似

```js
const myWeakSet = new WeakSet();

let obj0 = {};
let obj1 = {};
let obj2 = {};

// myWeakSet.add(1); // TypeError: Invalid value used in weak set

myWeakSet
  .add(obj0)
  .add(obj1)
  .add(obj2);

myWeakSet.has(obj0); // true
myWeakSet.has(obj1); // true
myWeakSet.has(obj2); // true

myWeakSet.delete(obj0); //
myWeakSet.has(obj0); // false

myWeakSet.clear();
myWeakSet.has(obj2); // false
```



