# 访问者模式

策略模式能将算法与其所作用的对象隔离开来

## 适用场景

- 需要对一个复杂对象结构（例如对象树）中的所有元素执行某些操作，可使用访问者模式。
- 可使用访问者模式来清理辅助行为的业务逻辑。
- 当某个行为仅在类层次结构中的一些类中有意义，而在其他类中没有意义时，可使用该模式。

## 优/缺点

优点：

- 开闭原则。可以引入在不同类对象上执行的新行为，且无需对这些类做出修改。
- 单一职责原则。可将同一行为的不同版本移到同一个类中。
- 访问者对象可以在与各种对象交互时收集一些有用的信息。
  当想要遍历一些复杂的对象结构（例如对象树），并在结构中的每个对象上应用访问者时，这些信息可能会有所帮助。

缺点：

- 每次在元素层次结构中添加或移除一个类时，都要更新所有的访问者
- 在访问者同某个元素进行交互时，它们可能没有访问元素私有成员变量和方法的必要权限。

## 对比其他模式

- 可以将访问者模式视为命令模式的加强版本， 其对象可对不同类的多种对象执行操作。
- 可以使用访问者对整个组合模式树执行操作。
- 可以同时使用访问者和迭代器模式来遍历复杂数据结构，并对其中的元素执行所需操作，即使这些元素所属的类完全不同。

## 实现示例

```ts
interface Shape {
  move(x: number, y: number): void;
  draw(): void;
  //
  accept(v: Visitor): void;
}

class Dot implements Shape {
  // ……

  // 注意我们正在调用的`visitDot（访问点）`方法与当前类的名称相匹配。
  // 这样我们能让访问者知晓与其交互的元素类。
  accept(v: Visitor) {
    v.visitDot(this);
  }
}

class Circle implements Shape {
  accept(v: Visitor) {
    v.visitCircle(this);
  }
}

class Rectangle implements Shape {
  accept(v: Visitor) {
    v.visitRectangle(this);
  }
}

class CompoundShape implements Shape {
  accept(v: Visitor) {
    v.visitCompoundShape(this);
  }
}

interface Visitor {
  visitDot(d: Dot): any;
  visitCircle(c: Circle): any;
  visitRectangle(r: Rectangle): any;
  // addon
  visitCompoundShape(cs: CompoundShape): any;
}

class XMLExportVisitor implements Visitor {
  visitDot(d: Dot): any;
  visitCircle(c: Circle): any;
  visitRectangle(r: Rectangle): any;
  //
  visitCompoundShape(cs: CompoundShape): any;
}

class ImageExportVisitor implements Visitor {
  visitDot(d: Dot): any;
  visitCircle(c: Circle): any;
  visitRectangle(r: Rectangle): any;
  //
  visitCompoundShape(cs: CompoundShape): any;
}
```
