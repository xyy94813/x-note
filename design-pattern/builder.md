# 生成器模式

能够分步骤创建复杂对象。
允许用相同的创建代码生成不同类型和形式的对象。

## 适用场景

- 避免“重叠构造函数（telescoping constructor）”的出现
- 使用代码创建不同形式的产品
- 造组合树或其他复杂对象

## 优/缺点

优点：

- 可以分步创建对象，暂缓创建步骤或递归运行创建步骤
- 生成不同形式的产品时，你可以复用相同的制造代码
- 单一职责原则

缺点：

- 该模式需要新增多个类，因此代码整体复杂程度会有所增加

## 对比其他模式

- 常常由工厂方法演化而来
- 生成器重点关注如何分步生成复杂对象。抽象工厂专门用于生产一系列相关对象。抽象工厂会马上返回产品，生成器则允许你在获取产品前执行一些额外构造步骤。
- 创建复杂组合模式树时使用生成器，因为这可使其构造步骤以递归的方式运行
- 结合使用生成器和桥接模式：主管类负责抽象工作，各种不同的生成器负责实现工作
- 抽象工厂、生成器和原型都可以用单例模式来实现

## 实现示例

```ts
class Car {
  seats: number;
  engine: string;
  color: string;
  setSeats(seats: number) {
    this.seats = seats;
  }
  setEngine(engine: string) {
    this.engine = engine;
  }
  setColor(color: string) {
    this.color = color;
  }
}

class Manual {
  seats: number;
  engine: string;
  color: string;
  setSeats(seats: number) {
    this.seats = seats;
  }
  setEngine(engine: string) {
    this.engine = engine;
  }
  setColor(color: string) {
    this.color = color;
  }
}

interface CarBuilder {
  reset(): this;
  setSeats(seats: number): this;
  setEngine(engine: string): this;
  setColor(color: string): this;
  // getResult(): any
}

class ConcreteCarBuilder implements CarBuilder {
  private car: Car;
  constructor() {
    this.car = new Car();
  }
  reset() {
    this.car = new Car();
    return this;
  }
  setSeats(seats: number) {
    this.car.setSeats(seats);
    return this;
  }
  setEngine(engine: string) {
    this.car.setEngine(engine);
    return this;
  }
  setColor(color: string) {
    this.car.setColor(color);
    return this;
  }
  getResult() {
    const result = this.car;
    this.reset();
    return result;
  }
}

class ConcreteManualBuilder implements CarBuilder {
  private manual: Manual;
  constructor() {
    this.manual = new Manual();
  }
  reset() {
    this.manual = new Manual();
    return this;
  }
  setSeats(seats: number) {
    this.manual.setSeats(seats);
    return this;
  }
  setEngine(engine: string) {
    this.manual.setEngine(engine);
    return this;
  }
  setColor(color: string) {
    this.manual.setColor(color);
    return this;
  }
  getResult() {
    const result = this.manual;
    this.reset();
    return result;
  }
}

class Director {
  buildRedSportsCar(builder: Builder) {
    builder
      .reset() //
      .setSeats(2) //
      .setEngine("SportEngine")
      .setColor("red");
  }
  //   buildBlackSUV(builder: Builder) {
  //     builder
  //       .reset() //
  //       .setSeats(5) //
  //       .setEngine('Normal Engine')
  //       .setColor('black');
  //   }
}

function main() {
  const director = new Director();

  const carBuilder: CarBuilder = new ConcreteCarBuilder();
  director.buildRedSportsCar(carBuilder);
  const car: Car = carBuilder.getResult();

  const manualBuilder: CarBuilder = new ConcreteManualBuilder();
  director.buildRedSportsCar(manualBuilder);
  const manual: Manual = manualBuilder.getResult();
}
```
