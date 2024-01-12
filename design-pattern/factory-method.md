# 工厂方法

其在父类中提供一个创建对象的方法，允许子类决定实例化对象的类型

## 适用场景

- 如果无法预知对象确切类别及其依赖关系时
- 如果希望用户能扩展软件库或框架的内部组件
- 如果希望复用现有对象来节省系统资源，而不是每次都重新创建对象

## 优/缺点

优点：

- 避免创建者和具体产品之间的紧密耦合
- 单一职责原则。可以将产品创建代码放在程序的单一位置，从而使得代码更容易维护
- 开闭原则。无需更改现有客户端代码，就可以在程序中引入新的产品类型。

缺点：

- 应用工厂方法模式需要引入许多新的子类，代码可能会因此变得更复杂。最好的情况是将该模式引入创建者类的现有层次结构中。

## 对比其他模式

许多设计工作的初期都会使用工厂方法模式（较为简单，而且可以更方便地通过子类进行定制），随后演化为使用抽象工厂模式、原型模式或生成器模式（更灵活但更加复杂）

抽象工厂模式通常基于一组工厂方法，但也可以使用原型模式来生成这些类的方法

可以同时使用工厂方法和迭代器模式来让子类集合返回不同类型的迭代器，并使得迭代器与集合相匹配。

原型并不基于继承，因此没有继承的缺点。
另一方面，原型需要对被复制对象进行复杂的初始化。
工厂方法基于继承，但是它不需要初始化步骤。

工厂方法是模板方法模式的一种特殊形式。
同时，工厂方法可以作为一个大型模板方法中的一个步骤。

## 实现示例

```ts
interface Brand {
  slogan(): void;
}

class BrandA implements Brand {
  slogan(): void {
    //
    console.log("i am brand A");
  }
}

abstract class BrandStore {
  createBrand(): Brand;
  //
  publicize(): void {
    const brand: Brand = this.createBrand();
    brand.slogan();
  }
}

class BrandAStore extends BrandStore {
  createBrand() {
    return new BrandA();
  }
}

function main() {
  let store: BrandStore;

  const street: string = getGeoInfo();

  if (street === "a street") {
    store = new BrandAStore();
  } else if (street === "b street") {
    // 如果要扩展 product b
    store = new BrandBStore();
  } else {
    throw new Error("Unknown street");
  }

  store.publicize();
}

// 扩展新产品
class BrandB implements Brand {
  slogan(): void {
    console.log("i am brand B");
  }
}

class BrandBStore extends BrandStore {
  createBrand() {
    return new BrandB();
  }
}
```
