# 单例模式

保证一个类只有一个实例，并提供一个访问该实例的全局节点

## 适用场景

- 某个类对于所有客户端只有一个可用的实例
- 需要更加严格的控制全局变量

## 优/缺点

优点：

- 全局唯一
- 获得了一个指向该实例的全局访问节点
- 可以 lazy 单例节约内存

缺点：

- 违反了单一职责原则
- 单例模式可能掩盖不良设计，比如程序各组件之间相互了解过多
- 该模式在多线程环境下需要进行特殊处理，避免多个线程多次创建单例对象
- 单例的客户端代码单元测试可能会比较困难，因为许多测试框架以基于继承的方式创建模拟对象。

## 对比其他模式

- 外观模式类通常可以转换为单例模式类，因为在大部分情况下一个外观对象就足够
- 将对象的所有共享状态简化为一个享元对象，那么享元模式就和单例类似了。但这两个模式有两个根本性的不同。
- 抽象工厂模式、生成器模式和原型模式都可以用单例来实现

## 实现示例

```ts

let instance

class Database {
  private client: any;
  static public getInstance () => {
    if (!instance) {
        instance = new Database()
    }
    return instance
  }
}
```

JS 等语言利用自身特性可以轻易实现单例模式

```ts
// in Database.ts
let instance;

class Database {}

export const getInstance = () => {
  if (!instance) {
    instance = new Database();
  }
  return instance;
};
```

```ts
// in Database.ts
let instance;

class Database {
  constructor() {
    if (instance) {
      return instance;
    }
    instance = this;
  }
}

export default Database;
```
