# 观察者模式

观察者模式允许定义一种订阅机制，可在对象事件发生时通知多个“观察”该对象的其他对象。

## 适用场景

- 一个对象状态的改变需要改变其他对象，或实际对象是事先未知的或动态变化的时
- 当应用中的一些对象必须观察其他对象时，可使用该模式。但仅能在有限时间内或特定情况下使用。

## 优/缺点

优点：

- 开闭原则。无需修改发布者代码就能引入新的订阅者类（如果是发布者接口则可轻松引入发布者类）。
- 可以在运行时建立对象之间的联系

缺点：

- 订阅者的通知顺序是随机的

## 对比其他模式

- 责任链模式、命令模式、中介者模式和观察者模式用于处理请求发送者和接收者之间的不同连接方式：
  - 责任链按照顺序将请求动态传递给一系列的潜在接收者，直至其中一名接收者对请求进行处理。
  - 命令在发送者和请求者之间建立单向连接。
  - 中介者清除了发送者和请求者之间的直接连接，强制它们通过一个中介对象进行间接沟通。
  - 观察者允许接收者动态地订阅或取消接收请求。
- 中介者的主要目标是消除一系列系统组件之间的相互依赖。
  发布订阅的实现方式依赖于观察者。
  中介者对象担当发布者的角色，其他组件则作为订阅者，可以订阅中介者的事件或取消订阅。
  当中介者以这种方式实现时，它可能看上去与观察者非常相似。

## 实现示例

```ts
abstract class Event {
  private listeners: Record<string, Set<Function>>;

  private getListenersOfType = (eventType: string): Set<Function> => {
    let listenersOfType = listeners[eventType];
    if (!listenersOfType) {
      listenersOfType = new Set();
      listeners[eventType] = listenersOfType;
    }
    return listenersOfType;
  };

  private notify = (eventType: string, args: any[]) => {
    const listenersOfType = this.getListenersOfType(eventType);
    listenersOfType.forEach((listener) => {
      listener(args);
    });
  };

  subscribe = (eventType: string, listener: Function): (() => void) => {
    const listenersOfType = this.getListenersOfType(eventType);
    listenersOfType.add(eventType, listener);

    return () => this.unsubscribe(eventType, listener);
  };

  unsubscribe = (eventType: string, listener: Function) => {
    const listenersOfType = this.getListenersOfType(eventType);
    listenersOfType.remove(listener);
  };
}
```
