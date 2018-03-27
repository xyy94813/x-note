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
>
> * 不能添加或者修改一个属性为不可配置的，如果它不作为一个目标对象的不可配置的属性存在的话；
>
> * 如果目标对象存在一个对应的可配置属性，这个属性可能不会是不可配置的；
>
> * 如果一个属性在目标对象中存在对应的属性，那么`Object.defineProperty(target, prop, descriptor)`将不会抛出异常；
>
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



