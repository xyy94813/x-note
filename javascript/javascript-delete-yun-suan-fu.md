# JavaScript delete 运算符

在 JavaScript 中，`delete` 是一元操作符，它用来删除对象属性或者数组元素。

```js
const obj = {
    a: 1,
    ['a-2']: 2, 
};
console.log('a' in obj); // true
console.log('a-2' in obj); // true


delete obj.a;
delete obj['a-2'];
console.log('a' in obj); // false
console.log('a-2' in obj); // false
```

> delete  只能删除自有属性，不能删除继承属性

```js
function Man(name) {
    this.name = name;
}
Man.prototype.sex = 1;

const alex = new Man('alex');
delete alex.name;
delete alex.sex;
console.log('name' in obj); // false
console.log('sex' in obj); // true
```

> `delete`实际上只是断开了属性和宿主对象的联系

```js
const obj = {
    a: {
        a1: 1,
    },
};
const p = obj.a;

delete obj.a;

console.log(p.a1); // 1

```

由于已经删除的属性的引用仍然存在，在某些 JavaScript 运行环境中，可能导致这部分的内存无法进行回收，最终造成内存泄露。

