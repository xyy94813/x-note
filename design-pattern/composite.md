# 组合模式

使用它将对象组合成树状结构， 并且能像使用独立对象一样使用它们

## 适用场景

- 实现树状对象结构
- 希望客户端代码以相同方式处理简单和复杂元素

## 优/缺点

优点：

- 以利用多态和递归机制更方便地使用复杂树结构
- 开闭原则

缺点：

- 对于功能差异较大的类，提供公共接口或许会有困难

## 对比其他模式

- 桥接模式、状态模式和策略模式都是基于组合模式——即将工作委派给其他对象
- 在创建复杂组合树时使用生成器模式，因为这可使其构造步骤以递归的方式运行
- 责任链模式通常和组合模式结合使用
- 可以使用迭代器模式来遍历组合树
- 可以使用访问者模式对整个组合树执行操作
- 可以使用享元模式实现组合树的共享叶节点以节省内存
- 组合和装饰模式的结构图很相似，因为两者都依赖递归组合来组织无限数量的对象。
- 大量使用组合和装饰的设计通常可从对于原型模式的使用中获益

## 实现示例

```ts
interface Node {
  execute(): void;
}

class TypeANode implements Node {
  execute() {
    console.log("i am a");
  }
}

class TypeBNode implements Node {
  execute() {
    console.log("i am b");
  }
}

class NodeCompound {
  private children: Node[] = [];

  addChild(): void {}
  removeChild(): void {}

  execute(): void {
    this.children.forEach((child) => child.execute());
  }
}
```
