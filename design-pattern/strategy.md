# 策略模式

策略模式能让你定义一系列算法，并将每种算法分别放入独立的类中，以使算法的对象能够相互替换。

## 适用场景

- 想使用对象中各种不同的算法变体，并希望能在运行时切换算法时
- 有许多仅在执行某些行为时略有不同的相似类时
- 如果算法在上下文的逻辑中不是特别重要，使用该模式能将类的业务逻辑与其算法实现细节隔离开来。
- 当类中使用了复杂条件运算符以在同一算法的不同变体中切换时

## 优/缺点

优点：

- 可以在运行时切换对象内的算法
- 可以将算法的实现和使用算法的代码隔离开来
- 可以使用组合来代替继承
- 开闭原则。 无需对上下文进行修改就能够引入新的策略

缺点：

- 算法极少发生改变，那么没有任何理由引入新的类和接口。使用该模式只会让程序过于复杂。
- 客户端必须知晓策略间的不同——它需要选择合适的策略。
- 许多现代编程语言支持函数类型功能，允许在一组匿名函数中实现不同版本的算法。
  这样，使用这些函数的方式就和使用策略对象时完全相同，无需借助额外的类和接口来保持代码简洁。

## 对比其他模式

- 桥接模式、状态模式和策略模式都是基于组合模式——即将工作委派给其他对象
- 命令模式和策略看上去很像，因为两者都能通过某些行为来参数化对象。
  - 以使用命令来将任何操作转换为对象。操作的参数将成为对象的成员变量。
    可以通过转换来延迟操作的执行、将操作放入队列、保存历史命令或者向远程服务发送命令等。
  - 策略通常可用于描述完成某件事的不同方式，够在同一个上下文类中切换算法。
- 装饰模式可让你更改对象的外表，策略则让你能够改变其本质。
- 模板方法模式基于继承机制：它允许你通过扩展子类中的部分内容来改变部分算法。
  策略基于组合机制：你可以通过对相应行为提供不同的策略来改变对象的部分行为。
  模板方法在类层次上运作，因此它是静态的。
  策略在对象层次上运作，因此允许在运行时切换行为。
- 状态可被视为策略的扩展。
  策略使得这些对象相互之间完全独立，它们不知道其他对象的存在。
  但状态模式没有限制具体状态之间的依赖，且允许它们自行改变在不同情景下的状态。

## 实现示例

```ts
interface Operation {
  execute(a: number, b: number): number;
}

class AddOperation implements Operation {
  execute(a: number, b: number): number {
    return a + b;
  }
}

class SubtractOperation implements Operation {
  execute(a: number, b: number): number {
    return a - b;
  }
}
```
