# 抽象工厂

它能创建一系列相关的对象， 而无需指定其具体类

## 适用场景

- 如果代码需要与多个不同系列的相关产品交互，但是由于无法提前获取相关信息，或者出于对未来扩展性的考虑，不希望代码基于产品的具体类进行构建，在这种情况下，可以使用抽象工厂。
- 如果有一个基于一组抽象方法的类，且其主要功能因此变得不明确，那么在这种情况下可以考虑使用抽象工厂模式

## 优/缺点

优点：

- 可以确保同一工厂生成的产品相互匹配
- 可以避免客户端和具体产品代码的耦合
- 单一职责原则。可以将产品生成代码抽取到同一位置，使得代码易于维护。
- 开闭原则。向应用程序中引入新产品变体时，无需修改客户端代码。

缺点：

- 由于采用该模式需要向应用中引入众多接口和类，代码可能会比之前更加复杂

## 对比其他模式

- 早期都会使用工厂方法模式（较为简单，而且可以更方便地通过子类进行定制），随后演化为使用抽象工厂模式、原型模式或生成器模式（更灵活但更加复杂）
- 生成器重点关注如何分步生成复杂对象。抽象工厂专门用于生产一系列相关对象。抽象工厂会马上返回产品，生成器则允许你在获取产品前执行一些额外构造步骤
- 抽象工厂模式通常基于一组工厂方法，但也可以使用原型模式来生成这些类的方法
- 只需对客户端代码隐藏子系统创建对象的方式时，可以使用抽象工厂来代替外观模式
- 可以将抽象工厂和桥接模式搭配使用。如果由桥接定义的抽象只能与特定实现合作，这一模式搭配就非常有用。在这种情况下，抽象工厂可以对这些关系进行封装，并且对客户端代码隐藏其复杂性
- 抽象工厂、生成器和原型都可以用单例模式来实现

## 实现示例

```ts
interface GUIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

class Button {
  render(): void {
    // do something
    console.log(this.name);
  }
}

class Checkbox {
  render(): void {
    // do something
    console.log(this.name);
  }
}

class LightThemeButton extends Button {}
class LightThemeCheckbox extends Checkbox {}

class LightThemeFactory implements GUIFactory {
  createButton(): LightThemeButton {
    return new LightThemeButton();
  }
  createCheckbox(): LightThemeCheckbox {
    return new LightThemeCheckbox();
  }
}

class Page {
  guiFactory: GUIFactory;

  constructor(guiFactory: GUIFactory) {
    this.guiFactory = guiFactory;
  }
  render() {
    const button = this.guiFactory.createButton();
    button.render();
    const checkbox = this.guiFactory.createCheckbox();
    checkbox.render();
  }
}

function main() {
  const config = readApplicationConfigFile();
  let guiFactory: GUIFactory;

  if (config.theme === "Dark") {
    guiFactory = new DarkThemeFactory();
  } else {
    guiFactory = new LightThemeFactory();
  }

  const page = new Page(guiFactory);
  page.render();
}

// 新增一个主题
class DarkThemeButton extends Button {}
class DarkThemeCheckbox extends Checkbox {}

class DarkThemeFactory implements GUIFactory {
  createButton(): DarkThemeButton {
    return new DarkThemeButton();
  }
  createCheckbox(): DarkThemeCheckbox {
    return new DarkThemeCheckbox();
  }
}
```
