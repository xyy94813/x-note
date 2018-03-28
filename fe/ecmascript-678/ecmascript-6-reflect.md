# ECMAScript 6 Reflect

`Reflect`是一个内置的对象，提供拦截 JavaScript 操作的方法。这些方法与**处理器对象（handler）**的方法相同。

> `Reflect`不是一个函数对象，不可对其使用`new`操作符

ES6 推出`Reflect`的主要目的是，将一些明显属于语言范畴的方法，转移到`Reflect`对象上，例如：`Object.defineProperty`、`Function.prototype.apply()`，未来类似的功能也将只在`Reflect`对象上实现。其次，`Reflect`的方法返回值是一个布尔值，对于对象的操作更加合理，不需要使用`try...catch`进行捕获。此外，是某些命令式的操作，转为函数式的行为，比如：`delete obj[prop]`使用`Reflect.deleteProperty(obj, prop)`替代。

由于总能在`Reflect`对象上找到`Proxy`支持的拦截方法，`Proxy`可以通过调用`Reflect`对应的方法作为修改行为的基础，`Proxy`和`Reflect`的结合使得对对象的操作更加的便捷。

```js
const proxy = new Proxy({}, {
    get(...args) {
        const rest = Reflect.get(...args);
        if (rest) {
            console.log('get property success');
        } else {
            console.log('get property failed');
        }
        return rest;
    }
});

Reflect.get(proxy, 'a'); // get property failed

Reflect.set(proxy, 'a', 1);
Reflect.get(proxy, 'a'); //get property success
```



