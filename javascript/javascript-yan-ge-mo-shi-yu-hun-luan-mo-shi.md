# JavaScript 严格模式与混乱模式

从 ES5 开始，ES5引入了**严格模式（Strict Mode）**，并使用`use strict`作为语法声明未使用严格模式时，则称之为**混乱模式（非严格模式）**

ES6 的 `class` 与模块内默认采用严格模式。

| 严格模式 | 非严格模式 |
| :--- | :--- |
| 禁止使用`with`语句 | 允许使用`with`语句 |
| 所有变量要先声明 | 使用未声明的变量将隐式声明为全局变量 |
| 函数中的`this`是`undefined` | `this`默认指向全局对象 |
| `call()`和`apply()`传入的第一个值不会被转换 | `call()`和`apply()`传入的第一个值如果是`null`或`undefined`则会被全局对象取代，如果是原始值则转换为对应的包装对象 |
| 给只读属性和不可扩展的对象创建新成员将会抛出类型错误异常 | 只是简单的操作失败 |
| 传入`eval()`的代码不能再定义变量和函数 | 变量和函数定义在`eval()`创建的新作用域中 |
| `delete`后跟非法标识符将抛出语法错误异常 | 只是简单的返回`false` |
| `delete`删除不可配置的属性将抛出类型错误异常 | 只是简单的返回`false` |
| 在对象直接量中定义多个同名属性将产生语法错误 | 不会报错 |
| 函数声明存在多个同名的参数将产生语法错误 | 不会报错 |
| 不允许使用八进制直接量 | 允许部分实现 |
| 将`eval()`和`arguments`当作关键字，并且不允许更改 |  |
| 限制了对栈的检测能力，`arguments.caller`和`arguments.callee`将抛出类型错误异常 |  |
| 函数中的`arguments`对象拥有传入函数值的静态副本 |  |


示例：
```js
function func() {
    console.log(this);
}
func(); // window

function func2() {
    'use strict'
    console.log(this);
}
func2(); // undefined

class A {
    method(obj) {
        console.log(this);
    }
}
const a = new A;
const aMethod = a.method;
aMethod(); // undefined

```



