# MVVM

MVVM 模式支持 View 和 ViewModel 之间的双向数据绑定（data-binding），是其与 [MVP](./mvp) 最大的不同。
这样就可以自动将变化，在 ViewModel 的状态里面传播给 View。
一般情况下，ViewModel 利用观察者模式将 ViewModel 的变化告知 Model。
这种模式将一个应用程序分为以下几个主要方面：

- 模型（Model）
- 视图（View）
- 视图模型（ViewModel）

![mvvm-pattern from [Understanding The Difference Between MVC, MVP and MVVM Design Patterns](https://www.linkedin.com/pulse/understanding-difference-between-mvc-mvp-mvvm-design-rishabh-software)](../images/mvvm-pattern.jpeg)

## Model

模型表示解释业务逻辑即业务模型和数据模型（数据访问操作）的类的集合。
它还定义了数据的业务规则，即如何改变和操作数据。

## View

视图表示用户界面组件。
视图显示从控制器接收到的数据作为结果。
这也就把模型变成了用户界面。

## ViewModel

视图模型负责显示方法、命令和其他功能，这些方法、命令和功能协助维护视图的状态，将模型作为对视图的操作结果进行操作，并触发视图本身的事件。
