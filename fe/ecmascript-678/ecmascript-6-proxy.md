# ECMAScript 6 Proxy {#reflect}

`Proxy`主要用于定义基本操作的自定义行为，包括属性查找、赋值、枚举、函数调用等。

`Proxy(target, handler)`构造函数接收两个参数，`target`为被代理的对象，`handler`为处理器对象，用来自定义代理对象的各种操作。目前，一共有 13 种可代理操作：

#### get\(target, propery, receiver\)

在读取代理对象的某个属性时触发该操作，比如在执行 `proxy.foo` 时。

```js
const target = {
    name: 'aaa',
};
const targetProxy = new Proxy(target, {
    // 在读取代理对象的某个属性时触发该操作
    get(target, property, receiver) {
        return target[property] || null;
    }
});

targetProxy.name === target.name; // true
targetProxy.x === null; // true
```

通过`proxy`的`get`拦截可以轻松实现 readOnly 和单例模式。

#### set\(target, propKey, value, receiver\)

在给代理对象的某个属性赋值时触发该操作，比如在执行 `proxy.foo = 1`时。

```js
const proxy = new Proxy({}, {
  set(obj, property, value) {
    if (property === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 150 || value < 0) {
        throw new RangeError('Wooo. Are you kinding me?');
      }
    } else if (property.startsWith('_')) {
      throw new Error('Can not write property which start with "_"')
    }
    // 对于满足条件的 age 属性以及其他属性，直接保存
    obj[prop] = value;
    return true;
  }
});

proxy.age; // undefined
proxy.age = 24;
proxy.age; // 24
proxy.age = '24' // TypeError: The age is not an integer
proxy.age = -1 // RangeError: Wooo. Are you kinding me?
proxy._sex = 'male'; // Error: Can not write property which start with "_"
```

可以通过设置代理对象的`set`处理器轻松的根据需求对对象的写入进行控制。

#### deleteProperty\(target, property\)

在删除代理对象的某个属性时触发该操作，比如在执行 `delete proxy.foo`时。

```js
const proxy = new Proxy({
  _a: 1,
  a: 2,
}, {
  deleteProperty(target, property) {
    if (property.startsWith('_')) {
      throw Error('Can not delete property which start with "_"')
    }
    return delete target[property];
  }
});

delete proxy.a;
proxy.a; // undefined
delete proxy._a; // Error: Can not delete property which start with "_"
```

#### defineProperty\(target, property, descriptor\)

在定义代理对象某个属性时的属性描述时触发该操作，如`Object.defineProperty()`和`Object.defineProperties()`。

> * 如果目标对象不可扩展， 将不能添加属性；
> * 不能添加或者修改一个属性为不可配置的，如果它不作为一个目标对象的不可配置的属性存在的话；
> * 如果目标对象存在一个对应的可配置属性，这个属性可能不会是不可配置的；
> * 如果一个属性在目标对象中存在对应的属性，那么`Object.defineProperty(target, prop, descriptor)`将不会抛出异常；
> * 在严格模式下，`false`作为`handler.defineProperty`方法的返回值的话将会抛出 `TypeError`。

```js
const proxy = new Proxy({}, {
  defineProperty(target, prop, descriptor) {
    console.log('called: ' + prop);
    return Object.defineProperty(target, prop, descriptor);
  }
});

Object.defineProperty(proxy, 'a', { 
  configurable: false, 
  enumerable: true, 
  value: 10, 
});

Object.defineProperty(proxy, 'a', { 
  configurable: true, 
}); // TypeError: Cannot redefine property: a
```

#### has\(target, property\)

在判断代理对象是否拥有某个属性时触发该操作，如`in`运算符。

> * 如果目标对象的某一属性本身不可被配置，则该属性不能够被代理隐藏
> * 如果目标对象为不可扩展对象，则该对象的属性不能够被代理隐藏

```js
const proxy = new Proxy({
  a: 1,
  _a: 2,
}, {
  has(target, prop) {
    if (prop.startsWith('_')) {
      return false;
    }
    return prop in target;
  }
});
console.log('a' in proxy); // true
console.log('_a' in proxy); // false

const obj = { a: 10 };
Object.preventExtensions(obj);
const proxy2 = new Proxy(obj, {
  has(target, prop) {
    return false;
  }
});
console.log('a' in proxy2); 
// TypeError: 'has' on proxy: trap returned falsish for property 'a' but the proxy target is not extensible
```

