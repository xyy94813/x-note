# 享元模式

摒弃了在每个对象中保存所有数据的方式，通过共享多个对象所共有的相同状态，让你能在有限的内存容量中载入更多对象

## 适用场景

- 仅在程序必须支持大量对象且没有足够的内存容量时使用享元模式

## 优/缺点

优点：

- 如果程序中有很多相似对象，那么你将可以节省大量内存

缺点：

- 可能需要牺牲执行速度来换取内存，因为他人每次调用享元方法时都需要重新计算部分情景数据
- 代码会变得更加复杂

## 对比其他模式

- 可以使用享元模式实现组合模式树的共享叶节点以节省内存
- 享元展示了如何生成大量的小型对象，外观模式则展示了如何用一个对象来代表整个子系统
- 如果你能将对象的所有共享状态简化为一个享元对象，那么享元就和单例模式类似了。但这两个模式有两个根本性的不同

## 实现示例

```ts
/**
 * 享元类包含一个树的部分状态。
 * 这些成员变量保存的数值对于特定树而言是唯一的。
 * 例如，你在这里找不到树的坐标。
 * 但这里有很多树木之间所共有的纹理和颜色。
 * 由于这些数据的体积通常非常大，所以如果让每棵树都其进行保存的话将耗费大量内存。
 * 因此，我们可将纹理、颜色和其他重复数据导出到一个单独的对象中，然后让众多的单个树对象去引用它。
 * */
class TreeType {
  private name: string;
  private color: string;
  constructor(name, color) {}
  draw(canvas: Canvas, x: number, y: number): void {}
}

/**
 * 享元工厂决定是否复用已有享元或者创建一个新的对象
 * */
class TreeFactory {
  static treeTypes = [];
  static getTreeType(name, color): TreeType {
    const isEqual = (treeType: TreeType) => {
      return treeType.color === color && treeType.name === name;
    };
    let type: TreeType = TreeFactory.treeTypes.find(isEqual);

    if (!type) {
      type = new TreeType(name, color);
      treeTypes.add(type);
    }
    return type;
  }
}

/**
 * 情景对象包含树状态的外在部分。
 * 程序中可以创建数十亿个此类对象，因为它们体积很小：仅有两个整型坐标和一个引用成员变量。
 * */
class Tree {
  private x: number;
  private y: number;
  private type: TreeType;

  draw(canvas: Canvas): void {
    type.draw(canvas, this.x, this.y);
  }
}

// 树（Tree）和森林（Forest）类是享元的客户端。如果不打算继续对树类进行开
// 发，你可以将它们合并。
class Forest {
  private trees: Tree[];

  plantTree(x: number, y: number, name: string, color: string) {
    const tree = new Tree(x, y, TreeFactory.getTreeType(name, color));
    this.trees.add(tree);
  }

  draw(canvas: Canvas) {
    this.trees.forEach((tree) => tree.draw(canvas));
  }
}
```
