# 桥接模式

将一个大类或一系列紧密相关的类拆分为抽象和实现两个独立的层次结构，从而能在开发时分别使用

## 适用场景

- 想要拆分或重组一个具有多重功能的庞杂类（例如能与多个数据库服务器进行交互的类），可以使用桥接模式。
- 希望在几个独立维度上扩展一个类，可使用该模式

## 优/缺点

优点：

- 可以创建与平台无关的类和程序
- 客户端代码仅与高层抽象部分进行互动，不会接触到平台的详细信息。
- 开闭原则。你可以新增抽象部分和实现部分，且它们之间不会相互影响。
- 单一职责原则。抽象部分专注于处理高层逻辑，实现部分处理平台细节。

缺点：

- 对高内聚的类使用该模式可能会让代码更加复杂

## 对比其他模式

- 常会于开发前期进行设计，使你能够将程序的各个部分独立开来以便开发。另一方面，适配器模式通常在已有程序中使用，让相互不兼容的类能很好地合作
- 桥接、状态模式和策略模式（在某种程度上包括适配器）模式的接口非常相似。实际上，它们都基于组合模式——即将工作委派给其他对象，不过也各自解决了不同的问题
- 以将抽象工厂模式和桥接搭配使用。如果由桥接定义的抽象只能与特定实现合作，这一模式搭配就非常有用。在这种情况下，抽象工厂可以对这些关系进行封装，并且对客户端代码隐藏其复杂性。
- 可以结合使用生成器模式和桥接模式：主管类负责抽象工作，各种不同的生成器负责实现工作

## 实现示例

```ts
interface Device {
  enable(): void;
  disable(): void;
  isEnabled(): boolean;
  getVolume(): number;
  setVolume(percent: number): void;
}

class RemoteControl {
  private device: Device;
  constructor(device: Device) {
    this.device = device;
  }
  togglePower() {
    if (device.isEnabled()) {
      device.disable();
    } else {
      device.enable();
    }
  }

  volumeUp() {
    device.setVolume(device.getVolume() + 10);
  }
  volumeDown() {
    device.setVolume(device.getVolume() - 10);
  }
}

class AdvancedRemoteControl extends RemoteControl {
  mute() {
    device.setVolume(25);
  }
}

class TV implements Device {}

class Radio implements Device {}

function main() {
  const tv = new TV();
  const tvControl = new AdvancedRemoteControl(tv);

  const radio = new Radio();
  const radioControl = new AdvancedRemoteControl(radio);
}
```
