# JavaScript 原型

每一个除 `null` 以外的对象都和另一个对象关联。这"另一个"对象的值就是原型，每一个对象都从原型继承属性。

所有通过过对象直接量创建的对象都具有同一个原型对象，并可以通过 JavaScript 代码 Object.prototype 获得对原型对象的引用。通过关键字 `new` 和构造函数创建对象的原型就是构造函数的 `prototype` 属性的值。因此，使用 `{}` 创建对象一样，通过 `new Object()` 创建的对象也继承自 `Object.prototype` 。同样，通过 `new Array()` 创建的对象原型就是 `Array.prototype` ，通过 `new Date()` 创建的对象的原型就是 `Date.prototype` 。

没有原型的对象不多， `Object.prototype` 就是其中之一。它不继承任何属性。其他原型对象都是普通对象，普通对象都具有原型。所有的内置构造函数（以及大部分能自定义的构造函数）都具有一个继承自 `Object.prototype` 的原型。例如，`Date.prototype` 的属性继承自 `Object.prototype` 。这一系列链接的原型对象就是所谓的**"原型链"（prototype chain）**。

