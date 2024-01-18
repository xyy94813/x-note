# 中介者模式

中介者模式能减少对象之间混乱无序的依赖关系。
该模式会限制对象之间的直接交互，迫使它们通过一个中介者对象进行合作。（EventBus）

## 适用场景

- 当一些对象和其他对象紧密耦合以致难以对其进行修改时
- 当组件因过于依赖其他组件而无法在不同应用中复用时
- 为了能在不同情景下复用一些基本行为，但会导致被迫创建大量组件子类时

## 优/缺点

优点：

- 单一职责原则。可以将多个组件间的交流抽取到同一位置，使其更易于理解和维护。
- 开闭原则。无需修改实际组件就能增加新的中介者。
- 可以减轻应用中多个组件间的耦合情况。
- 可以更方便地复用各个组件

缺点：

- 一段时间后，中介者可能会演化成为上帝对象。

## 对比其他模式

- 责任链模式、命令模式、中介者模式和观察者模式用于处理请求发送者和接收者之间的不同连接方式：
  - 责任链按照顺序将请求动态传递给一系列的潜在接收者，直至其中一名接收者对请求进行处理。
  - 命令在发送者和请求者之间建立单向连接。
  - 中介者清除了发送者和请求者之间的直接连接，强制它们通过一个中介对象进行间接沟通。
  - 观察者允许接收者动态地订阅或取消接收请求。
- 外观模式和中介者的职责类似：它们都尝试在大量紧密耦合的类中组织起合作
- 中介者的主要目标是消除一系列系统组件之间的相互依赖。观察者的目标是在对象之间建立动态的单向连接，使得部分对象可作为其他对象的附属发挥作用。

## 实现示例

```ts
interface Mediator {
  notify(sender: Component, event: string): void;
}

class AuthenticationDialog implements Mediator {
  private title: string;
  private loginOrRegisterChkBx: Checkbox;
  private loginUsername: Textbox;
  private loginPassword: Textbox;
  private registrationUsername: Textbox;
  private registrationPassword: Textbox;
  private registrationEmail: Textbox;
  private okBtn: Button;
  private cancelBtn: Button;

  notify = (sender: any, event: string): void => {
    if (sender === this.loginOrRegisterChkBx) {
      if (event === 'check') {
        if (this.loginOrRegisterChkBx.checked) {
          this.title = '登录';
        } else {
          this.title = '注册';
        }
      }
    }
    // else if {}
  };
}

class Component {
  dialog: Mediator;

  constructor(dialog: Dialog) {
    this.dialog = dialog;
  }

  click = (): void => {
    dialog.notify(this, 'click');
  };
}

class Checkbox extends Component {
  check = (): void => {
    dialog.notify(this, 'check');
  };
}
```
