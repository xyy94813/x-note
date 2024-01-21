# 模板方法模式

模板方法模式在超类中定义了一个算法的框架，允许子类在不修改结构的情况下重写算法的特定步骤。

## 适用场景

- 只希望客户端扩展某个特定算法步骤，而不是整个算法或其结构时，可使用模板方法模式。
- 多个类的算法除一些细微不同之外几乎完全一样时，可使用该模式。
  但其后果就是，只要算法发生变化，你就可能需要修改所有的类。

## 优/缺点

优点：

- 可仅允许客户端重写一个大型算法中的特定部分，使得算法其他部分修改对其所造成的影响减小。
- 可将重复代码提取到一个超类中。

缺点：

- 部分客户端可能会受到算法框架的限制。
- 通过子类抑制默认步骤实现可能会导致违反里氏替换原则。
- 模板方法中的步骤越多，其维护工作就可能会越困难。

## 对比其他模式

- 工厂方法模式是模板方法模式的一种特殊形式。工厂方法可以作为一个大型模板方法中的一个步骤。
- 模板方法基于继承机制：允许通过扩展子类中的部分内容来改变部分算法。
  策略模式基于组合机制：可以通过对相应行为提供不同的策略来改变对象的部分行为。
  模板方法在类层次上运作，因此它是静态的。
  策略在对象层次上运作，因此允许在运行时切换行为。

## 实现示例

```ts
abstract class GameAI {
  turn() {
    this.collectResources();
    this.buildStructures();
    this.buildUnits();
    this.attack();
  }
  collectResources() {
    this.builtStructures().forEach((s) => {
      s.collect();
    });
  }
  attack(): void {
    const enemy = closestEnemy();
    if (enemy == null) {
      this.sendScouts(map.center);
    } else {
      this.sendWarriors(enemy.position);
    }
  }
  buildStructures(): any[];
  buildUnits(): void;
  sendScouts(position: Position): void;
  sendWarriors(position: Position): void;
}

class OrcsAI extends GameAI {
  buildStructures() {
    // do something
  }
  buildUnits() {
    // do something
  }
  sendScouts() {
    // do something
  }
  sendWarriors() {
    // do something
  }
}

class MonstersAI extends GameAI {
  buildStructures() {
    // do something
  }
  buildUnits() {
    // do something
  }
  sendScouts() {
    // do something
  }
  sendWarriors() {
    // do something
  }
}
```