`has`拦截对`for...in`循环不生效。

```js
const obj = { x: 1, y: 2 };
const proxy = new Proxy(obj, {
  has(target, prop) {
    console.log(prop);
    return false;
  }
});

for (let b in proxy) {
  console.log(proxy[b]);
}
// 1 
// 2
```

#### ownKeys\(target\)

在获取代理对象的所有属性键时触发该操作，如在 `Object.getOwnPropertyNames(obj)`时。

> * 结果必须是一个数组；
> * 数组的元素类型只能是`String`或`Symbol`；
> * 结果列表必须包含目标对象的所有不可配置（non-configurable ）、自有（own）属性的key；
> * 如果目标对象不可扩展，那么结果列表必须包含目标对象的所有自有（own）属性的key，不能有其它值。

```js
const obj = {};
Object.defineProperty(obj, 'a', { 
  configurable: false, 
  enumerable: true, 
  value: 10 
});

const p = new Proxy(obj, {
  ownKeys(target) {
    return ['a', Symbol.for('b'), 'c'];
  }
});
console.log(Object.getOwnPropertyNames(p)); // [ 'a', 'c' ]

const p2 = new Proxy(obj, {
  ownKeys(target) {
    return ['a', 123, 12.5, true, false, undefined, null, {}, []];
  }
});
console.log(Object.getOwnPropertyNames(p2)); // TypeError: XXX is not a valid property name

const p3 = new Proxy(obj, {
  ownKeys(target) {
    return ['b', 'c'];
  }
});
console.log(Object.getOwnPropertyNames(p3)); 
// TypeError: 'ownKeys' on proxy: trap result did not include 'a'

Object.preventExtensions(obj);
const p4 = new Proxy(obj, {
  ownKeys(target) {
    return ['a', 'b'];
  }
});
console.log(Object.getOwnPropertyNames(p4)); 
// TypeError: 'ownKeys' on proxy: trap returned extra keys but proxy target is non-extensible
```

#### getOwnPropertyDescriptor\(target, property\)

在获取代理对象某个属性的属性描述时触发该操作，如：`Object.getOwnPropertyDescriptor(obj, prop)`。

> * 必须返回一个`object`或`undefined`
> * 如果属性作为目标对象的不可配置的属性存在，则该属性无法报告为不存在
> * 如果属性作为目标对象的属性存在，并且目标对象不可扩展，则该属性无法报告为不存在
> * 如果属性不存在作为目标对象的属性，并且目标对象不可扩展，则不能将其报告为存在
> * 属性不能被报告为不可配置，如果它不作为目标对象的自身属性存在，或者作为目标对象的可配置的属性存在
> * `Object.getOwnPropertyDescriptor(target)`的结果可以使用`Object.defineProperty`应用于目标对象，也不会抛出异常

```js
const obj = { a: 20 };
Object.defineProperty(obj, 'b', { 
  configurable: false, 
  enumerable: true, 
  value: 20 
});


var p = new Proxy(obj, {
  getOwnPropertyDescriptor(target, prop) {
    return { configurable: true, enumerable: true, value: 10 };
  }
});
console.log(Object.getOwnPropertyDescriptor(p, 'a').value);


var p2 = new Proxy(obj, {
  getOwnPropertyDescriptor(target, prop) {
    return [];
  }
});
console.log(Object.getOwnPropertyDescriptor(p2, 'b'));
// TypeError: 'getOwnPropertyDescriptor' on proxy: trap reported non-configurability for property 'a' which is either non-existant or configurable in the proxy target

var p3 = new Proxy(obj, {
  getOwnPropertyDescriptor(target, prop) {
    return undefined;
  }
});
console.log(Object.getOwnPropertyDescriptor(p3, 'b'));
// TypeError: 'getOwnPropertyDescriptor' on proxy: trap returned undefined for property 'a' which is non-configurable in the proxy target
```

#### apply\(target, context, args\)

在调用一个目标对象为函数的代理对象时触发该操作，比如 `proxy()`、`proxy.call()`、`proxy.apply()`。

> `target`必须可被调用

