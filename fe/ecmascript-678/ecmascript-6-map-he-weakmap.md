# ECMAScript 6 Map 和 WeakMap

`Map`和`WeakMap`也是 ES6 推出的新的数据结构。

## Map

`Map`对象保存着多组键值对，与`Object`不同的是，任何对象或原始值都可以作为一个键。

> `Map`对象的键是唯一的，`Map`比较键是否是唯一的算法是"Same-value-zero equality"，与`Set`比较值是否唯一的算法一样，除了`NaN === NaN`，其他运算结构与`===`运算发一致。

```js
const myMap = new Map();

const key1 = {};
const key2 = function () {};
const key3 = 'str';

myMap
.set(key1, 'key is object')
.set(key2, 'key is function')
.set(key3, 'key is str')
.set(NaN, 'key is NaN');
myMap.size; // 3

  
myMap.get(key1); // key is object
myMap.get(key2); // key is function
myMap.get(key3); // key is str
myMap.get(NaN); // key is NaN
myMap.get({}); // undefined
```

能够给`Map`对象的构造函数传入一个二维数组，直接生成一个`Map`实例

```js
const myMap = new Map([['key', 'val']]);
myMap.get('key'); // val
```

Map对象也实现了 Iterator 协议，因此能够使用`for...of`语句遍历，使用`for...of`语句遍历`Map`对象时，返回一个`[key, val]`数组

```js
var myMap = new Map();
myMap
.set(0, "zero")
.set(1, "one");
for (var [key, value] of myMap) {
  console.log(key, value);
}
// 0 'zero'
// 1 'one'
```

大多数情况下，都能使用`Object`来替代`Map`对象，那么在选择使用Map对象还是Object时，可以考虑以下问题：

* 在运行之前 key 是否是未知的？
* 是否需要动态地查询 key 呢？
* 是否所有的值都是统一类型，这些值可以互换么？
* 是否需要不是字符串类型的 key ？
* 键值对经常增加或者删除么？
* 是否有任意个且非常容易改变的键值对?
* 这个集合可以遍历么\(Is the collection iterated\)？

如果，满足上述条件中的大多数，那么使用`Map`会比`Object`更好。

### 相关 API

**Map.prototype.set\(key, val\)**

设置Map对象中键的值。返回该Map对象。

```js
const myMap = new Map();

myMap
  .set('key1', 'val1')
  .set('key2', 'val2')
  .set('key3', 'val3');
myMap.size; // 3
```

**Map.prototype.get\(key\)**

返回键对应的值，如果不存在，则返回`undefined`。

```js
const myMap = new Map([['key', 'val']]);
myMap.get('key'); // val
myMap.get('key2'); // undefined
```

**Map.prototype.delete\(key\)**

移除该键的关联，如果存在关联返回`true`，不存在则返回`false`。

```js
const myMap = new Map([['key', 'val']]);
myMap.get('key'); // val
myMap.delete('key'); // true
myMap.get('key'); // undefined
myMap.delete('key'); // false
```

**Map.prototype.has\(key\)**

返回一个布尔值，表示`Map`实例是否包含键对应的值。

```js
const myMap = new Map([['key', 'val']]);
myMap.has('key'); // true
```

**Map.prototype.clear\(\)**

移除`Map`对象的所有键/值对

```js
const myMap = new Map([['key', 'val']]);
myMap.clear();
```

**Map.prototype.entries\(\)**

返回一个新的`Iterator`对象，它按插入顺序包含了Map对象中每个元素的`[key, value]`数组。

```js
const myMap = new Map([['key', 'val'], ['key2', 'val2']]);
const mapIter = myMap.entries();
mapIter.next().value; // ['key', 'val']
mapIter.next().value; // ['key2', 'val2']
```

**Map.prototype.keys\(\)**

返回一个新的`Iterator`对象， 它按插入顺序包含了Map对象中每个元素的键。

```js
const myMap = new Map([['key', 'val'], ['key2', 'val2']]);
const mapIter = myMap.keys();
mapIter.next().value; // key
mapIter.next().value; // key2
```

**Map.prototype.values\(\)**

返回一个新的`Iterator`对象，它按插入顺序包含了Map对象中每个元素的值。

```js
const myMap = new Map([['key', 'val'], ['key2', 'val2']]);
const mapIter = myMap.keys();
mapIter.next().value; // val
mapIter.next().value; // val2
```

**Map.prototype.forEach\(callBack\[, context\]\)**

按插入顺序，为`Map`对象里的每一键值对调用一次回调函数。如果为`forEach`提供了`context`，它将在每次回调中作为`this`值。

```js
const myMap = new Map([['key', 'val'], ['key2', 'val2']]);
myMap.forEach(function (val, key, map) {
    console.log(key, val, this);
}, {});
// key val {}
// key2 val2 {}
```

## WeakMap

与`Map`所不同的是，`WeakMap`中的键必须是对象，而其是弱引用的。（值可以是任意的）

在 JavaScript 里，可以通过四个共用两个数组\(一个存放键，一个存放值\)的 API 方法来实现 map API。给该 map 设置值时会同时将键和值推到这两个数组的末尾。从而使得键和值的索引在两个数组中相对应。当从该 map 取值的时候，需要遍历所有的键，然后使用此索引从存储值的数组中检索出相应的值。

但这样的实现会有两个很大的缺点，首先是** O\(n\) 的时间复杂度**\(n 是键值对的个数\)。另外一个则可能会导致**内存泄漏**，在这种自己实现的 WeakMap 中，存放键的数组中的每个索引将会保持对所引用对象的引用,阻止他们被当作垃圾回收。

而在原生的 WeakMap 中，每个键对自己所引用对象的引用是 "弱引用"，这意味着,如果没有其他引用和该键引用同一个对象，这个对象将会被当作垃圾回收。原生 WeakMap 的结构是特殊且有效的，其用于映射的 key 只有在其没有被回收时才是有效的。

正由于这样的弱引用，`WeakMap`的 key 是非枚举的 \(没有方法能给出所有的 key\)。如果key 是可枚举的话，其列表将会受垃圾回收机制的影响，从而得到不确定的结果. 因此,如果你想要这种类型对象的 key 值的列表，你应该使用`Map`。

```js
var wm1 = new WeakMap(),
    wm2 = new WeakMap(),
    wm3 = new WeakMap();
var o1 = {},
    o2 = function(){},
    o3 = window;

wm1.set(o1, 37);
wm1.set(o2, "azerty");
wm2.set(o1, o2); // value可以是任意值,包括一个对象
wm2.set(o3, undefined);
wm2.set(wm1, wm2); // 键和值可以是任意对象,甚至另外一个WeakMap对象
wm1.get(o2); // "azerty"
wm2.get(o2); // undefined,wm2中没有o2这个键
wm2.get(o3); // undefined,值就是undefined

wm1.has(o2); // true
wm2.has(o2); // false
wm2.has(o3); // true (即使值是undefined)

wm3.set(o1, 37);
wm3.get(o1); // 37
wm3.clear();
wm3.get(o1); // undefined,wm3已被清空
wm1.has(o1);   // true
wm1.delete(o1);
wm1.has(o1);   // false
```

### 相关 API

**WeakMap.prototype.set\(key, val\)**

移除`key`的关联对象。

**WeakMap.prototype.get\(key\)**

返回`key`关联对象， 没有key关联对象时返回`undefined`。

**WeakMap.prototype.delete\(key, val\)**

移除该键的关联，如果存在关联返回`true`，不存在则返回`false`。

**WeakMap.prototype.has\(key, val\)**

返回一个布尔值，表明`WeakMap`实例是否包含键对应的值。

