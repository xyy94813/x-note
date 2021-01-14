# MVC

MVC 设计模式最早出现于 20 世纪 70 年代。
它推动了关注点的分叉，这意味着领域模型和控制器逻辑与用户界面（视图）分离。
因此，应用程序的维护和测试变得简单和容易。
MVC 设计模式将一个应用程序分为三个主要方面：

- 模型（Model）
- 视图（View）
- 控制器（Controller）

![mvc-pattern from [Understanding The Difference Between MVC, MVP and MVVM Design Patterns](https://www.linkedin.com/pulse/understanding-difference-between-mvc-mvp-mvvm-design-rishabh-software)](../images/mvc-pattern.jpeg)

## Model

模型表示解释业务逻辑即业务模型和数据模型（数据访问操作）的类的集合。
它还定义了数据的业务规则，即如何改变和操作数据。

## View

视图表示用户界面组件。
视图显示从控制器接收到的数据作为结果。
这也就把模型变成了用户界面。

## Controller

Controller 的职责是处理传入的请求。
它通过 View 获得用户的输入，然后通过 Model 处理用户的数据，将结果传回 View。
它通常充当 View 和 Model 之间的中介。
