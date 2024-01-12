# 原型模式

复制已有对象，而又无需使代码依赖它们所属的类

## 适用场景

- 需要复制一些对象，同时又希望代码独立于这些对象所属的具体类，可以使用原型模式
- 子类的区别仅在于其对象的初始化方式，那么你可以使用该模式来减少子类的数量

## 优/缺点

优点：

- 可以克隆对象，而无需与它们所属的具体类相耦合
- 可以克隆预生成原型，避免反复运行初始化代码
- 可以更方便地生成复杂对象
- 可以用继承以外的方式来处理复杂对象的不同配置

缺点：

- 克隆包含循环引用的复杂对象可能会非常麻烦

## 对比其他模式

- 通常由工厂方法演化而来
- 抽象工厂模式通常基于一组工厂方法，但你也可以使用原型模式来生成这些类的方法。
- 原型可用于保存命令模式的历史记录。
- 大量使用组合模式和装饰模式的设计通常可从对于原型的使用中获益。你可以通过该模式来复制复杂结构，而非从零开始重新构造。
- 原型并不基于继承，因此没有继承的缺点。另一方面，原型需要对被复制对象进行复杂的初始化。工厂方法基于继承，但是它不需要初始化步骤。
- 有时候原型可以作为备忘录模式的一个简化版本，其条件是你需要在历史记录中存储的对象的状态比较简单，不需要链接其他外部资源，或者链接可以方便地重建。
- 抽象工厂、生成器和原型都可以用单例模式来实现。

## 实现示例

```ts
interface Prototype {
  clone(): Prototype;
}

class ConcretePrototype implements Prototype {
  filed1: any;
  constructor(prototype: Prototype) {
    this.filed1 = prototype.filed1;
  }
  clone() {
    return new ConcretePrototype(this);
  }
}

class SubClassPrototype implements Prototype {
  filed2: any;
  constructor(prototype: Prototype) {
    super(prototype);
    this.filed2 = prototype.filed2;
  }
  clone(): SubClassPrototype {
    return new SubClassPrototype(this);
  }
}
```

> 可以进一步了解 JS 中的原型链
