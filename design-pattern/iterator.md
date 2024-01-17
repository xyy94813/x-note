# 迭代器模式

迭代器模式能在不暴露集合底层表现形式（列表、栈和树等）的情况下遍历集合中所有的元素

## 适用场景

- 当集合背后为复杂的数据结构，且你希望对客户端隐藏其复杂性时（出于使用便利性或安全性的考虑），可以使用迭代器模式
- 使用该模式可以减少程序中重复的遍历代码
- 如果你希望代码能够遍历不同的甚至是无法预知的数据结构，可以使用迭代器模式

## 优/缺点

优点：

- 单一职责原则。通过将体积庞大的遍历算法代码抽取为独立的类，你可对客户端代码和集合进行整理。
- 开闭原则。可实现新型的集合和迭代器并将其传递给现有代码，无需修改现有代码。
- 可以并行遍历同一集合，因为每个迭代器对象都包含其自身的遍历状态
- 可以暂停遍历并在需要时继续

缺点：

- 只与简单的集合进行交互，应用该模式可能会矫枉过正
- 对于某些特殊集合，使用迭代器可能比直接遍历的效率低

## 对比其他模式

- 可以使用迭代器模式来遍历组合模式树
- 可以同时使用工厂方法模式和迭代器来让子类集合返回不同类型的迭代器，并使得迭代器与集合相匹配
- 可以同时使用备忘录模式和迭代器来获取当前迭代器的状态，并且在需要的时候进行回滚
- 可以同时使用访问者模式和迭代器来遍历复杂数据结构，并对其中的元素执行所需操作，即使这些元素所属的类完全不同

## 实现示例

```ts
interface Iterator {
  getNext(): any;
  hasMore(): bool;
}

class ArrayIterator<T = any> implements Iterator {
  private index: number = 0;
  constructor(arr: Array) {
    this.source = arr;
  }
  getNext(): T {
    if (!this.hasMore()) return null;
    const result = this.source[this.index];
    this.index += 1;
    return result;
  }
  hasMore() {
    return this.index < this.source.length;
  }
}
```

js 中实现通用的 iterator：

```ts
class Foo {
  length = 3;
  *[Symbol.iterator]() {
    for (let i = 0; i < this.length; i++) {
      yield i ** 2;
    }
  }
}
```
