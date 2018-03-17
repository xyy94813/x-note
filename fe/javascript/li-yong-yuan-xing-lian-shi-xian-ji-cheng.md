# 利用原型链实现继承

在 ES6 实现`class`关键字之前，JavaScript 只能通过一些特殊的方式实现继承的概念。

常见的有以下几种实现方式：

* 临时变量
* `call`和`apply`改变上下文
* 原型链
* mixin

尽管前两种能够获得父类的属性和方法，但是`childreni_instance instanceof parent_instance`的结果为`false`



使用原型链实现继承：

```js
function Parent() {
    this.sayAge = function() {
        console.log(this.age)
    }
}

function Child(firstname) {
    this.fname = firstname;
    this.age = 40;
    this.saySomeThing = function() {
        console.log(this.fname);
        this.sayAge();
    }
}
Child.prototype = new Parent();
let child = new Child('zhang')
child.saySomeThing(); //  zhang 40
console.log(child instanceof Parent); // true

```

这种实现方式，中每个`Parent`实例的`sayAge()`方法实际上应该是共用的。
可以通过使用`call`或`apply`改变上下文的方式改良。改良如下：

```js
function Parent() { } 
Parent.prototype.sayAge = function() { 
    console.log(this.age); 
} 
function Child(firstname) { 
    Parent.call(this); 
    this.fname = firstname; 
    this.age = 40; 
    this.saySomeThing = function() { 
        console.log(this.fname); 
        this.sayAge(); 
    } 
} 
Child.prototype = new Parent(); 
let child = new Child('zhang'); 
child.saySomeThing(); // zhang 40

```
还有点问题，上述实现方式中，`Parent`实际上共创建了`Child`实例个数 + 1 个实例。

进一步结合 mixin 的方式改良：

```js
function extend (Child, Parent) {
    var F = function () {};
    F.prototype = Parent.prototype;
    var tmp = new F();
    tmp.constructor = Child;
    Child.prototype = tmp;
    return Child;
}
function Parent() { 
    console.log('a'); 
} 
Parent.prototype.sayAge = function() { 
    console.log(this.age); 
} 
function Child(firstname) { 
    Parent.call(this); 
    this.fname = firstname; 
    this.age = 40; 
    this.saySomeThing = function() { 
        console.log(this.fname); 
        this.sayAge(); 
    } 
} 
extend(Child, Parent);
var child = new Child('zhang'); 
child.saySomeThing(); // a zhang 40


```




