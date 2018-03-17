# 利用原型链实现继承

在 ES6 实现`class`关键字之前，JavaScript 只能通过一些特殊的方式实现继承的概念。

常见的有以下几种实现方式：

* 临时变量
* `call`和`apply`改变上下文
* 原型链
* mixin

尽管前三种能够获得父类的属性和方法，但是`childreni_instance instanceof parent_instance`的结果为`false` 