```js
function target(...args) {
    return args.reduce((sum, item) => sum + item, 0);
}

const proxy = new Proxy(target, {
    apply(target, context, args) {
        console.log(context);
        return target(...args) - 1;
    }
});

proxy(1, 2, 3); // undefined 5
proxy.call(null, 1, 2, 3); // null 5
proxy.apply(window, [1, 2, 3]); // window 5
```

#### construct \(target, args, newTarget\)

> `target`必须是一个合法的 constructor ，并且必须返回一个对象

```js
const proxy = new Proxy(function () {}, {
  construct: function(target, args, newTarget) {
    return { value: args[0] * 10 };
  }
});
(new proxy(1)).value; // 10

const proxy2 = new Proxy(function () {}, {
  construct: function(target, args, newTarget) {}
});
new proxy2(1); // TypeError: 'construct' on proxy: trap returned non-object ('undefined')

const proxy3 = new Proxy({}, {
  construct: function(target, args, newTarget) {
    return { value: args[0] * 10 };
  }
});
new proxy3(1); // TypeError: proxy3 is not a constructor
```

#### getPrototypeOf\(proxy\)

在读取代理对象的原型时触发该操作，比如在执行`Object.getPrototypeOf(proxy)`时。

```js
const proxy = new Proxy({}, {
    getPrototypeOf(target) {
      return Array.prototype;
    }
});

Object.getPrototypeOf(proxy) === Array.prototype; // true
proxy.__proto__ === Array.prototype; // true
proxy.isPrototypeOf(Array.prototype); // true
console.log(proxy instanceof Array); // true
```

当同时触发处理器`get`和`getPrototypeOf`操作时，`get`的优先级更高。

```js
const proxy = new Proxy({}, {
    getPrototypeOf(target) {
      return Array.prototype;
    },
    get(target, property) {
      return null;
    }
});

Object.getPrototypeOf(proxy) === Array.prototype; // true
proxy.__proto__ === Array.prototype; // false
proxy.isPrototypeOf(Array.prototype); // TypeError: proxy.isPrototypeOf is not a function
console.log(proxy instanceof Array); // true
```

#### setPrototypeOf\(target, proto\)

在设置代理对象的原型时触发该操作，如：`Object.setPrototypeOf(proxy)`。如果返回`false`会抛出个`TypeError`异常。

> 如果`target`不可扩展，原型参数必须与`Object.getPrototypeOf()`的值相同。

```js
const proxy = new Proxy(function () {}, {
    setPrototypeOf(target, proto) {
        return false;
    }
});
Object.setPrototypeOf(proxy, {}); // TypeError: 'setPrototypeOf' on proxy: trap returned falsish

const target = {};
Object.preventExtensions(target);
const proxy2 = new Proxy(target, {
    setPrototypeOf(target, proto) {
        return true;
    }
});
Object.setPrototypeOf(proxy2, {}); 
// TypeError: 'setPrototypeOf' on proxy: trap returned truish for setting a new prototype on the non-extensible proxy target
```

#### isExtensible\(target\)

在判断一个代理对象是否是可扩展时触发该操作，如：`Object.isExetensible(proxy)`时。

> `Object.isExtensible(proxy)` 必须返回与`Object.isExtensible(target)`相同的值。

```js
const p = new Proxy({}, {
  isExtensible(target) {
    return true;
  }
});
Object.isExtensible(p); // true

const obj = {};
Object.preventExtensions(obj);
const p1 = new Proxy(obj, {
  isExtensible(target) {
    return true;
  }
});
const p2 = new Proxy(obj, {
  isExtensible(target) {
    return false;
  }
});
Object.isExtensible(p1); // TypeError: 'isExtensible' on proxy: trap result does not reflect extensibility of proxy target (which is 'false')
Object.isExtensible(p2); // false
```

#### preventExtensions\(target\)

在让一个代理对象不可扩展时触发该操作，比如在执行`Object.preventExtensions(proxy)`时。如果返回`false`会抛出个`TypeError`异常。

> 如果`Object.isExtensible(proxy)`是`false`，`Object.preventExtensions(proxy)`只能返回`true`。

```js
const p = new Proxy({}, {
  preventExtensions(target) {
    Object.preventExtensions(target);
    return false;
  }
});

Object.preventExtensions(p) // TypeError: 'preventExtensions' on proxy: trap returned falsish
```



